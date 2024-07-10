---
layout: post
title:  "Soroban Rust Backend Compiler Plugin Deep Dive: Translating Branching Logic"
date: 2024-07-09
---

DTR, or [Digicus Textual Representation](http://localhost:4000/digicus/#digicus-textual-representation), defines method logic via a set of sequentially executed transactions. DTR instructions have a _scope_ field to represent blocks of contiguously executed code. By jumping between scopes, we can emulate branching.

Consider the following DTR:
```
{ id: 0, instruction: evaluate, input: (equal_to, 1, x), assign: CONDITONAL_RESULT_1, scope: 0}
{ id: 1, instruction: jump, input: (CONDITIONAL_RESULT_1, 1), scope: 0}
{ id: 2, instruction: jump, input: (2), scope: 0}
{ id: 3, instruction: print, input: ("If only"), scope: 1}
{ id: 4, instruction: jump, input: (0), scope: 1}
{ id: 5, instruction: evaluate, input: (equal_to, 1, y), assign: CONDITONAL_RESULT_2, scope: 2}
{ id: 6, instruction: jump, input: (CONDITIONAL_RESULT_2, 3), scope: 2}
{ id: 7, instruction: jump, input: (4), scope: 2}
{ id: 8, instruction: print, input: ("ElseIf"), scope: 3}
{ id: 9, instruction: jump, input: (0), scope: 3}
{ id: 10, instruction: print, input: ("Else Only"), scope: 4}
{ id: 11, instruction: jump, input: (0), scope: 4}
{ id: 12, instruction: print, input: ("The End"), scope: 0}
```

Graphically, the flow works like this:
<div style="text-align:center">
  <img src="../../../images/graphical-if-elseif-else-dtr_to_rust.png" alt="loop visual"/>
</div>

From here, we can condense down the conditional evaluations to get to a slightly easier to consume graphic. Notice that we retain the ids.
<div style="text-align:center">
  <img src="../../../images/graphical-if-elseif-else-dtr_to_rust_2.png" alt="loop visual"/>
</div>

Combining jumps back to 0, we get:
<div style="text-align:center">
  <img src="../../../images/graphical-if-elseif-else-dtr_to_rust_3.png" alt="loop visual"/>
</div>

Which can be reorganized with pseudocode annotations:
<div style="text-align:center">
  <img src="../../../images/graphical-if-elseif-else-dtr_to_rust_4.png" alt="loop visual"/>
</div>

So the question becomes: *how do we perform this (1) condensation, (2) compaction, and then (3) Connection* programmatically?

To do so, we'll construct a directed, acyclic graph. Each vertex in this graph has at most two edges:
1. **detour:** the highest precedence edge to flow through
2. **continue:** the lowest precedence edge

We'll name this structure a left child preferential binary tree (where the left child is the _detour_).

This graph will be constructed in three steps:
1. **Condensation:** in this step, we seek to condense the conditional evaluation logic such that we can replace the programmatically generated input variable within the conditional jump instruction. For more elaborate/complex evaluations, this may be a bit of challenge.
2. **Compaction:** combine same scope, sequential instructions into _segments_. Each of these _segments_ represents a linear execution flow and will be a node in the flow graph.
3. **Connection:** if we operate under the assumption that related conditionals are contiguous (which holds true for the match statement implementation in the `soroban_rust_frontend` compiler frontend), we can iterate through the instructions, linking nodes based on their entry points.

Continuing with our example, we'd get a graph defined as so (where each number is an instruction identifier).

```
Self: 1
Detour: 3
Continue: 2

Self: 2
Detour: 6
Continue: X

Self: 3
Detour: X
Continue: X 

Self: 6
Detour: 8
Continue: 7

Self: 7
Detour: 10
Continue: X

Self: 8
Detour: X
Continue: X

Self: 10
Detour: X
Continue: X

Self: 12
Detour: X
Continue: X 
```

With this, we can generate rust code by traversing our graph based on the precedence defined. Here, (1) is our root, so we will start there. We will explain each step with our current generated code following along below each explanation.

First, start with 1. We generate self and then detour to 3.
```
// 1
if 1 == x {
```

At 3 we generate self (indented since we are an inner scope) then we halt since we're jumping back to 0.

```
if 1 == x {
  // 3
  log!("If Only");
// 4
}
```

Since our detour finished, at 1 we then continue on to 2. At 2 we generate self, then detour to 6.
```
if 1 == x {
  // 3
  log!("If Only");
// 4
} 
// 2
else {
```

At 6 we generate self, then detour to 8.
```
if 1 == x {
  // 3
  log!("If Only");
// 4
} 
// 2
else {
  // 6
  if 1 == y {
```

At 8 we generate self, then we halt since we're jumping back to 0.
```
if 1 == x {
  // 3
  log!("If Only");
// 4
} 
// 2
else {
  // 6
  if 1 == y {
    // 8
    log!("ElseIf");
  // 9
  }
```

Returning from 8, back at 6, we continue to 7. We generate self, then detour to 10.
```
if 1 == x {
  // 3
  log!("If Only");
// 4
} 
// 2
else {
  // 6
  if 1 == y {
    // 8
    log!("ElseIf");
  // 9
  }
  // 7
  else {
```

At 10, we generate self, and then halt since we're jumping to 0.
```
if 1 == x {
  // 3
  log!("If Only");
// 4
} 
// 2
else {
  // 6
  if 1 == y {
    // 8
    log!("ElseIf");
  // 9
  }
  // 7
  else {
    // 10
    log!("Else Only");
  // 11
  }
// TECHNICALLY IN REAL DTR WE'D have another jump here... typo in the example.
}
```

All the way back at 1, we sequentially iterate (due to the nature of execution of DTR instruction sets) at scope 0 until we make it to instruction 12. Finally at 12, we mark 12 as visited and generate 12. As this instruction halts, we sequentially iterate, realizing we're at the end of execution.
```
if 1 == x {
  // 3
  log!("If Only");
// 4
} 
// 2
else {
  // 6
  if 1 == y {
    // 8
    log!("ElseIf");
  // 9
  }
  // 7
  else {
    // 10
    log!("Else Only");
  // 11
  }
}
log!("The End");
```

And it's a success! We've designed an algorithm to construct a left child preferential binary tree and traversed it to generate the correct Rust code from DTR. We can leverage this strategy to similarly solve the following branching constructs:

* if-else
* nested if-else
* match statements

Stay tuned for the next post on looping.

