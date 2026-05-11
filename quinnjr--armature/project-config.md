---
trigger: always_on
description: Guidelines for GraphQL API development with Armature.
---


# GraphQL Development

Guidelines for GraphQL API development with Armature.

## GraphQL Crates

| Crate | Purpose |
|-------|---------|
| `armature-graphql` | GraphQL server with async-graphql |
| `armature-graphql-client` | GraphQL client for external APIs |

## Schema Definition

### Types and Objects

```rust
use async_graphql::*;

/// A user in the system.
#[derive(SimpleObject)]
pub struct User {
    /// Unique identifier
    pub id: ID,

    /// User's email address
    pub email: String,

    /// Display name
    pub name: String,

    /// Account creation timestamp
    pub created_at: DateTime<Utc>,
}

/// Extended user type with relations
#[derive(Default)]
pub struct UserType {
    pub user: User,
}

#[Object]
impl UserType {
    async fn id(&self) -> &ID {
        &self.user.id
    }

    async fn email(&self) -> &str {
        &self.user.email
    }

    async fn name(&self) -> &str {
        &self.user.name
    }

    /// User's posts (resolved lazily)
    async fn posts(&self, ctx: &Context<'_>) -> Result<Vec<Post>> {
        let loader = ctx.data::<DataLoader<PostLoader>>()?;
        let posts = loader.load_one(self.user.id.parse()?).await?;
        Ok(posts.unwrap_or_default())
    }

    /// User's role
    async fn role(&self, ctx: &Context<'_>) -> Result<Role> {
        let service = ctx.data::<RoleService>()?;
        service.get_user_role(self.user.id.parse()?).await
    }
}
```

### Input Types

```rust
/// Input for creating a new user.
#[derive(InputObject)]
pub struct CreateUserInput {
    /// User's email (must be unique)
    #[graphql(validator(email))]
    pub email: String,

    /// User's password (min 8 characters)
    #[graphql(validator(min_length = 8))]
    pub password: String,

    /// Display name
    #[graphql(validator(min_length = 1, max_length = 100))]
    pub name: String,
}

/// Input for updating a user.
#[derive(InputObject)]
pub struct UpdateUserInput {
    /// New email address
    #[graphql(validator(email))]
    pub email: Option<String>,

    /// New display name
    #[graphql(validator(min_length = 1, max_length = 100))]
    pub name: Option<String>,
}

/// Filter options for listing users.
#[derive(InputObject, Default)]
pub struct UserFilter {
    /// Filter by name (contains)
    pub name: Option<String>,

    /// Filter by role
    pub role: Option<Role>,

    /// Filter by creation date (after)
    pub created_after: Option<DateTime<Utc>>,
}
```

### Enums

```rust
/// User role in the system.
#[derive(Enum, Copy, Clone, Eq, PartialEq)]
pub enum Role {
    /// Regular user
    User,
    /// Moderator with elevated privileges
    Moderator,
    /// Administrator with full access
    Admin,
}

/// Sort direction.
#[derive(Enum, Copy, Clone, Eq, PartialEq, Default)]
pub enum SortDirection {
    #[default]
    Asc,
    Desc,
}
```

## Query Implementation

```rust
pub struct QueryRoot;

#[Object]
impl QueryRoot {
    /// Get the currently authenticated user.
    async fn me(&self, ctx: &Context<'_>) -> Result<Option<User>> {
        let user = ctx.data_opt::<AuthenticatedUser>();
        match user {
            Some(auth) => {
                let service = ctx.data::<UserService>()?;
                service.find_by_id(auth.user_id).await
            }
            None => Ok(None),
        }
    }

    /// Get a user by ID.
    async fn user(&self, ctx: &Context<'_>, id: ID) -> Result<Option<User>> {
        let service = ctx.data::<UserService>()?;
        service.find_by_id(id.parse()?).await
    }

    /// List users with optional filtering and pagination.
    async fn users(
        &self,
        ctx: &Context<'_>,
        #[graphql(default)] filter: UserFilter,
        #[graphql(default = 1)] page: u32,
        #[graphql(default = 20, validator(maximum = 100))] per_page: u32,
    ) -> Result<UserConnection> {
        let service = ctx.data::<UserService>()?;
        let (users, total) = service.list(filter, page, per_page).await?;

        Ok(UserConnection {
            nodes: users,
            page_info: PageInfo {
                page,
                per_page,
                total,
                has_next_page: (page * per_page) < total as u32,
                has_previous_page: page > 1,
            },
        })
    }

    /// Search users by name or email.
    async fn search_users(
        &self,
        ctx: &Context<'_>,
        query: String,
        #[graphql(default = 10, validator(maximum = 50))] limit: u32,
    ) -> Result<Vec<User>> {
        let service = ctx.data::<UserService>()?;
        service.search(&query, limit).await
    }
}
```

## Mutation Implementation

```rust
pub struct MutationRoot;

#[Object]
impl MutationRoot {
    /// Create a new user account.
    async fn create_user(
        &self,
        ctx: &Context<'_>,
        input: CreateUserInput,
    ) -> Result<User> {
        let service = ctx.data::<UserService>()?;

        // Check for existing email
        if service.email_exists(&input.email).await? {
            return Err(Error::new("Email already registered"));
        }

        service.create(input).await
    }

    /// Update the current user's profile.
    #[graphql(guard = "AuthGuard")]
    async fn update_profile(
        &self,
        ctx: &Context<'_>,
        input: UpdateUserInput,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
