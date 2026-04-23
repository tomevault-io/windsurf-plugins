---
trigger: always_on
description: This project follows the standard Go project layout:
---

# Go Code Style and Structure

## Project Structure

This project follows the standard Go project layout:

- `/cmd` - Main application entry points
- `/internal` - Private application and library code
  - `/api` - HTTP API controllers
  - `/service` - Business logic layer
  - `/middleware` - HTTP middleware
  - `/model` - Data models
  - `/util` - Utility functions

## Naming Conventions

- Package names should be lowercase words, without underscores or mixed case.
- Use CamelCase for naming.
  - Exported functions, variables, constants, and types should start with an uppercase letter (e.g., `PublicFunction`).
  - Unexported functions, variables, constants, and types should start with a lowercase letter (e.g., `privateVariable`).
- Interface names often end with "er" (e.g., `Reader`, `Writer`).
- Controller structs should end with "Controller".
- Constants should be in all uppercase with underscores separating words (e.g., `MAX_CONNECTIONS`).

## Code Organization

- Each Controller should have a corresponding Service.
- The Controller is responsible for handling HTTP requests and input validation.
- The Controller should implement the `github.com/sven-victor/ez-console/server.Controller` interface and be registered via `server.RegisterControllers`. Example:

```go
package api

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/sven-victor/ez-console/server"
)

type EchoController struct {
}

func (c *EchoController) RegisterRoutes(router *gin.RouterGroup) {
    echoRouter := router.Group("echo")
	echoRouter.GET("", c.Get)
}

func (c *EchoController) Get(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, gin.H{"message": "ok"})
}

/*
import (
    ldap "github.com/go-ldap/ldap/v3"
)
// service.LDAPClient
type LDAPClient struct {
    *ldap.Conn
}


// server.Service
type Service interface {
	// LDAP Service
	GetLDAPSession(ctx context.Context) (*service.LDAPClient, error)
	GetLDAPSettings(ctx context.Context) (*model.LDAPSettings, error)
	TestLDAPConnection(ctx context.Context, ldapSettings model.LDAPSettings, username, password string) (*model.LDAPTestResponse, error)

	// Setting Service
	FilterLDAPEntries(ctx context.Context, baseDN string, filter string, attributes []string) ([]*ldap.Entry, error)
	GetSettingsMap(ctx context.Context) (map[string]string, error)
}
*/
func init() {
	server.RegisterControllers(func(svc server.Service) server.Controller {
		return &EchoController{}
	})
}
```

- After registering a Controller, the framework will automatically call its `RegisterRoutes` method to register the routes with the Gin engine.
- Routes registered via `RegisterRoutes` require login by default.
- The Service is responsible for business logic implementation and database queries.
- All API endpoints should use a RESTful design style.
- Models (types) that need to be synchronized with the database should be registered using `server.RegisterModels()`. The framework will then automatically perform an `AutoMigrate` operation.

## Commenting Conventions

- All exported functions, types, and variables should have comments.
- Use full sentences for comments, starting with the function/method/variable name.
- Complex logic should have inline comments for explanation.

## Permission Control

- Insert permission middleware when registering routes. Code example:

```go
package api

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/sven-victor/ez-console/server"
	"github.com/sven-victor/ez-console/pkg/middleware"
)

type ApplicationController struct {
}

func (c *ApplicationController) RegisterRoutes(router *gin.RouterGroup) {
    users := router.Group("/applications")
	router.GET("", middleware.RequirePermission("applications:view"), c.GetApplications)
	router.POST("roles", middleware.RequirePermission("applications:roles:create"), c.CreateApplicationRole)
}

func (c *ApplicationController) GetApplications(ctx *gin.Context) {
    // example code
}
func (c *ApplicationController) CreateApplicationRole(ctx *gin.Context) {
    // example code
}

func init() {
	server.RegisterControllers(func(svc server.Service) server.Controller {
		return &ApplicationController{}
	})
    
	middleware.RegisterPermission("Application Management", "Manage application creation, editing, deletion, and role assignment", []model.Permission{
		{
			Code:        "applications:view",
			Name:        "View applications",
			Description: "View applications list and details",
		},
		{
			Code:        "applications:roles:create",
			Name:        "Create application roles",
			Description: "Create application roles",
		},
	})
}
```

## Error Handling

- Errors should be returned, not panicked.
- Use custom error types where appropriate.
- The API layer should convert errors to appropriate HTTP responses using `util.RespondWithError`. Example:

```go
userID := ctx.Param("id")
if userID == "" {
    util.RespondWithError(ctx, util.ErrorResponse{
        Code: "E4001",
        Err:  errors.New("User ID cannot be empty"),
    })
    return
}
var filters service.AuditLogFilters
if err := ctx.BindQuery(&filters); err != nil {
    util.RespondWithError(ctx, util.ErrorResponse{
        Code:    "E4001",
        Err:     err,
        Message: "failed to parse query parameters",
    })
    return
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sven-victor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
