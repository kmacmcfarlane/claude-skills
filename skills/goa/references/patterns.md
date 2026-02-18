# Goa v3 Design Patterns

## Complete Service Example

```go
package design

import . "goa.design/goa/v3/dsl"

var _ = API("calc", func() {
    Title("Calculator Service")
    Description("HTTP service for multiplying numbers")
    Server("calc", func() {
        Description("calc hosts the Calculator Service.")
        Services("calc")
        Host("development", func() {
            Description("Development hosts.")
            URI("http://localhost:8000/calc")
            URI("grpc://localhost:8080")
        })
        Host("production", func() {
            Description("Production hosts.")
            URI("https://{version}.goa.design/calc")
            URI("grpcs://{version}.goa.design")
            Variable("version", String, "API version", func() {
                Default("v1")
            })
        })
    })
})

var _ = Service("calc", func() {
    Description("The calc service performs operations on numbers")
    Method("multiply", func() {
        Payload(func() {
            Attribute("a", Int, "Left operand", func() {
                Meta("rpc:tag", "1")
            })
            Field(2, "b", Int, "Right operand")
            Required("a", "b")
        })
        Result(Int)
        HTTP(func() {
            GET("/multiply/{a}/{b}")
            Response(StatusOK)
        })
        GRPC(func() {
            Response(CodeOK)
        })
    })
    Files("/openapi.json", "gen/http/openapi3.json")
})
```

## CRUD Service Pattern

```go
var _ = Service("users", func() {
    Description("User management service")

    Method("list", func() {
        Payload(func() {
            Attribute("page", Int, "Page number", func() {
                Default(1)
                Minimum(1)
            })
            Attribute("per_page", Int, "Items per page", func() {
                Default(20)
                Minimum(1)
                Maximum(100)
            })
        })
        Result(CollectionOf(UserResult))
        HTTP(func() {
            GET("/users")
            Param("page")
            Param("per_page")
            Response(StatusOK)
        })
    })

    Method("get", func() {
        Payload(func() {
            Attribute("id", String, "User ID", func() {
                Format(FormatUUID)
            })
            Required("id")
        })
        Result(UserResult)
        Error("not_found", ErrorResult, "User not found")
        HTTP(func() {
            GET("/users/{id}")
            Response(StatusOK)
            Response("not_found", StatusNotFound)
        })
    })

    Method("create", func() {
        Payload(func() {
            Attribute("name", String, "User name", func() {
                MinLength(1)
                MaxLength(255)
            })
            Attribute("email", String, "Email address", func() {
                Format(FormatEmail)
            })
            Required("name", "email")
        })
        Result(UserResult)
        Error("already_exists", ErrorResult, "User already exists")
        HTTP(func() {
            POST("/users")
            Response(StatusCreated)
            Response("already_exists", StatusConflict)
        })
    })

    Method("update", func() {
        Payload(func() {
            Attribute("id", String, "User ID", func() {
                Format(FormatUUID)
            })
            Attribute("name", String, "User name", func() {
                MinLength(1)
                MaxLength(255)
            })
            Attribute("email", String, "Email address", func() {
                Format(FormatEmail)
            })
            Required("id")
        })
        Result(UserResult)
        Error("not_found", ErrorResult, "User not found")
        HTTP(func() {
            PUT("/users/{id}")
            Response(StatusOK)
            Response("not_found", StatusNotFound)
        })
    })

    Method("delete", func() {
        Payload(func() {
            Attribute("id", String, "User ID", func() {
                Format(FormatUUID)
            })
            Required("id")
        })
        Error("not_found", ErrorResult, "User not found")
        HTTP(func() {
            DELETE("/users/{id}")
            Response(StatusNoContent)
            Response("not_found", StatusNotFound)
        })
    })
})

var UserResult = ResultType("application/vnd.user", "UserResult", func() {
    Attributes(func() {
        Attribute("id", String, "User ID", func() {
            Format(FormatUUID)
        })
        Attribute("name", String, "User name")
        Attribute("email", String, "Email address")
        Attribute("created_at", String, "Created timestamp", func() {
            Format(FormatDateTime)
        })
        Required("id", "name", "email", "created_at")
    })
    View("default", func() {
        Attribute("id")
        Attribute("name")
        Attribute("email")
        Attribute("created_at")
    })
    View("tiny", func() {
        Attribute("id")
        Attribute("name")
    })
})
```

## Error Handling

### Defining Errors at Three Levels

```go
// API-level (shared across all services)
var _ = API("myapi", func() {
    Error("unauthorized")
    HTTP(func() {
        Response("unauthorized", StatusUnauthorized)
    })
})

// Service-level (shared across all methods in service)
var _ = Service("users", func() {
    Error("invalid_arguments", ErrorResult, "Invalid arguments")
})

// Method-level
Method("divide", func() {
    Error("div_by_zero")
})
```

### Custom Error Types

```go
var DivByZero = Type("DivByZero", func() {
    Field(1, "message", String, "Error message")
    Field(2, "dividend", Int, "Dividend value")
    Field(3, "name", String, "Error name", func() {
        Meta("struct:error:name")  // Required for multiple custom errors
    })
    Required("message", "dividend", "name")
})

Method("divide", func() {
    Error("DivByZero", DivByZero, "Division by zero")
    HTTP(func() {
        Response("DivByZero", StatusBadRequest)
    })
})
```

### Error Properties

```go
Error("service_unavailable", ErrorResult, func() { Temporary() })  // Client should retry
Error("request_timeout", ErrorResult, func() { Timeout() })        // Deadline exceeded
Error("internal_error", ErrorResult, func() { Fault() })           // Server-side problem
```

