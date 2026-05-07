---
trigger: always_on
description: This library implements a `URLPattern` collection called a `URLPatternList` that
---


# URLPatternList Copilot Instructions

## Overview

This library implements a `URLPattern` collection called a `URLPatternList` that
has optimized matching against the set of patterns using a **prefix tree (trie)
data structure** instead of linear scanning.

`URLPattern` is a web standard that is also available in server runtimes like
Node and Deno.
- URLPattern on MDN:
  https://developer.mozilla.org/en-US/docs/Web/API/URL_Pattern_API
- Specification: https://urlpattern.spec.whatwg.org/

`URLPattern` is often used for client-side and HTTP-server routing. Especially
in the server routing use case, a server might have hundreds or thousands of
routes and need to match an incoming request URL to a route as fast as possible.
Linearly scanning a list of URLPattern and testing each pattern does not scale
well enough, and prefix-tree (trie) based search can be much faster.

## API

`URLPatternList` has two main APIs:
- `addPattern(pattern)` which adds a new pattern to the list and updates the
  internal prefix tree.
- `match(url, baseUrl)` which searches the prefix tree for a match.

Patterns cannot currently be removed or inserted into the middle of the list.
This may be an important feature to add in the future.

## Architecture

### Parsing

`src/lib/parse-pattern.ts` contains the `parse()` function which parses a
`URLPattern` into an array of `Parts`. The Part types include:
- Fixed: A fixed string, like `/api`.
- SegmentWildcard: A named parameter like `/:id`
- FullWildcard: A `*` wildcard
- Regex: A regex like `(\\d+)` or `(small|large)`

These parsed parts correspond to the types of prefix tree nodes as well. Fixed
parts are split by path separators (`/`) to make construction of the tree
simpler. **This splitting happens for all URL components, not just pathnames.**

The decision to split by `/` in search and hash components is intentional and
optimizes for common real-world patterns where path-like structures appear in
these components (e.g., `?path=/api/users` or `#/admin/dashboard`). This allows
the prefix tree to share common prefixes like `/api` even within search
parameters and hash fragments, improving performance.

Instead of building a radix-tree that sometimes needs to split nodes when new
nodes are added, we always make a path segment a separate node, and never
subdivide segments, so we only have to append to the tree. One of the goals of
this approach is to make it less costly to add patterns.

### Tree Construction

`URLPatternList.prototype.addPattern` (in `src/index.ts`) builds the prefix tree
from new patterns. The tree is made of `PrefixTreeNode` objects.

`PrefixTreeNode` is an abstract base class with five concrete implementations:
- `RooPrefixTreeNode`: The root of the tree.
- `FixedPrefixTreeNode`: Fixed string segments like `/api` or `/users`
- `WildcardPrefixTreeNode`: Named parameters like `:id` or `:userId` (with
  prefix/suffix support)
- `FullWildcardPrefixTreeNode`: Catch-all segments like `*`
- `RegexPrefixTreeNode`: Regex patterns like `(\\d+)` or `(small|large)`

`addPattern()` parses the pathname into a `Array<Part>` and then calls
`#addPatternToTree()` to recursively build the tree. During the recursion the
current part is checked against the current tree node to see if the tree node
matches that part.

Each tree node has it's own part matching logic. Fixed nodes match fixed parts
if their value and modifier are the same. Wildcard nodes match if their
modifier, prefix, and suffix are the same, and don't care about the parameter
name. Full wildcard nodes match if their modifier is the same. Regex nodes match
if the regex string is the same.

The point of the part matching logic is to share the ability to match a segment
of a URL pathname and consolidate and reduce the number of checks against the
path. This is why the parameter name doesn't matter for wildcard nodes.

If a tree node matches the current parsed part, then the process continues
recursively. If there is no match at that level of the tree, then a new tree
node is created and the process continues recursively from the new node.

Once all the parsed parts have been consumed the pattern is added to the final
tree node's `patterns` array, as part of a `URLPatternListItem` object.

`URLPatternListItem` stores the pattern, the value associated with the pattern
that was passed to `addPattern()`, and a `sequence` number that records the
order the pattern was added to the tree. `sequence` helps us implement the same
first-added-match-wins semantics of a linear search.

`PrefixTreeNode` has a `minSequence` field that tracks the minium `sequence`
number of any pattern stored in its subtree. This number is updated any time a
new pattern is added to a node's subtree. It's used to make the first-match
behavior efficient.

### URL Matching

Matching against a URL is done by calling `match()` on the root node, which
recursively calls `match()` on the subtrees.

`match()` is called with an array of `URLComponent`s. The given URL transformed
into this array so we can match it against the prefix tree from left-most
component (protocol) to right-most (hash) by incrementing an array index.

`match()` is implemented by each tree node implementation. It first checks for a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [justinfagnani/url-pattern-list](https://github.com/justinfagnani/url-pattern-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
