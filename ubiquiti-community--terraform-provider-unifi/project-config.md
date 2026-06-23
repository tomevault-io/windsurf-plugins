---
trigger: always_on
description: This guide provides comprehensive instructions for developing resources in the terraform-provider-unifi using the Terraform Plugin Framework. The provider is built entirely on the Plugin Framework (no SDK v2 dependencies).
---

# Terraform Provider Development Guide: Plugin Framework

## Overview

This guide provides comprehensive instructions for developing resources in the terraform-provider-unifi using the Terraform Plugin Framework. The provider is built entirely on the Plugin Framework (no SDK v2 dependencies).

## Prerequisites

- **Repository**: [terraform-provider-unifi](https://github.com/ubiquiti-community/terraform-provider-unifi)
- **Framework**: Terraform Plugin Framework v1.12+
- **Go Version**: 1.22+
- **API Client**: [go-unifi](https://github.com/ubiquiti-community/go-unifi)

## Key Documentation References

- [Plugin Framework Overview](https://developer.hashicorp.com/terraform/plugin/framework)
- [Provider Framework Tutorial](https://developer.hashicorp.com/terraform/tutorials/providers-plugin-framework)
- [Schema Concepts](https://developer.hashicorp.com/terraform/plugin/framework/handling-data/schemas)
- [Resource Implementation](https://developer.hashicorp.com/terraform/plugin/framework/resources)
- [Nested Attributes](https://developer.hashicorp.com/terraform/plugin/framework/handling-data/attributes#nested-attribute-types)
- [Testing Framework](https://developer.hashicorp.com/terraform/plugin/framework/testing)

## Project Structure

The provider uses a flat structure within the `unifi/` directory:

```
terraform-provider-unifi/
├── main.go                    # Provider entry point
├── unifi/
│   ├── provider.go           # Provider implementation
│   ├── provider_test.go      # Provider tests
│   ├── resource_*.go         # Resource implementations
│   ├── data_source_*.go      # Data source implementations
│   ├── *_action.go           # Action implementations
│   └── util/                 # Utility functions
│       ├── conversion.go     # Type conversion helpers
│       └── retry/           # Retry logic
└── docs/                     # Generated documentation
```

## Provider Entry Point

The provider uses the Plugin Framework's server implementation:
```go
package main

import (
    "context"
    "flag"
    "log"

    "github.com/hashicorp/terraform-plugin-framework/providerserver"
    "github.com/ubiquiti-community/terraform-provider-unifi/unifi"
)

//go:generate go tool tfplugindocs generate -provider-name unifi
func main() {
    var debug bool

    flag.BoolVar(
        &debug,
        "debug",
        false,
        "set to true to run the provider with support for debuggers like delve",
    )
    flag.Parse()

    opts := providerserver.ServeOpts{
        Address: "registry.terraform.io/ubiquiti-community/unifi",
        Debug:   debug,
    }

    err := providerserver.Serve(context.Background(), unifi.New, opts)
    if err != nil {
        log.Fatal(err.Error())
    }
}
```

## Provider Implementation

The provider implements multiple interfaces to support various features:

```go
package unifi

import (
    "github.com/hashicorp/terraform-plugin-framework/action"
    "github.com/hashicorp/terraform-plugin-framework/datasource"
    "github.com/hashicorp/terraform-plugin-framework/ephemeral"
    "github.com/hashicorp/terraform-plugin-framework/list"
    "github.com/hashicorp/terraform-plugin-framework/provider"
    "github.com/hashicorp/terraform-plugin-framework/resource"
)

// Ensure provider defined types fully satisfy framework interfaces.
var (
    _ provider.Provider                       = &unifiProvider{}
    _ provider.ProviderWithEphemeralResources = &unifiProvider{}
    _ provider.ProviderWithListResources      = &unifiProvider{}
)

type unifiProvider struct{}

type unifiProviderModel struct {
    ApiKey         types.String `tfsdk:"api_key"`
    Username       types.String `tfsdk:"username"`
    Password       types.String `tfsdk:"password"`
    ApiUrl         types.String `tfsdk:"api_url"`
    Site           types.String `tfsdk:"site"`
    AllowInsecure  types.Bool   `tfsdk:"allow_insecure"`
    CloudConnector types.Bool   `tfsdk:"cloud_connector"`
    HardwareID     types.String `tfsdk:"hardware_id"`
}

// Client wraps the UniFi client with site information.
type Client struct {
    *unifi.ApiClient
    Site string
}

func New() provider.Provider {
    return &unifiProvider{}
}
```

### Provider Methods

```go
func (p *unifiProvider) Metadata(
    ctx context.Context,
    req provider.MetadataRequest,
    resp *provider.MetadataResponse,
) {
    resp.TypeName = "unifi"
}

func (p *unifiProvider) Schema(
    ctx context.Context,
    req provider.SchemaRequest,
    resp *provider.SchemaResponse,
) {
    resp.Schema = schema.Schema{
        Attributes: map[string]schema.Attribute{
            "username": schema.StringAttribute{
                Optional:    true,
                Description: "Username for UniFi controller",
            },
            "password": schema.StringAttribute{
                Optional:    true,
                Sensitive:   true,
                Description: "Password for UniFi controller",
            },
            "api_url": schema.StringAttribute{
                Optional:    true,
                Description: "URL of the UniFi controller",
            },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ubiquiti-community/terraform-provider-unifi](https://github.com/ubiquiti-community/terraform-provider-unifi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