### Producing Errors in Implementation

```go
func (s *dividerSvc) IntegralDivide(ctx context.Context, p *divider.IntOperands) (int, error) {
    if p.Divisor == 0 {
        return 0, gendivider.MakeDivByZero(fmt.Errorf("divisor cannot be zero"))
    }
    return p.Dividend / p.Divisor, nil
}
```

### Consuming Errors (Client-Side)

```go
res, err := client.Divide(ctx, payload)
if serr, ok := err.(*goa.ServiceError); ok {
    switch serr.Name {
    case "DivByZero":
        // Handle division by zero
    }
}
```

## Security Patterns

### JWT Authentication

```go
var JWT = JWTSecurity("jwt", func() {
    Scope("api:read", "Read access")
    Scope("api:write", "Write access")
})

var _ = Service("secure", func() {
    Security(JWT)

    Method("list", func() {
        Security(JWT, func() {
            Scope("api:read")
        })
        Payload(func() {
            Token("token", String)
            Required("token")
        })
        Result(CollectionOf(ItemResult))
        HTTP(func() {
            GET("/items")
        })
    })

    // Public endpoint
    Method("health", func() {
        NoSecurity()
        Result(String)
        HTTP(func() {
            GET("/health")
        })
    })
})
```

### Basic Auth

```go
var BasicAuth = BasicAuthSecurity("basicauth", func() {
    Description("Use your credentials")
})

Method("login", func() {
    Security(BasicAuth)
    Payload(func() {
        Username("user", String)
        Password("pass", String)
    })
    HTTP(func() {
        POST("/login")
    })
})
```

### API Key

```go
var APIKeyAuth = APIKeySecurity("api_key", func() {
    Description("API key authentication")
})

Method("secure_endpoint", func() {
    Security(APIKeyAuth)
    Payload(func() {
        APIKey("api_key", "key", String, "API key")
        Required("key")
    })
    HTTP(func() {
        GET("/secure")
        Param("key:key")  // Send as query parameter
        // Or: Header("key:X-API-Key")  // Send as header
    })
})
```

### OAuth2

```go
var OAuth2 = OAuth2Security("oauth2", func() {
    AuthorizationCodeFlow("/authorize", "/token", "/refresh")
    Scope("api:read", "Read access")
    Scope("api:write", "Write access")
})
```

## ResultType with Views

```go
var BottleResult = ResultType("application/vnd.bottle", "BottleResult", func() {
    Description("A bottle of wine")
    Attributes(func() {
        Attribute("id", Int, "ID of bottle")
        Attribute("name", String, "Name of bottle")
        Attribute("vintage", Int, "Vintage year")
        Attribute("account", AccountResult, "Owner account")
        Required("id", "name")
    })
    View("default", func() {
        Attribute("id")
        Attribute("name")
        Attribute("vintage")
    })
    View("extended", func() {
        Attribute("id")
        Attribute("name")
        Attribute("vintage")
        Attribute("account")
    })
    View("tiny", func() {
        Attribute("id")
        Attribute("name")
    })
})

// Collection with specific view
var TinyBottles = CollectionOf(BottleResult, func() {
    View("tiny")
})
```

## Payload with Validation

```go
Payload(func() {
    Attribute("username", String, "Username", func() {
        Pattern("^[a-zA-Z][a-zA-Z0-9_]{2,31}$")
        MinLength(3)
        MaxLength(32)
    })
    Attribute("email", String, "Email address", func() {
        Format(FormatEmail)
    })
    Attribute("age", Int, "User age", func() {
        Minimum(0)
        Maximum(150)
    })
    Attribute("role", String, "User role", func() {
        Enum("admin", "user", "moderator")
        Default("user")
    })
    Attribute("tags", ArrayOf(String), "User tags", func() {
        MinLength(1)
        MaxLength(10)
    })
    Required("username", "email")
})
```

## Multi-Transport (HTTP + gRPC)

```go
Method("add", func() {
    Payload(func() {
        Field(1, "a", Int, "Left operand")
        Field(2, "b", Int, "Right operand")
        Required("a", "b")
    })
    Result(Int)
    HTTP(func() {
        POST("/add")
        Response(StatusOK)
    })
    GRPC(func() {
        Response(CodeOK)
    })
})
```

Use `Field()` instead of `Attribute()` when supporting gRPC — the numeric tag becomes the protobuf field number.

## Service Implementation Pattern

```go
package myapi

import (
    "context"
    "log"
    gen "mymodule/gen/myservice"
)

type myservicesrvc struct {
    logger *log.Logger
}

func NewMyService(logger *log.Logger) gen.Service {
    return &myservicesrvc{logger}
}

func (s *myservicesrvc) List(ctx context.Context, p *gen.ListPayload) (*gen.ListResult, error) {
    s.logger.Printf("myservice.list called with page=%d", p.Page)
    // Your business logic here
    return &gen.ListResult{}, nil
}
```

## Static Files

```go
Service("web", func() {
    Files("/static/{*path}", "./public")
    Files("/favicon.ico", "./public/favicon.ico")
    Files("/openapi.json", "gen/http/openapi3.json")
})
```

## Interceptors

```go
var Logging = Interceptor("Logging", func() {
    ReadPayload(func() {
        Attribute("id", String)
    })
})

var _ = Service("catalog", func() {
    ServerInterceptor(Logging)
    // methods...
})
```

Interceptors vs HTTP middleware:
- HTTP middleware: standard `http.Handler` pattern for transport concerns (CORS, compression)
- Goa interceptors: type-safe access to domain types, compile-time checked
