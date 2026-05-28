---
trigger: always_on
description: This repo uses the Certora Prover (CVL) to formally verify Solidity contracts.
---

## Review guidelines

### CVL Context

This repo uses the Certora Prover (CVL) to formally verify Solidity contracts.
The primer below describes how CVL actually behaves — read it before reasoning about `.spec` files, because CVL semantics differ in subtle ways from Solidity and from what an LLM might assume by default.

#### Rules, invariants, `satisfy`

- A `rule` passes iff every `assert` holds on every execution path that satisfies all preceding `require`s. A counterexample is a concrete trace (env, args, method choice) that satisfies the `require`s and falsifies an `assert`.
- A rule can be *vacuous*: if the `require`s exclude every path, it passes trivially. Vacuity is silent unless a sanity rule is run. Basic sanity is run by default.
- `satisfy p` is the dual of `assert`: the Prover must find at least one feasible path where `p` holds. A rule with only `satisfy` and no `assert` therefore does not check any universal property.
- CVL has both *weak* and *strong* invariants. A weak invariant is proven by induction over methods: base case checks it after the constructor; inductive step assumes it, runs an arbitrary method `f`, then asserts it again. By default invariants are checked only for `public`/`external` non-`view`/non-`pure` methods. A strong invariant assumes the invariant before any unresolved external calls, havoc the state and assert the invariant after the call. There is no temporal reasoning beyond this.
- A `preserved` block is a code block, that notably allows to inject extra `require`s into the inductive step for one method. Those assumptions are *not* checked by default, but they could be checked in other rules/invariants. Unsound `preserved` is a common source of fake invariant proofs.
- `requireInvariant J` assumes another invariant `J` at the start of the rule/invariant. It's sound only because `J` was itself proven by the same induction scheme.
- A *parametric rule* (one with a `method f` parameter) is expanded into one sub-rule per method in scope. A `filtered { f -> ... }` clause drops methods from that expansion; filtered-out methods are simply not checked.

#### Reverts and path pruning

- By default, a call `f(e, args)` **only explores non-reverting paths**. Reverting executions are silently pruned from the rule. This is a semantic choice, not an optimization.
- `f@withrevert(e, args)` explores both reverting and non-reverting paths. After it, the builtin `lastReverted` is true on the revert branches and false otherwise.
- Without `@withrevert`, `lastReverted` after a call is always `false` (since the revert paths were pruned). Testing `lastReverted` after a plain call is meaningless.
- Non-persistent ghosts are rolled back on revert; persistent ghosts are not.

#### Methods, env, calldataarg

- `env e` captures block number, timestamp, `msg.sender`, `msg.value`, etc. The Prover quantifies over all `env`s consistent with active `require`s.
- `calldataarg args` is an opaque bundle of arguments. You cannot inspect or constrain its fields; it only exists to call a `method f` parametrically.
- `f(e, args)` targets `currentContract` by default. Call other contracts via `other.f(e, args)` where `other` is declared with `using OtherContract as other;`.
- Addresses drawn nondeterministically may coincide with known contracts unless explicitly constrained (`require addr != currentContract`, etc.). The Prover does *not* assume addresses are distinct.
- `envfree` tells the Prover a method doesn't read `env`; such calls omit the `env` argument. The Prover statically checks this.

#### Ghosts, hooks, havoc

- A ghost is an SMT variable (possibly a function `uint → uint`, etc.), not contract state. It exists only in the spec and can be updated by hooks or CVL assignments.
- On an *unresolved* external call, the Prover havocs all non-persistent ghosts (assumes they take any value consistent with their axioms). `persistent ghost` declarations survive havoc.
- Hooks fire on EVM-level events: `Sload`, `Sstore`, `CALL`, `REVERT`, etc. They match by storage slot / selector / opcode. Signature or layout drift silently disables a hook — it does not error. Hooks are not triggered by CVL code, including CVL access to Solidity storage, and hooks are not recursive.
- Inside an `Sstore` hook, the bound names conventionally written as old/new values refer to the pre-write and post-write values at that slot.
- A two-state ghost function can be referenced as `g@old` / `g@new` inside `havoc g assuming ...`, letting you specify how the ghost changes across a havoc (e.g. `havoc g assuming g@new(x) == g@old(x) + 1`).
- `axiom P` constrains the ghost in every state the Prover considers — adding an unsatisfiable axiom makes every rule vacuously pass. `init_state axiom P` only constrains the ghost in the base case of invariant induction, which is almost always what you want for "starts at zero"-style facts.
- Ghost axioms may refer only to the ghost itself and quantified variables — not to Solidity or CVL functions.

#### Summaries and dispatch

- The `methods { ... }` block declares how external calls are resolved. Exact-signature entries beat wildcard entries (`function _.foo() ...`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morpho-org/midnight](https://github.com/morpho-org/midnight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
