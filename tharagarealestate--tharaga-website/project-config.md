---
trigger: always_on
description: Row Level Security (RLS) is PostgreSQL's built-in security feature that allows you to control which rows users can access in your database tables. In Supabase, RLS is essential for protecting your data.
---

# Supabase Row Level Security (RLS) Policies

## Overview
Row Level Security (RLS) is PostgreSQL's built-in security feature that allows you to control which rows users can access in your database tables. In Supabase, RLS is essential for protecting your data.

## RLS Fundamentals

### Golden Rules
1. **Enable RLS on all tables** - Especially tables containing user data
2. **Test policies thoroughly** - Ensure users can only access appropriate data
3. **Use authenticated/anon roles** - Leverage Supabase's built-in roles
4. **Keep policies simple** - Complex policies are harder to debug and maintain
5. **Consider performance** - Policies are applied to every query

### Enable RLS
```sql
-- Enable RLS on a table
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;

-- Disable RLS (use with caution!)
-- ALTER TABLE table_name DISABLE ROW LEVEL SECURITY;

-- Force RLS even for table owner
ALTER TABLE table_name FORCE ROW LEVEL SECURITY;
```

## Policy Structure

### Basic Syntax
```sql
CREATE POLICY "policy_name"
ON table_name
FOR operation  -- SELECT, INSERT, UPDATE, DELETE, ALL
TO role        -- authenticated, anon, service_role
USING (condition)      -- For SELECT and DELETE
WITH CHECK (condition); -- For INSERT and UPDATE
```

### Policy Components
- **Policy Name**: Descriptive name in quotes
- **Operation**: SELECT, INSERT, UPDATE, DELETE, or ALL
- **Role**: authenticated, anon, service_role
- **USING**: Row-level security check for reading/deleting
- **WITH CHECK**: Row-level security check for creating/updating

## Common Policy Patterns

### User-Owned Resources
Users can only access their own rows:

```sql
-- Read own rows
CREATE POLICY "Users can view own posts"
ON posts
FOR SELECT
TO authenticated
USING (auth.uid() = user_id);

-- Insert own rows
CREATE POLICY "Users can create own posts"
ON posts
FOR INSERT
TO authenticated
WITH CHECK (auth.uid() = user_id);

-- Update own rows
CREATE POLICY "Users can update own posts"
ON posts
FOR UPDATE
TO authenticated
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);

-- Delete own rows
CREATE POLICY "Users can delete own posts"
ON posts
FOR DELETE
TO authenticated
USING (auth.uid() = user_id);
```

### Public Read, Authenticated Write
Anyone can read, only authenticated users can write:

```sql
-- Public read
CREATE POLICY "Anyone can view published posts"
ON posts
FOR SELECT
TO anon, authenticated
USING (status = 'published');

-- Authenticated write
CREATE POLICY "Authenticated users can create posts"
ON posts
FOR INSERT
TO authenticated
WITH CHECK (true);
```

### Role-Based Access
Different access based on user role:

```sql
-- Add role column to profiles
-- user_role: 'admin', 'moderator', 'user'

-- Admins can view all
CREATE POLICY "Admins can view all posts"
ON posts
FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE profiles.user_id = auth.uid()
    AND profiles.user_role = 'admin'
  )
);

-- Moderators can update
CREATE POLICY "Moderators can update posts"
ON posts
FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE profiles.user_id = auth.uid()
    AND profiles.user_role IN ('admin', 'moderator')
  )
);
```

### Team/Organization Based
Access based on team membership:

```sql
-- Users can view posts in their organization
CREATE POLICY "Users can view organization posts"
ON posts
FOR SELECT
TO authenticated
USING (
  organization_id IN (
    SELECT organization_id
    FROM organization_members
    WHERE user_id = auth.uid()
  )
);

-- Team members can update
CREATE POLICY "Team members can update posts"
ON posts
FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM team_members
    WHERE team_members.team_id = posts.team_id
    AND team_members.user_id = auth.uid()
  )
)
WITH CHECK (
  EXISTS (
    SELECT 1 FROM team_members
    WHERE team_members.team_id = posts.team_id
    AND team_members.user_id = auth.uid()
  )
);
```

### Relationship-Based Access
Access based on relationships:

```sql
-- Users can view posts from people they follow
CREATE POLICY "Users can view posts from followed users"
ON posts
FOR SELECT
TO authenticated
USING (
  user_id IN (
    SELECT followed_user_id
    FROM follows
    WHERE follower_user_id = auth.uid()
  )
  OR user_id = auth.uid() -- Also own posts
);
```

### Time-Based Access
Access based on timestamps:

```sql
-- Users can only edit posts within 1 hour of creation
CREATE POLICY "Users can edit posts within 1 hour"
ON posts
FOR UPDATE
TO authenticated
USING (
  auth.uid() = user_id
  AND created_at > NOW() - INTERVAL '1 hour'
);

-- Scheduled content access
CREATE POLICY "View published and scheduled posts"
ON posts
FOR SELECT
TO authenticated
USING (
  (status = 'published' AND published_at <= NOW())
  OR (user_id = auth.uid())
);
```

### Hierarchical Access
Parent-child relationship access:

```sql
-- Users can view comments on posts they can view
CREATE POLICY "Users can view comments on accessible posts"
ON comments
FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM posts
    WHERE posts.id = comments.post_id
    AND (
      posts.status = 'published'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tharagarealestate) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
