---
layout: post
title: "How Lean tracks your definitions"
date: 2025-12-08
categories: lean
---

The big idea is that Lean has a dictionary that maps names of previously defined constants to their definitions. When it encounters a constant, it looks up what the constant means inside of this dictionary. Very reasonable.

Here's an example of it working:
```lean
import Lean
open Lean

def test := "hi"

run_meta
  let env : Environment ← getEnv
  
  let info := env.find? ``test |>.get!
  Lean.logInfo m!"{info.type}"
```
We define a new constant `test` which Lean stores in its dictionary. Then we get the environment (the environment has our dictionary), look up the name `test` inside of the environment dictionary, and print some of the information we get back. Great.

Now here's a strange situation where things don't work:
```lean
import Lean
open Lean

def test := "hi"

run_meta
  let env : Environment ← getEnv
  let kernelEnv : Kernel.Environment := env.toKernelEnv
  let env : Environment := Environment.ofKernelEnv kernelEnv

  let info := env.find? ``test |>.get!
  Lean.logInfo m!"{info.type}"
```
Here it fails to find the information about `test`. That is not ideal. But ok we've added some weird stuff at the start. What's going on?

## The two environments
We begin by getting the environment `env` which has type `Environment`. We then convert it into `kernelEnv` which has type `Kernel.Environment`. Lastly, we convert it into `env` which again has type `Environment`.

The type `Kernel.Environment` is the final actual environment that Lean produces. However, Lean doesn't actually process each of our definitions one after another: it tries to process them asynchronously. And that's where `Environment` comes into play. The `Environment` tracks all these asynchronous things happening and builds up `Kernel.Environment` as things are completed.

When we convert `env` into `kernelEnv`, it tells Lean to wait for all the asynchronous things to finish so that we can have a `Kernel.Environment` representing all the constants that have been defined so far. Then when we convert `kernelEnv` back into `env`, it just sets all the extra asynchronous tracking stuff to be empty since nothing asynchronous is going on.

So why would this conversion back and forth mean we can't find our defined constant `test` anymore? One might wonder if perhaps our code made Lean forget about all the constants. But the following code does work:
```lean
import Lean
open Lean

run_meta
  let env ← getEnv
  let kernelEnv := env.toKernelEnv
  let env := Environment.ofKernelEnv kernelEnv

  let info := env.find? ``String |>.get!
  Lean.logInfo m!"{info.type}"
```
Here we just tried to find the information about the definition of `String` and it succeeds. So our messing with the environment is forgetting `test` but not `String`?
## The staged map
It turns out that Lean doesn't actually store all the constants we've defined in a single dictionary. It splits things into two dictionaries! The first dictionary stores all the constants defined in stuff that we're importing while the second dictionary stores all the constants we're defining locally. And so `String` is in the first dictionary and `test` is in the second. Somehow our environment conversions are losing everything in the second dictionary of local constants.

But wait why does Lean split things up into two dictionaries? The answer is speed and the key is that the first dictionary is implemented as a hashmap while the second as a hash trie.

The first hashmap has type `Std.HashMap`. Let's look at how that works. This is the hashmap implementation that one is typically going to use in Lean. Since things are immutable in a functional language like Lean, inserting into an `Std.HashMap` should be pretty slow as it has to create an entirely new array every time we insert. But Lean is clever. So long as we only have a single reference to an array, it will do the update mutably and thus quickly. This works great for reading in the constants from imported modules. They've already been processed so Lean just reads them all in quickly into the `Std.HashMap` doing mutable updates.

In the current file, each definition has to be fully processed before adding itself to the list of constants and that's why Lean does all this asynchronous stuff. However, the asynchronous things happening means that there is more than one reference to the dictionary storing these constants and so using `Std.HashMap` would be slow. That's where the `PersistentHashMap` comes into play. The way it is implemented is called a hash trie and it looks like a shallow and wide tree. Insertions just have to update a single path in the tree and all the rest of the tree can be reused. Thus, when there are many references to the dictionary and things need to be copied, the `PersistentHashMap` is faster.

Ok so we understand that Lean stores imported constants in one dictionary and locally defined constants in another in order to be fast. But the question still remains: why does going from `Environment` to `Kernel.Environment` back to `Environment` lose all the constants in the dictionary used for locally defined constants?

## Finding a constant in the maps
When you have a `Kernel.Environment` and try to look for a constant in it, Lean checks for the constant in both of the dictionaries it has. However, if you have an `Environment` there might be constants that have been defined but are still being processed asynchronously. In order to handle this, Lean has yet another dictionary! This dictionary is called `asyncConstsMap`. Each definition first adds itself immediately to the `asyncConstsMap` with its associated value being a promise (accessing a promise blocks until its asynchronous processing is completed). And then once the processing is complete, the constant is added to the dictionary in the `Kernel.Environment` that stores locally defined constants.

So when we try to find a constant in the `Environment`, it first checks the `Kernel.Environment` that it has built up so far for the constant and then checks to see if the constant is inside `asyncConstsMap`. But every constant that ends up in the second dictionary in `Kernel.Environment` was first added to `asyncConstsMap`. So in order to avoid looking through these constants twice, Lean does not check the second dictionary of the `Kernel.Environment`.

And so there is root of our issue. Lean assumes that every locally defined constant that appears in the second dictionary in its current `Kernel.Environment` also appears in `asyncConstsMap`. When we turn our `Environment` into a `Kernel.Environment` and then back into an `Environment`, we lose everything inside of `asyncConstsMap` and break that invariant.

To drive the point home, let's modify our failing example:
```lean
import Lean
open Lean

def test := "hi"

run_meta
  let env : Environment ← getEnv
  let kernelEnv : Kernel.Environment := env.toKernelEnv
  let env : Environment := Environment.ofKernelEnv kernelEnv

  -- Succeeds
  let info := kernelEnv.find? ``test |>.get!
  Lean.logInfo m!"{info.type}"

  -- Fails
  let info := env.find? ``test |>.get!
  Lean.logInfo m!"{info.type}"
```
If we look for `test` inside of `kernelEnv` it succeeds because it checks both dictionaries. So indeed the final `env` knows about `test` but fails to find it due to the broken invariant.
