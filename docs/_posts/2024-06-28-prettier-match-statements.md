---
layout: post
title:  "Prettier Match Statements"
date: 2024-06-28
---

## A Recap on Scopes

**Definition:** a scope represents a continuous, uninterrupted block of logic. While one implementation may decide to increment (and decrement) scope sequentially, scopes hold no meaning except that scope 1 == scope 1; thus, it may be more useful to think of a scope like a labelled block. 

**How do you _jump_ between scopes?**

Punny right? To change scope, we use the `jump` instruction. This instruction is variadic:

* jump(SCOPE)           | unconditional scope modification
* jump(BOOLEAN, SCOPE)  | conditional scope modification
 
A scope is not incremented or decremented, but assigned. We've intentionally decided to require explicit scope declaration. 

Consider the following annotated code:

```rust
// [0]
if x == 10 {
  // [1]
  i += 1;

  // [1]
  if i < 7 {
    // [2a]
    x -= 1;
  } else {
    // [2b]
    x *= 2;
  }

  // [1]
  i = 7;
}
// [0]
```

This results in the following DTR.

```
{ instruction: "evaluate", input: (equal_to, x, 10), assign: CONDITIONAL_0, scope: 0 }
{ instruction: "jump", input: (CONDITIONAL_0, 1), scope: 0 }
{ instruction: "evaluate", input: }

```

##
In Ruby and Rust, we get some pretty nice pattern matching.

```ruby
case x
when 10
  "ten"
when 20
  "twenty"
else
  "Not ten and not twenty"
end
```

In simple cases, where we don't leverage the pattern matching, this is basically syntactic sugar for:

```ruby
if x == 10
  "ten"
elsif x == 20
  "twenty"
else
  "Not ten and not twenty"
end
```

This is equivalent to:

```ruby
if x == 10
  "ten"
else
  if x == 20
    "twenty"
  else
    "Not ten and not twenty"
  end
end
```

Which translates quite well into DTR.

```
// [0] CONDITONAL_0 = x == 10
{ instruction: "evaluate", input: ("equal_to", x, 10), assign: CONDITONAL_0, scope: 0 }
// [0] if CONDITONAL_0, scope = 1
{ instruction: "jump", input: (CONDITONAL_0, x, 1), scope: 0 }
// [1] print "ten"
{ instruction: "print", input: ("ten"), scope: 1 }

// [0] CONDITONAL_1 = x == 20
{ instruction: "evaluate", input: ("equal_to", x, 20), assign: CONDITONAL_1, scope: 0 }
// [0] if CONDITONAL_0, scope = 1
{ instruction: "jump", input: (CONDITONAL_0, x, 1), scope: 0 }
// [1] print "ten"
{ instruction: "print", input: ("ten"), scope: 1 }


{ instruction: "evaluate", input: ("equal_to", x, 10), assign: CONDITONAL_0, scope: 0 }
{ instruction: "jump", input: (CONDITONAL_0, x, 1), scope: 0 }
{ instruction: "print", input: ("ten"), scope: 1 }

```