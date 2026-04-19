---
trigger: always_on
description: validateSearch: zodValidator(searchSchema),
---

# ActiveLinkOptions type

The `ActiveLinkOptions` type extends the [`LinkOptions`](../LinkOptionsType.md) type and contains additional options that can be used to describe how a link should be styled when it is active.

```tsx
type ActiveLinkOptions = LinkOptions & {
  activeProps?:
    | React.AnchorHTMLAttributes<HTMLAnchorElement>
    | (() => React.AnchorHTMLAttributes<HTMLAnchorElement>)
  inactiveProps?:
    | React.AnchorHTMLAttributes<HTMLAnchorElement>
    | (() => React.AnchorHTMLAttributes<HTMLAnchorElement>)
}
```

## ActiveLinkOptions properties

The `ActiveLinkOptions` object accepts/contains the following properties:

### `activeProps`

- `React.AnchorHTMLAttributes<HTMLAnchorElement>`
- Optional
- The props that will be applied to the anchor element when the link is active

### `inactiveProps`

- Type: `React.AnchorHTMLAttributes<HTMLAnchorElement>`
- Optional
- The props that will be applied to the anchor element when the link is inactive

# AsyncRouteComponent type

The `AsyncRouteComponent` type is used to describe a code-split route component that can be preloaded using a `component.preload()` method.

```tsx
type AsyncRouteComponent<TProps> = SyncRouteComponent<TProps> & {
  preload?: () => Promise<void>
}
```

# FileRoute class

> [!CAUTION]
> This class has been deprecated and will be removed in the next major version of TanStack Router.
> Please use the [`createFileRoute`](../createFileRouteFunction.md) function instead.

The `FileRoute` class is a factory that can be used to create a file-based route instance. This route instance can then be used to automatically generate a route tree with the `tsr generate` and `tsr watch` commands.

## `FileRoute` constructor

The `FileRoute` constructor accepts a single argument: the `path` of the file that the route will be generated for.

### Constructor options

- Type: `string` literal
- Required, but **automatically inserted and updated by the `tsr generate` and `tsr watch` commands**.
- The full path of the file that the route will be generated from.

### Constructor returns

- An instance of the `FileRoute` class that can be used to create a route.

## `FileRoute` methods

The `FileRoute` class implements the following method(s):

### `.createRoute` method

The `createRoute` method is a method that can be used to configure the file route instance. It accepts a single argument: the `options` that will be used to configure the file route instance.

#### .createRoute options

- Type: `Omit<RouteOptions, 'getParentRoute' | 'path' | 'id'>`
- [`RouteOptions`](../RouteOptionsType.md)
- Optional
- The same options that are available to the `Route` class, but with the `getParentRoute`, `path`, and `id` options omitted since they are unnecessary for file-based routing.

#### .createRoute returns

A [`Route`](../RouteType.md) instance that can be used to configure the route to be inserted into the route-tree.

> ⚠️ Note: For `tsr generate` and `tsr watch` to work properly, the file route instance must be exported from the file using the `Route` identifier.

### Examples

```tsx
import { FileRoute } from '@tanstack/react-router'

export const Route = new FileRoute('/').createRoute({
  loader: () => {
    return 'Hello World'
  },
  component: IndexComponent,
})

function IndexComponent() {
  const data = Route.useLoaderData()
  return <div>{data}</div>
}
```

# LinkOptions type

The `LinkOptions` type extends the [`NavigateOptions`](../NavigateOptionsType.md) type and contains additional options that can be used by TanStack Router when handling actual anchor element attributes.

```tsx
type LinkOptions = NavigateOptions & {
  target?: HTMLAnchorElement['target']
  activeOptions?: ActiveOptions
  preload?: false | 'intent'
  preloadDelay?: number
  disabled?: boolean
}
```

## LinkOptions properties

The `LinkOptions` object accepts/contains the following properties:

### `target`

- Type: `HTMLAnchorElement['target']`
- Optional
- The standard anchor tag target attribute

### `activeOptions`

- Type: `ActiveOptions`
- Optional
- The options that will be used to determine if the link is active

### `preload`

- Type: `false | 'intent' | 'viewport' | 'render'`
- Optional
- If set, the link's preloading strategy will be set to this value.
- See the [Preloading guide](../../../guide/preloading.md) for more information.

### `preloadDelay`

- Type: `number`
- Optional
- Delay intent preloading by this many milliseconds. If the intent exits before this delay, the preload will be cancelled.

### `disabled`

- Type: `boolean`
- Optional
- If true, will render the link without the href attribute

# LinkProps type

The `LinkProps` type extends the [`ActiveLinkOptions`](../ActiveLinkOptionsType.md) and `React.AnchorHTMLAttributes<HTMLAnchorElement>` types and contains additional props specific to the `Link` component.

```tsx
type LinkProps = ActiveLinkOptions &
  Omit<React.AnchorHTMLAttributes<HTMLAnchorElement>, 'children'> & {
    children?:
      | React.ReactNode
      | ((state: { isActive: boolean }) => React.ReactNode)
  }
```

## LinkProps properties

- All of the props from [`ActiveLinkOptions`](../ActiveLinkOptionsType.md)
- All of the props from `React.AnchorHTMLAttributes<HTMLAnchorElement>`

#### `children`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pulgueta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
