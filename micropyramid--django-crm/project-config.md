---
trigger: always_on
description: This page describes patterns that hold across most of the API: how the base URL is put together,
---

# Conventions

This page describes patterns that hold across most of the API: how the base URL is put together,
how a client authenticates, how list endpoints paginate and filter, and what a response body looks
like. It is deliberately drawn from the code that is actually there (a real pagination class, a
real list view, the real `REST_FRAMEWORK` settings) rather than from what a typical DRF API "should"
look like. Several of these conventions are less uniform than that would suggest, and the
differences matter to anyone integrating against them.

## Base URL

Every endpoint in this section, and every endpoint in [Endpoint index](endpoint-index.md), is
mounted under `/api/` at the Django root (`backend/crm/urls.py`: `path("api/",
include("common.app_urls", ...))`), which in turn includes `backend/common/urls.py`, the file most
of this section's page references point at, at the same, empty prefix. Individual apps are mounted
under their own segment from there, for example `path("leads/", include("leads.urls", ...))`, so a
lead endpoint is `/api/leads/…`.

Locally this is `http://localhost:8000/api/` (the default in the Docker quick start's
`.env.docker`, and what `PUBLIC_DJANGO_API_URL` points the frontend at. See
[Docker quick start](../getting-started/docker-quick-start.md)). In a self-hosted deployment it is
whatever host you put `DOMAIN_NAME` / `ALLOWED_HOSTS` on, still under `/api/`.

## Authentication header

Three credential types authenticate a request, tried by `common.middleware.get_company.GetProfileAndOrg`
in this order and, independently, by the DRF authentication classes in `DEFAULT_AUTHENTICATION_CLASSES`
(`backend/crm/settings.py`, `common.pat_auth.PATAuthentication`,
`rest_framework_simplejwt.authentication.JWTAuthentication`, `common.external_auth.APIKeyAuthentication`):

```
Authorization: Bearer <jwt-access-token>          # interactive sign-in (see authentication.md)
Authorization: Bearer bcrm_pat_...                 # personal access token
Token: bcrm_pat_...                                 # personal access token, alternate header
Token: <org-api-key>                                # organization API key
```

A personal access token is recognized by its `bcrm_pat_` prefix and can be presented either as a
bearer token or in the `Token` header (`backend/common/pat_auth.py`, `_extract_raw`); an organization
API key is only ever read from `Token` (`backend/common/external_auth.py`,
`APIKeyAuthentication.authenticate`). See [Authentication](authentication.md) for how a JWT is
obtained and [Tokens and API keys](tokens-and-api-keys.md) for the other two. Which identity a
request authenticates as, and which org it is scoped to, is never something the request supplies
directly. It is derived from whichever of these three the server can validate.

The two non-interactive credentials are bounded in ways a signed-in session is not. A personal
access token is limited to its `scopes` (`<resource>:<action>`, enforced in middleware before the
view runs; an empty list means unrestricted). The organization API key is read-only. Neither may
reach `/api/profile/tokens/`, `/api/org/tokens/` or `/api/org/api-key/` at all: credential
management requires an interactive sign-in. See
[Tokens and API keys](tokens-and-api-keys.md#scopes).

One thing the generated OpenAPI schema does *not* reflect accurately: a large minority of
operations (112 of the schema's 350, about a third) declare an `org` header parameter
(`organization_params_in_header`, defined in `backend/common/swagger_params.py` and copied into the
`swagger_params.py` of six other apps: `leads`, `cases`, `opportunity`, `contacts`, `accounts` and
`tasks`). Nothing in the
request-handling code reads a header by that name. Org context comes only from the JWT's `org_id`
claim, the PAT's owning profile, or the API key's associated org, as above. Sending an `org` header
has no effect; do not rely on it.

## Pagination

List endpoints paginate with `rest_framework.pagination.LimitOffsetPagination`
(`DEFAULT_PAGINATION_CLASS` in `backend/crm/settings.py`), using the query parameters `limit` and
`offset`. The default page size is `PAGE_SIZE = 10` from the same settings block; there is no
`max_limit` set on the pagination class, so a caller can request a larger page and get it.

That default pagination class is not applied automatically, though. There is no
`generics.ListAPIView` anywhere in this codebase. Every list endpoint is a hand-written `APIView`
that mixes `LimitOffsetPagination` in directly and calls `self.paginate_queryset(...)` itself, for
example:

```python
class LeadListView(APIView, LimitOffsetPagination):
    ...
    def get_context_data(self, **kwargs):
        ...
        results_leads_open = self.paginate_queryset(
            queryset_open.distinct(), self.request, view=self
        )
```

(`backend/leads/views/lead_views.py:51`, `:167`.) The same shape repeats in `common/views/user_views.py`,
`common/views/team_views.py`, `common/views/document_views.py`, `common/views/tags_views.py`,
`cases/views.py`, `contacts/views.py`, `accounts/views.py`, `opportunity/views/opportunity_views.py`,
`tasks/views/task_views.py` and `invoices/api_views.py`, among others.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicroPyramid/Django-CRM](https://github.com/MicroPyramid/Django-CRM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
