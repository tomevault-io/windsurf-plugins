---
trigger: always_on
description: always use bun as package manager.
---


## package manager
always use bun as package manager.

## frontend

Should use mantine component all the time. we should use design token from mantine. 

We should not use tailwind classes. Strictly prohibited.

We should not use any external css framework. Strictly prohibited.

when working with jsx array, we should not use index as key. we should use the id of the item if available.

we should use `<meta>` in the page component.

we should use `typeCreateLoader` and `typeCreateActionRpc` to create the loader and action functions.

in `clientAction`, we should use `notifications` from `@mantine/notifications` to show notifications. we check the `actionData.status` against `StatusCode` to show the appropriate notification.

after using hook to update server data, we don't need to use `useRevalidator` to revalidate the page because react router will handle the revalidation automatically.

use interface but not type for component props except for `createRouteComponent` where we can use type.

in the loader, we should always throw ErrorResponse if fail, unless it is a non component route. in action, we return error response from `utils/responses` instead of throwing error.

we should not use `as` operator for type casting, especially in the UI components. we should fix the type in the upstream internal functions or context. 

avoid create complex props type for component, try infer type from loader data `Route.ComponentProps["loaderData"]` if possible.

we should break down the components so that each component only use one action hook. 

## Routing

we should use `href` from `react-router` to create hrefs for the routes.

in the `routes.ts`, we use `:id` to define the user id parameter. when it is not userId or in special case, we should always specify the type of `:noteId` or `:moduleId`. 

when a route has search param defined by loader Params, it is better to expose a getRouteUrl function to get the route url with the search params.

### styling

we should not hardcode `bg` color in mantine components unless it's absolutely necessary. we should provide comment to explain.

## contexts 

react router context should **not** import types from payload types. It should create types for itself to provide type stability for the frontend.

## testing 

every features on the backend should be tested using `bun:test`.

we should only use payload local api. we should not use anything related to nextjs.

testing should be simple and easy to read. for complex test, we better just skip and and test manually instead.

each test files should only have one describe block. each test file related to database, s3, redis or payload should have do a refresh using beforeAll. 

if you really want a completely isolated test, you should always create a new root describe block and its own beforeAll and AfterAll. beforeAll and AfterAll can have overrideAccess: true because they are not part of the test suite and are not affected by the test suite. in test case, we should try to provide the test user so that we can test the authentication unless otherwise specified.

when we need to test result ok and then do type narrowing, we should use `expect(result.ok).toBe(true); if (!result.ok) {` pattern for early return so we don't need to have nested if statements. 

for UI e2e testing, we should use AI agent to test browser. you should write test in `tests/e2e`. see `tests/e2e/login.test.yaml` for example. It use `describe`, `it`, `expect` and `beforeEach`, `AfterEach`, `Before`, `After` to write the test. we don't need to be very rigorous because the UI might change.

## error handling

we **should not** use try catch, especially not in internal functions which we use typescript result, it will be confusing and hard to manage. we should use typescript result to handle the error, using `Result.wrap`. When a function return a result, function should be named with `try` prefix like `tryFunction` or `trySomething`.

in the loader of react router, we should throw ErrorResponse rather than just error. 

when throwing error using typescript result, we should first create the error in `app/utils/error.ts` first. The error class should be used in the `transformError` in `Result.wrap` or `Result.try`.

## permissions and access control

all permissions should be calculated and checked in the server loader rather than in the component. the value is then passed to component through loader data. we should use `PermissionResult` from `server/utils/permissions.ts` to check the permission.

## Database 

when an operation is doing multiple mutation to the database, we should **always** use transaction.

database migration should always be non breaking and backward compatible.

payload local api return document with unknown depth. we should handle both case when depth is 0 and 1, value is either object or id (string). 

**strictly avoid** using `as` operator in internal functions. **No type casting allowed**.

use virtual field over depth. we would rather have a nested virtual field than have depth more than 2, because depth can cause infinite loop.

all globals collections should be defined in a single `server/collections/globals.ts`.


## json


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paideia-lms/Paideia](https://github.com/paideia-lms/Paideia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
