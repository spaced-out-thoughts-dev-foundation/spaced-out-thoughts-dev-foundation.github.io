---
layout: page
title: Digicus
permalink: /digicus/
---

## Technical Architecture

## Overview

### The Problem
**Consider the following:**
1. [While Rust is one of the most admired languages, it is far from one of the most popular](https://survey.stackoverflow.co/2023/#section-admired-and-desired-programming-scripting-and-markup-languages).
2. [Scratch is an incredible first language, in fact it’s also more popular than Rust](https://www.tiobe.com/tiobe-index/).
3. [More mature and seasoned smart contract platforms are complicated and error prone](https://www.mdpi.com/2624-800X/2/2/19).
4. [65% of the general population are visual learners (https://gitnux.org/visual-learner-statistics](#:~:text=Highlights%3A%20The%20Most%20Important%20Visual,children%20identify%20as%20visual%20learners.).

Furthermore, consider the following quotes from the Rust SDK Docs:
* “Before submitting to the network, developers should inspect the resulting Wasm binary emitted by the Rust compiler to ensure that it contains only the intended code and data, and is as small as possible.”
* “​​Developers must understand the difference between code that is compiled-in to Wasm modules for deployment and code that is conditionally compiled for testing. See debugging contracts for more details.”

While the Soroban developer experience is an improvement over most smart contract platforms in existence today, technical bar is still very high.

### The Solution

**Hypothesis:** the existence of a visual, block-based programming language which can translate to and from Soroban smart contracts will promote comprehension, adaption, and best practices helping the young, Stellar smart contract community reach maturity at an expedited rate.

Generally described as “scratch for smart contracts”, Digicus (hybrid Soroban) is a compiler which functions as an abstraction layer on top of existing Soroban tooling. Digicus allows users to create smart contracts by piecing together blocks (much like Legos) as well as visualize existing smart contracts.

***

## Hello World Example

### Rust

```rust
#![no_std]
use soroban_sdk::{contract, contractimpl, symbol_short, vec, Env, Symbol, Vec};

#[contract]
pub struct HelloContract;

#[contractimpl]
impl HelloContract {
    pub fn hello(env: Env, to: Symbol) -> Vec<Symbol> {
        vec![&env, symbol_short!("Hello"), to]
    }
}
```

### Digicus Textual Representation (DTR)

```
[Contract]: HelloContract

[Functions]:
  * [hello]
      * Input:
        { 
          to: Symbol 
        }
      * Output: Symbol
      * Instructions:
        {
          { instruction: AddSymbols, input: ("Hello", to), assign: HelloToResult }
          { instruction: Return, input: HelloToResult }
        }
```

### Digicus Block Based Visual Represention

![mockup](../images/digicus-ide-mockup.png)
