---
trigger: always_on
description: When you are building new react components, or react pages.
---


# TDD Approach

## Available debug tools you can run:
- `monitor-logs.sh` for seeing RLS or Postgres issues.

## Core Philosophy

We follow an integration-first TDD approach that emphasizes testing React components, RLS policies, and database operations together. This approach ensures our tests reflect real-world usage patterns.

We have several files that are powering our RLS + integration tests.
[setup.ts](mdc:apps/web/integration/setup.ts)
[test-utils.ts](mdc:apps/web/integration/test-utils.ts)

Database types:
[database-functions.types.ts](mdc:packages/supabase/src/types/database-functions.types.ts)

Always use tanstack query and never useEffect, you must use vittest and not jest.

If you are using tanstack queries, assume that the toast notification has already been included for that hook and don't add an extra onSuccess or onError toast message. You can still include custom logic that will be execued on them though (after the hook has run).

We have factories that are in the [index.ts](mdc:packages/supabase/src/factories/index.ts), i.e. [member.factory.ts](mdc:packages/supabase/src/factories/member.factory.ts) or [role.factory.ts](mdc:packages/supabase/src/factories/role.factory.ts) or [user.factory.ts](mdc:packages/supabase/src/factories/user.factory.ts)
You can create new factories, if they are missing.
One recommendation is to try using the module folder first CRUD rather than replicating that CRUD, the only reason why we did this for organisations and org_members and team_members is because we had a custom sign up. But just assume you can re-use the CRUD.

Core Philosophy
- Ask if the code requires roles or not.
- Handle all happy critical paths first.
- Handle all edge cases.
- Write CRUD tests for RLS (regardless of whether or not roles will be utilized)

There are a couple of scenarios so far:
- Client/Team
- Admin/Member

We always make react-query hooks for the module crud. I.e. [users.react.ts](mdc:packages/supabase/src/module/users.react.ts) or [teams.react.ts](mdc:packages/supabase/src/module/teams.react.ts)

Always remember to render with renderWithProviders from `import { renderWithProviders } from "../test-utils";` as this wraps the app in our [ClientSideProviders.tsx](mdc:apps/web/components/providers/ClientSideProviders.tsx), we are using react hot toast. And you can find an example of a fail notification in the error:
```export async function updateOrganizationMember({
  supabase,
  memberId,
  member,
}: {
  supabase: SupabaseClient;
  memberId: string;
  member: OrganizationMemberUpdate;
}): Promise<OrganizationMember> {
  try {
    const { data, error } = await supabase
      .from("organization_members")
      .update(member)
      .eq("id", memberId)
      .select()
      .single();

    if (error) throw error;
    if (!data) throw new Error("No data returned after update");
    return data;
  } catch (error) {
    throw new OrganizationMemberOperationError(
      "Update Organization Member",
      `Failed to update organization member with ID: ${memberId}`,
      "Unable to update organization member. Please try again.",
      SupabaseErrorCode.UPDATE_FAILED,
      error
    );
  }
}
```

For the success toast notification, we will need to add those manually though.

It is also worth noting that we are working on the lowest level of granularity for our permissions, i.e 
-- Create new team policies using helper functions
CREATE POLICY "teams_view" ON public.teams
    FOR SELECT TO authenticated
    USING (
      is_org_member(organization_id)  -- user must be in the org to see its teams
    );

CREATE POLICY "teams_insert" ON public.teams
    FOR INSERT TO authenticated
    WITH CHECK (
      -- Must have manage_organization permission to create teams
      has_org_permission(organization_id, 'manage_organization'::permission_action)
    );

We use custom RLS helpers [20240124000015_fn_rls_helpers.sql](mdc:packages/supabase/supabase/migrations/20240124000015_fn_rls_helpers.sql) to keep everything tidy.

The permissions are coming from [20240124000004_tbl_create_permissions.sql](mdc:packages/supabase/supabase/migrations/20240124000004_tbl_create_permissions.sql) and other sql files.

We are using this pattern in [README.md](mdc:apps/web/features/authorization/README.md), so basically higher order react components to hide or disable buttons given the different roles.

Have a look at these files to understand it such as:
[get-member-roles.ts](mdc:apps/web/features/authorization/actions/get-member-roles.ts)
[withRoleCheck.tsx](mdc:apps/web/features/authorization/components/withRoleCheck.tsx)
[use-member-roles.ts](mdc:apps/web/features/authorization/hooks/use-member-roles.ts)
[useRoleCheck.ts](mdc:apps/web/features/authorization/hooks/useRoleCheck.ts)
[roles.ts](mdc:apps/web/features/authorization/lib/roles.ts)


The other thing to note is our react-query set up using react server pages, which looks like this:
[page.tsx](mdc:apps/web/app/(application)/org/[orgId]/[teamId]/page.tsx)
And this for the client side:
[Dashboard.tsx](mdc:apps/web/components/dashboard/Dashboard.tsx)

## Running tests

- `pnpm test:integration` for `.integration.ts` test suite.
- `pnpm test` for smaller, light weight component tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Just-Understanding-Data-Ltd/supabase-tdd-boilerplate](https://github.com/Just-Understanding-Data-Ltd/supabase-tdd-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
