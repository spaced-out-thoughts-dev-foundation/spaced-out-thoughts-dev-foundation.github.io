---
layout: page
title: Digicus
permalink: /digicus/
---

## Overview

Digicus, Scratch for smart contracts, simplifies contract creation & modification via an intuitive, Lego-like graphical interface.

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

## Overall Architecture


![overall architecture](../images/overall_architecture.png)


### Tools

![tools](../images/tools.png)

* [Digit](https://github.com/spaced-out-thoughts-dev-foundation/digicus/tree/main/digit) Rust to DTR two way compiler
* [Digicus IDE](https://github.com/spaced-out-thoughts-dev-foundation/digicus/tree/main/digicus_ide) block-based graphical programming environment for Soroban smart contracts

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

***

## Roadmap

* [ ] **Phase 1 ([SCF #26](https://dashboard.communityfund.stellar.org/scfawards/scf-26/earlybirdsubmission/suggestion/887)) - Minimal Viable Digicus Language and IDE (Digicus 0.1)**
  <br>
    [*Description*]: develop the core Digicus intermediate representation translation and visualization layer. From this, prove to the community such a concept is possible and gather feedback and suggestions. Begin administering developer experience surveys during this phase.
  </br>
* [ ] **Phase 2 - Digicus First Contract Development (Digicus 1.0)**
  <br>
    [*Description*]: with suggestions and a minimum viable compiler and React app in hand, continue developing the IDE so folks can create contracts from scratch with a nice and easy drag and drop interface (like what folks familiar with Scratch would expect). 
  </br>
* [ ] **Phase 3 - Enhanced, Feedback Driven Contract Development (Digicus 2.0)**
  <br>
    [*Description*]: our community is hard at work on security and vulnerability detection frameworks, standardizing smart contract best practices, etc. We plan to incorporate this work into the Digicus IDE (along with Copilot-esque auto-complete suggestions) to give Digicus developers a seriously awesome developer experience.
  </br>
* [ ] **Phase 4 - Multi-Targeted Digicus Compiler (Digicus 3.0)**
  <br>
    [*Description*]: develop an LLVM-esque compiler for Digicus that targets numerous backends. This should be straightforward for targeting other SDKs (i.e. AssemblyScript), however directly compiling to WASM and decompiling WASM and translating to Digicus (novel work) would unlock some very interesting use cases.
  </br>

***

## KPIs

We will work with SDF to administer developer experience surveys. Surveys will be administered before and after each phase of development and then quarterly after the 1.0 release.

1. 20% of existing (*hard-core*) Soroban smart contract developers will try Digicus
2. 5% of existing (*hard-core*) Soroban smart contract developers will use Digicus weekly
3. 66% of net new Soroban smart contract developers will try Digicus
4. 33% of new new Soroban smart contract developers will use Digicus weekly
