---
trigger: always_on
description: We like to keep the source consistent and readable.  Herein are some
---

# Libgit2 Conventions

We like to keep the source consistent and readable.  Herein are some
guidelines that should help with that.

## External API

We have a few rules to avoid surprising ways of calling functions and
some rules for consumers of the library to avoid stepping on each
other's toes.

 - Property accessors return the value directly (e.g. an `int` or
   `const char *`) but if a function can fail, we return a `int` value
   and the output parameters go first in the parameter list, followed
   by the object that a function is operating on, and then any other
   arguments the function may need.

 - If a function returns an object as a return value, that function is
   a getter and the object's lifetime is tied to the parent
   object. Objects which are returned as the first argument as a
   pointer-to-pointer are owned by the caller and it is responsible
   for freeing it. Strings are returned via `git_buf` in order to
   allow for re-use and safe freeing.

 - Most of what libgit2 does relates to I/O so as a general rule
   you should assume that any function can fail due to errors as even
   getting data from the filesystem can result in all sorts of errors
   and complex failure cases.

 - Paths inside the Git system are separated by a slash (0x2F). If a
   function accepts a path on disk, then backslashes (0x5C) are also
   accepted on Windows.

 - Do not mix allocators. If something has been allocated by libgit2,
   you do not know which is the right free function in the general
   case. Use the free functions provided for each object type.

## Compatibility

`libgit2` runs on many different platforms with many different compilers.

The public API of `libgit2` is [ANSI C](http://en.wikipedia.org/wiki/ANSI_C)
(a.k.a. C89) compatible.

Internally, `libgit2` is written using a portable subset of C99 - in order
to maximize compatibility (e.g. with MSVC) we avoid certain C99
extensions.  Specifically, we keep local variable declarations at the tops
of blocks only and we avoid `//` style comments.

Also, to the greatest extent possible, we try to avoid lots of `#ifdef`s
inside the core code base.  This is somewhat unavoidable, but since it can
really hamper maintainability, we keep it to a minimum.

## Match Surrounding Code

If there is one rule to take away from this document, it is *new code should
match the surrounding code in a way that makes it impossible to distinguish
the new from the old.* Consistency is more important to us than anyone's
personal opinion about where braces should be placed or spaces vs. tabs.

If a section of code is being completely rewritten, it is okay to bring it
in line with the standards that are laid out here, but we will not accept
submissions that contain a large number of changes that are merely
reformatting.

## Naming Things

All external types and functions start with `git_` and all `#define` macros
start with `GIT_`.  The `libgit2` API is mostly broken into related
functional modules each with a corresponding header.  All functions in a
module should be named like `git_modulename_functioname()`
(e.g. `git_repository_open()`).

Functions with a single output parameter should name that parameter `out`.
Multiple outputs should be named `foo_out`, `bar_out`, etc.

Parameters of type `git_oid` should be named `id`, or `foo_id`.  Calls that
return an OID should be named `git_foo_id`.

Where a callback function is used, the function should also include a
user-supplied extra input that is a `void *` named "payload" that will be
passed through to the callback at each invocation.

## Typedefs

Wherever possible, use `typedef`.  In some cases, if a structure is just a
collection of function pointers, the pointer types don't need to be
separately typedef'd, but loose function pointer types should be.

## Exports

All exported functions must be declared as:

```c
GIT_EXTERN(result_type) git_modulename_functionname(arg_list);
```

## Internals

Functions whose *modulename* is followed by two underscores,
for example `git_odb__read_packed`, are semi-private functions.
They are primarily intended for use within the library itself,
and may disappear or change their signature in a future release.

## Parameters

Out parameters come first.

Whenever possible, pass argument pointers as `const`.  Some structures (such
as `git_repository` and `git_index`) have mutable internal structure that
prevents this.

Callbacks should always take a `void *` payload as their last parameter.
Callback pointers are grouped with their payloads, and typically come last
when passed as arguments:

```c
int git_foo(git_repository *repo, git_foo_cb callback, void *payload);
```

## Memory Ownership

Some APIs allocate memory which the caller is responsible for freeing; others
return a pointer into a buffer that's owned by some other object.  Make this
explicit in the documentation.

## Return codes

Most public APIs should return an `int` error code.  As is typical with most
C library functions, a zero value indicates success and a negative value
indicates failure.

Some bindings will transform these returned error codes into exception
types, so returning a semantically appropriate error code is important.
Check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salesforce/p4-fusion](https://github.com/salesforce/p4-fusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
