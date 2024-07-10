---
layout: post
title:  "Soroban Rust Backend Compiler Plugin Deep Dive: Translating And If Let Back to Rust"
date: 2024-07-10
---

In a previous post title _Rust Specific Construct Representations ep. 2: Let-Else_, I demonstrated how we can translate `if let` from Rust to DTR.

Consider the simplest example:

```rust
if let Ok(foobar) = ok_foobar {
  return foobar;
}
```

The DTR is nearly just as straightforward.

```
{ id: 0, instruction: try_assign, input: (ok_foobar, Ok(foobar)), assign: CONDITIONAL_JUMP_ASSIGNMENT_1, scope: 0 }
{ id: 1, instruction: jump, input: (CONDITIONAL_JUMP_ASSIGNMENT_1, 1), scope: 0 }
{ id: 2, instruction: return, input: (foobar), scope: 1 }
```

Yet (and at the risk of sounding like a broken record), DTR, or [Digicus Textual Representation](http://localhost:4000/digicus/#digicus-textual-representation), defines method logic via a set of sequentially executed transactions. DTR instructions have a _scope_ field to represent blocks of contiguously executed code. Only by jumping between scopes can we emulate branching and looping.

Thus, code generation is performed over the limited set of instructions defined in the Digicus spec and without an `if let` instruction, it may not be immediately clear we're handling such a statement.

To accomplish this, we'll need two tools from our toolkit:
1. the **condensation** step outlined in the previous post
2. continued (ab)use of a variadic definition of jump

In the previous post, we briefly looked at taking _assign_ instructions defining CONDITIONAL_RESULTs and substituting these into the `jump`. Lets say we accomplish this with the help of a [symbol table](https://en.wikipedia.org/wiki/Symbol_table). Our entry may look like this:

| Name                          | Scope | Value                   |
| ----------------------------- | ----- | ----------------------- |
| CONDITIONAL_JUMP_ASSIGNMENT_1 | X     | [OK(foobar), ok_foobar] |

From here, we'd substitute in `CONDITIONAL_JUMP_ASSIGNMENT_1` within the jump instruction:

```
{ id: 1, instruction: jump, input: (OK(foobar), ok_foobar, 1), scope: 0 }
```

Note that at time of writing, three inputs is not supported

<div style="text-align:center">
  <img src="../../../images/jump_instruction_description.png" alt="jump instruction definition"/>
</div>

Thus, we can define a third case.

> whenever three inputs are given, the first is the _let_ left hand side, the second the _let_ hand side, and the third the scope to jump to if the condition is true

With this new definition, and our condensed instruction set (reproduced below)
```
{ id: 1, instruction: jump, input: (OK(foobar), ok_foobar, 1), scope: 0 }
{ id: 2, instruction: return, input: (foobar), scope: 1 }
```

we can translate back to rust (instruction ids are square bracketed):

```rust
// [1]
if let Ok(foobar) = ok_foobar {
  // [2]
  return foobar;
}
```
