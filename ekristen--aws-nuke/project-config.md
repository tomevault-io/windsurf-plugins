---
trigger: always_on
description: This guide provides step-by-step instructions for adding a new AWS resource to aws-nuke. It covers resource structure, testing strategies, and best practices.
---

# Writing a New Resource for AWS Nuke

This guide provides step-by-step instructions for adding a new AWS resource to aws-nuke. It covers resource structure, testing strategies, and best practices.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Resource Structure](#resource-structure)
- [Step-by-Step Guide](#step-by-step-guide)
- [Testing](#testing)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)

## Overview

Resources in aws-nuke represent AWS resources that can be listed and deleted. Each resource must implement specific interfaces and follow established patterns for consistency and maintainability.

## Prerequisites

Before creating a new resource, ensure you have:

1. Go 1.21 or later installed
2. AWS SDK v2 knowledge (we use AWS SDK v2, not v1)
3. Familiarity with the AWS service you're implementing
4. golangci-lint installed for linting
5. Read the [CONTRIBUTING.md](CONTRIBUTING.md) guide

## Resource Structure

Every resource consists of:

1. **Resource File** - `resources/<service-name>-<resource-name>.go`
2. **Mock Tests** - `resources/<service-name>-<resource-name>_mock_test.go` (using gomock)
3. **Integration Tests** - `resources/<service-name>-<resource-name>_test.go` (optional but preferred)

### Basic Resource Template

```go
package resources

import (
	"context"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/service/<servicename>"

	"github.com/ekristen/libnuke/pkg/registry"
	"github.com/ekristen/libnuke/pkg/resource"
	"github.com/ekristen/libnuke/pkg/types"

	"github.com/ekristen/aws-nuke/v3/pkg/nuke"
)

const MyResourceResource = "MyResource"

func init() {
	registry.Register(&registry.Registration{
		Name:     MyResourceResource,
		Scope:    nuke.Account,
		Resource: &MyResource{},
		Lister:   &MyResourceLister{},
	})
}

type MyResourceLister struct{}

func (l *MyResourceLister) List(ctx context.Context, o interface{}) ([]resource.Resource, error) {
	opts := o.(*nuke.ListerOpts)
	svc := servicename.NewFromConfig(*opts.Config)
	var resources []resource.Resource

	// List resources using pagination
	params := &servicename.ListMyResourcesInput{
		MaxResults: aws.Int32(100),
	}

	paginator := servicename.NewListMyResourcesPaginator(svc, params)

	for paginator.HasMorePages() {
		resp, err := paginator.NextPage(ctx)
		if err != nil {
			return nil, err
		}

		for _, item := range resp.Items {
			resources = append(resources, &MyResource{
				svc:  svc,
				Name: item.Name,
			})
		}
	}

	return resources, nil
}

type MyResource struct {
	svc  *servicename.Client
	Name *string
}

func (r *MyResource) Remove(ctx context.Context) error {
	_, err := r.svc.DeleteMyResource(ctx, &servicename.DeleteMyResourceInput{
		Name: r.Name,
	})
	return err
}

func (r *MyResource) Properties() types.Properties {
	return types.NewPropertiesFromStruct(r)
}

func (r *MyResource) String() string {
	return *r.Name
}
```

## Step-by-Step Guide

### 1. Create the Resource File

Create a new file in the `resources/` directory following the naming convention:
- Format: `<service>-<resource>.go`
- Example: `eks-clusters.go`, `inspector2.go`

### 2. Define the Resource Constant

```go
const EKSClusterResource = "EKSCluster"
```

The constant should match the resource name used in configuration files.

### 3. Implement the Lister

The lister is responsible for discovering all resources of this type in an AWS account.

#### Simple Lister Example

```go
type EKSClusterLister struct{}

func (l *EKSClusterLister) List(ctx context.Context, o interface{}) ([]resource.Resource, error) {
	opts := o.(*nuke.ListerOpts)
	svc := eks.NewFromConfig(*opts.Config)
	var resources []resource.Resource

	params := &eks.ListClustersInput{
		MaxResults: aws.Int32(100),
	}

	paginator := eks.NewListClustersPaginator(svc, params)

	for paginator.HasMorePages() {
		resp, err := paginator.NextPage(ctx)
		if err != nil {
			return nil, err
		}

		for _, cluster := range resp.Clusters {
			resources = append(resources, &EKSCluster{
				svc:  svc,
				Name: aws.String(cluster),
			})
		}
	}

	return resources, nil
}
```

**Key Points:**
- Always use context for AWS SDK calls
- Use paginators when available to handle large result sets
- Convert AWS SDK types to your resource struct
- Handle errors appropriately

### 4. Implement the Resource Struct

The resource struct holds the data for a single resource instance.

```go
type EKSCluster struct {
	svc        *eks.Client
	Name       *string
	CreatedAt  *time.Time
	Tags       map[string]string
	settings   *libsettings.Setting
	protection *bool
}
```

**Required Fields:**
- Service client (to make deletion calls)
- Identifier fields (name, ID, ARN, etc.)

**Optional Fields:**
- Timestamps (CreatedAt, LastUpdatedTime)
- Tags
- Settings (if the resource supports settings)
- Status information

### 5. Implement Required Methods

Every resource must implement these methods:

#### Remove Method

```go
func (r *EKSCluster) Remove(ctx context.Context) error {
	_, err := r.svc.DeleteCluster(ctx, &eks.DeleteClusterInput{
		Name: r.Name,
	})
	return err
}
```

#### Properties Method

```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekristen/aws-nuke](https://github.com/ekristen/aws-nuke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
