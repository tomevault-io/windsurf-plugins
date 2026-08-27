---
trigger: always_on
description: Rules for persistent datastore logic
---

# Datastore Implementation Guidelines

## Overview
Our datastore layer uses [Ent](mdc:https:/entgo.io) as the ORM and follows consistent patterns for data access. This document outlines the key patterns and best practices to follow when working with the datastore package.

## General Architecture

- **Package**: All datastore code lives in the `internal/datastore` package
- **Models**: We use models from the `internal/models` package for input/output, never exposing Ent types directly
- **Client**: The datastore uses an Ent client for database operations
- **Logging**: We use `zap.Logger` for structured logging throughout the datastore

```go
// Base Datastore struct
type Datastore struct {
    dbClient *ent.Client
    logger   *zap.Logger
}

func NewDatastore(dbClient *ent.Client, logger *zap.Logger) *Datastore {
    return &Datastore{
        dbClient: dbClient,
        logger:   logger,
    }
}
```

## File Organization

Each entity type should have its own file in the datastore package. For example:
- `content_idea.go` - Content idea based on trending news and posts related to a niche
- `content_brief.go` - A content development brief including SEO plan
- `interview_question.go` - An interview question and the user's response for content development

## Standard Methods

Each entity type should implement the following standard methods:

1. **Model Conversion Function**: Convert from Ent type to model type
```go
// Convert from Ent entity to model
func toContentIdeaModel(e *ent.ContentIdea) *models.ContentIdea {
    return &models.ContentIdea{
        ID:        e.ID,
        ProjectID: e.Edges.Project.ID,
        Title:     e.Title,
        Summary:   e.Summary,
        SourceURL: e.SourceURL,
        Approved:  e.Approved,
        CreatedAt: e.CreatedAt,
        UpdatedAt: e.UpdatedAt,
    }
}
```

2. **Create Method**: Single entity creation with transaction and authorization
```go
func (d *Datastore) CreateContentIdea(ctx context.Context, userID uuid.UUID, contentIdea models.ContentIdea) (*models.ContentIdea, error) {
    // Start transaction
    tx, err := d.dbClient.Tx(ctx)
    if err != nil {
        d.logger.Error("failed to start transaction", zap.Error(err))
        return nil, err
    }

    // Rollback in case of error
    defer func() {
        if v := recover(); v != nil {
            tx.Rollback()
            panic(v)
        }
    }()

    // Check if project exists and belongs to the user
    projectExists, err := tx.Project.Query().
        Where(
            project.ID(contentIdea.ProjectID),
            project.HasOwnerWith(
                user.ID(userID),
            ),
        ).
        Exist(ctx)

    if err != nil {
        d.logger.Error("failed to query project", zap.Error(err))
        if rerr := tx.Rollback(); rerr != nil {
            d.logger.Error("failed to rollback transaction", zap.Error(rerr))
        }
        return nil, err
    }

    if !projectExists {
        d.logger.Error("project not found or user not authorized",
            zap.String("project_id", contentIdea.ProjectID.String()),
            zap.String("user_id", userID.String()))
        if rerr := tx.Rollback(); rerr != nil {
            d.logger.Error("failed to rollback transaction", zap.Error(rerr))
        }
        return nil, ErrProjectNotFound
    }

    // Create entity
    entContentIdea, err := tx.ContentIdea.Create().
        SetTitle(contentIdea.Title).
        SetSummary(contentIdea.Summary).
        SetSourceURL(contentIdea.SourceURL).
        SetApproved(contentIdea.Approved).
        SetProjectID(contentIdea.ProjectID).
        Save(ctx)
        
    if err != nil {
        d.logger.Error("failed to create content idea", zap.Error(err))
        if rerr := tx.Rollback(); rerr != nil {
            d.logger.Error("failed to rollback transaction", zap.Error(rerr))
        }
        return nil, err
    }
    
    // Load relationships needed for model conversion
    entContentIdea, err = tx.ContentIdea.Query().
        Where(contentidea.ID(entContentIdea.ID)).
        WithProject().
        Only(ctx)

    if err != nil {
        d.logger.Error("failed to load project relationship", zap.Error(err))
        if rerr := tx.Rollback(); rerr != nil {
            d.logger.Error("failed to rollback transaction", zap.Error(rerr))
        }
        return nil, err
    }
    
    // Commit transaction
    if err := tx.Commit(); err != nil {
        d.logger.Error("failed to commit transaction", zap.Error(err))
        return nil, err
    }
    
    // Return model
    return toContentIdeaModel(entContentIdea), nil
}
```

3. **Query Methods**: Methods to retrieve data with flexible filtering and pagination
```go
func (d *Datastore) ListContentIdeas(ctx context.Context, userID uuid.UUID, pageNum, pageSize int, filters models.ContentIdeaFilters) (*models.PaginatedResponse, error) {
    // Start transaction
    tx, err := d.dbClient.Tx(ctx)
    if err != nil {
        d.logger.Error("failed to start transaction", zap.Error(err))
        return nil, err
    }

    // Rollback in case of error
    defer func() {
        if v := recover(); v != nil {
            tx.Rollback()
            panic(v)
        }
    }()

    // Build query with user authorization
    query := tx.ContentIdea.Query().
        Where(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theimaginaryfoundation/what-iff](https://github.com/theimaginaryfoundation/what-iff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
