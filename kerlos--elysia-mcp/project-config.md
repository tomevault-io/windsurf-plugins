---
trigger: always_on
description: TITLE: Elysia Supported Schema Validation Types
---

TITLE: Elysia Supported Schema Validation Types
DESCRIPTION: Elysia provides declarative schema support for various parts of an HTTP request and response, enabling robust validation and automatic OpenAPI generation.
SOURCE: https://github.com/elysiajs/documentation/blob/main/docs/essential/validation.md#_snippet_3

LANGUAGE: APIDOC
CODE:
```
Schema Types:
  Body: Validate an incoming HTTP Message
  Query: Query string or URL parameter
  Params: Path parameters
  Headers: Headers of the request
  Cookie: Cookie of the request
  Response: Response of the request
```

----------------------------------------

TITLE: Elysia.js User Authentication and Session Management Service
DESCRIPTION: Provides a comprehensive user authentication service built with Elysia.js. It defines in-memory state for users and sessions, models for sign-in credentials and cookies, a custom macro for authentication checks, and routes for user sign-up, sign-in, and sign-out, including password hashing and session token management.
SOURCE: https://github.com/elysiajs/documentation/blob/main/docs/tutorial.md#_snippet_41

LANGUAGE: typescript
CODE:
```
// @errors: 2538
import { Elysia, t } from 'elysia'

export const userService = new Elysia({ name: 'user/service' })
    .state({
        user: {} as Record<string, string>,
        session: {} as Record<number, string>
    })
    .model({
        signIn: t.Object({
            username: t.String({ minLength: 1 }),
            password: t.String({ minLength: 8 })
        }),
        session: t.Cookie(
            {
                token: t.Number()
            },
            {
                secrets: 'seia'
            }
        ),
        optionalSession: t.Cookie(
            {
                token: t.Optional(t.Number())
            },
            {
                secrets: 'seia'
            }
        )
    })
    .macro({
        isSignIn(enabled: boolean) {
            if (!enabled) return

            return {
                beforeHandle({
                    status,
                    cookie: { token },
                    store: { session }
                }) {
                    if (!token.value)
                        return status(401, {
                            success: false,
                            message: 'Unauthorized'
                        })

                    const username = session[token.value as unknown as number]

                    if (!username)
                        return status(401, {
                            success: false,
                            message: 'Unauthorized'
                        })
                }
            }
        }
    })

export const getUserId = new Elysia()
    .use(userService)
    .guard({
        isSignIn: true,
        cookie: 'session'
    })
    .resolve(({ store: { session }, cookie: { token } }) => ({
        username: session[token.value]
    }))
    .as('scoped')

export const user = new Elysia({ prefix: '/user' })
    .use(userService)
    .put(
        '/sign-up',
        async ({ body: { username, password }, store, status }) => {
            if (store.user[username])
                return status(400, {
                    success: false,
                    message: 'User already exists'
                })

            store.user[username] = await Bun.password.hash(password)

            return {
                success: true,
                message: 'User created'
            }
        },
        {
            body: 'signIn'
        }
    )
    .post(
        '/sign-in',
        async ({
            store: { user, session },
            status,
            body: { username, password },
            cookie: { token }
        }) => {
            if (
                !user[username] ||
                !(await Bun.password.verify(password, user[username]))
            )
                return status(400, {
                    success: false,
                    message: 'Invalid username or password'
                })

            const key = crypto.getRandomValues(new Uint32Array(1))[0]
            session[key] = username
            token.value = key

            return {
                success: true,
                message: `Signed in as ${username}`
            }
        },
        {
            body: 'signIn',
            cookie: 'optionalSession'
        }
    )
    .get(
        '/sign-out',
        ({ cookie: { token } }) => {
            token.remove()

            return {
                success: true,
                message: 'Signed out'
            }
        },
        {
            cookie: 'optionalSession'
        }
    )
    .use(getUserId)
    .get('/profile', ({ username }) => ({
        success: true,

```

----------------------------------------

TITLE: Basic Elysia Server with Routes, Files, Streams, and WebSockets
DESCRIPTION: This snippet sets up a basic Elysia server demonstrating various functionalities: a simple 'Hello World' GET route, serving a static file, streaming responses, and a WebSocket endpoint for real-time communication. The server listens on port 3000.
SOURCE: https://github.com/elysiajs/documentation/blob/main/docs/index.md#_snippet_4

LANGUAGE: typescript
CODE:
```
import { Elysia, file } from 'elysia'

new Elysia()
	.get('/', 'Hello World')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kerlos/elysia-mcp](https://github.com/kerlos/elysia-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
