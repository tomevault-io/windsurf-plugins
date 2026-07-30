---
trigger: always_on
description: Incorrect/Correct pairs for all architecture rules. Use as a quick reference
---

# Architecture Rules — Expanded Reference

Incorrect/Correct pairs for all architecture rules. Use as a quick reference
when reviewing or writing Inertia.js + Rails code.

## Table of Contents

- [Rule 1: Server Owns Data (CRITICAL)](#rule-1-server-owns-data-critical)
- [Rule 2: Server Owns Auth (CRITICAL)](#rule-2-server-owns-auth-critical)
- [Rule 3: Use Form Component (CRITICAL)](#rule-3-use-form-component-critical)
- [Rule 4: Navigation (HIGH)](#rule-4-navigation-high)
- [Rule 5: Data Refresh (HIGH)](#rule-5-data-refresh-high)
- [Rule 6: Global Data (HIGH)](#rule-6-global-data-high)
- [Rule 7: Flash Messages (HIGH)](#rule-7-flash-messages-high)
- [Rule 8: Expensive Queries (MEDIUM)](#rule-8-expensive-queries-medium)
- [Rule 9: Persistent Layouts (MEDIUM)](#rule-9-persistent-layouts-medium)
- [Rule 10: Components as Renderers (MEDIUM)](#rule-10-components-as-renderers-medium)

---

## Rule 1: Server Owns Data (CRITICAL)

**Incorrect: useEffect + fetch for page data**
```tsx
// BAD — SPA pattern in an Inertia app
export default function Users() {
  const [users, setUsers] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data)
        setLoading(false)
      })
  }, [])

  if (loading) return <Spinner />
  return <UserList users={users} />
}
```

**Correct: Server provides data as props**
```ruby
# app/controllers/users_controller.rb
class UsersController < InertiaController
  def index
    render inertia: {
      users: User.all.as_json(only: [:id, :name, :email]),
    }
  end
end
```
```tsx
// app/frontend/pages/users/index.tsx — path matches controller/action
export default function Index({ users }: { users: User[] }) {
  return <UserList users={users} />
}
```

No loading state needed. No error handling for fetch. No race conditions.
The data arrives with the page, fully server-rendered and type-safe.

**Refreshing data without full page reload:**
```tsx
// Refresh only the users prop
router.reload({ only: ['users'] })

// After a mutation Rails redirects back and returns only the users prop
router.post('/users', {
  data: formData,
  only: ['users'],
})
```

---

## Rule 2: Server Owns Auth (CRITICAL)

**Incorrect: Client-side auth checks**
```tsx
// BAD — checking auth in React
export default function Dashboard() {
  const { auth } = usePage().props
  if (!auth.user) {
    router.visit('/login')
    return null
  }
  return <DashboardContent />
}
```

**Correct: Server handles auth, React trusts it**
```ruby
# app/controllers/dashboard_controller.rb
class DashboardController < InertiaController
  before_action :authenticate_user! # Redirect happens server-side

  def index
    render inertia: {
      stats: DashboardStats.for(Current.user),
    }
  end
end
```
```tsx
// If this component renders, user IS authenticated
export default function Index({ stats }: DashboardIndexProps) {
  return <DashboardContent stats={stats} />
}
```

If unauthenticated, the user never receives the page component.
The redirect happens server-side before any React code runs.

---

## Rule 3: Use Form Component (CRITICAL)

**Incorrect: Rolling your own form submission**
```tsx
// BAD — manual fetch/axios for forms
export default function CreateUser() {
  const [name, setName] = useState('')
  const [errors, setErrors] = useState({})
  const [submitting, setSubmitting] = useState(false)

  const handleSubmit = async (e) => {
    e.preventDefault()
    setSubmitting(true)
    try {
      const res = await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'X-CSRF-Token': ... },
        body: JSON.stringify({ name }),
      })
      if (!res.ok) setErrors(await res.json())
    } finally { setSubmitting(false) }
  }
}
```

**Correct: Inertia `<Form>` component**
```tsx
import { Form } from '@inertiajs/react'

export default function CreateUser() {
  return (
    <Form method="post" action="/users">
      {({ errors, processing }) => (
        <>
          <input type="text" name="name" />
          {errors.name && <span>{errors.name}</span>}
          <button type="submit" disabled={processing}>Create</button>
        </>
      )}
    </Form>
  )
}
```

`<Form>` handles: CSRF tokens, redirect following, error mapping, processing state,
file upload detection, scroll preservation, and browser history state — all
without manual `onChange` handlers or state management.

Use `useForm` hook only when you need programmatic control (dynamic fields,
external submit triggers, complex transforms, pre-populated edit forms).

---

## Rule 4: Navigation (HIGH)

**Incorrect: Traditional links or window.location**
```tsx
// BAD — causes full page reload, loses SPA behavior
<a href="/users">Users</a>
window.location.href = '/users'
```

**Correct: Inertia Link and router**
```tsx
import { Link, router } from '@inertiajs/react'

// Declarative
<Link href="/users">Users</Link>
<Link href="/users/1/edit" method="get">Edit</Link>

// Programmatic
router.visit('/users')
router.post('/users', { data: { name: 'John' } })

// With prefetching
<Link href="/users" prefetch cacheFor="30s">Users</Link>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inertia-rails/skills](https://github.com/inertia-rails/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
