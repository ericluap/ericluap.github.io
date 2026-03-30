---
layout: post
title: "What's a fibration?"
date: 2026-03-29
categories: types
---

Let's say we have a type X with some elements.

![](/assets/images/fibration/fibration_type_X.png){: .center-image width="300"}

We can imagine splitting up all the elements of X into separate groups.

![Image of a circle labelled "X" with some points inside it](/assets/images/fibration/fibration_mainFib_typeX.png){: .center-image width="300"}

This is a fibration. We call each imagined grouping a fiber. So this fibration of X has three fibers.

We can have other fibrations of X:

  <div style="display:flex; gap:10px; flex-wrap: wrap">
  <img src="/assets/images/fibration/fibration_sideFib1_typeX.png" width=300 height=300>
  <img src="/assets/images/fibration/fibration_sideFib2_typeX.png" width=300 height=300>
  </div>
 
  Both of these fibrations of X have three fibers.

## Describing fibrations

Now we would like to be able to describe a fibration nicely. We begin by giving a name to each fiber. We name ours `blue`, `orange`, and `green`.

![Image of name written next to each fiber](/assets/images/fibration/fibration_namedFib_typeX.png){: .center-image width="300"}

We can now create a function from X to {`blue`, `orange`, `green`} that assigns to each element of X the name of the fiber that it is in.

![Image of function assigning name to each element](/assets/images/fibration/fibration_funcEnc_typeX.png)
  
Great! This function nicely captures our fibration of X. A different function would encode a different fibration of X.


There is another way we could describe a fibration of X. We can create a function that, given the name of a fiber, returns the fiber.

![Image of function assigning fiber to each name](/assets/images/fibration/fibration_indexedEnc_typeX.png)

This function also captures our fibration of X. A different output for any given name would encode a different fibration.

Let's look at a specific example. We have the type `Bool × Bool` of pairs of booleans:

![Image of circle with points labelled by pairs of booleans](/assets/images/fibration/fibration_boolPair.png){: .center-image width="300"}

We can imagine grouping together elements whose first booleans are equal.

![Image grouping together pairs of booleans with same first component](/assets/images/fibration/fibration_fib_boolPair.png){: .center-image width="300"}

This is a fibration on `Bool × Bool` that has two fibers. And here are the two different ways to encode that fibration.

As a function from elements of `Bool × Bool` to the names of their fibers they are in:

![Image showing function mapping each grouping of pairs of booleans to their first component](/assets/images/fibration/fibration_funcEnc_boolPair.png)

And as a function from the names of the fibers to the entire fibers themselves:

![Image showing function mapping boolean to pairs of booleans with that boolean as first component](/assets/images/fibration/fibration_indexedEnc_boolPair.png)
## Going backwards

Let's say we are given some random function `p` from a type X to some other type Y. 

![Image showing function from a circle with points labelled "X" to another such one labelled "Y"](/assets/images/fibration/fibration_randFunc.png)

We can imagine that this function `p : X → Y` is actually encoding a fibration. We do this by treating Y as the type of names of each fiber. Then we get a fibration on X by imagining grouping together elements X that are given the same name by `p`.

![Image circling points in X that map to the same point in Y](/assets/images/fibration/fibration_funcEnc_randFunc.png)

(We have some unused names in Y, but that's ok for us.)

Now let's say we are given a random function `f` from a type `Y` to a collection of types `Type`.

![Image showing function from circle with points labelled "Y" to another circle with points labelled "Types"](/assets/images/fibration/fibration_randIndex.png)

We can imagine that this function `f : Y → Type` is encoding a fibration. We do this by treating Y as the type of names of each fiber. And then what `f` maps each name to we will treat as being the entire fiber associated with that name.

![Image highlighting the points in the image of the function from Y to Types](/assets/images/fibration/fibration_indexedEnc_randIndex.png)

Now the type that actually has the fibration that this function `f : Y → Type` is encoding is built by combining all the fibers together.

![Image showing turning the separate fibers into a single type](/assets/images/fibration/fibration_combine_randIndex.png)

## Moving between

We can now take a random function, view it as encoding a fibration, switch to the other encoding of the fibration, and get back out a new function.

Let's go through an example. We begin with a function `p : Bool × Bool → Bool` from pairs of booleans to a single boolean.

![Image showing a function from circle containing pairs of booleans to circle containing booleans](/assets/images/fibration/fibration_randFunc_boolPair.png)

From this function `p : Bool × Bool → Bool`, we get this fibration on `Bool × Bool`.

![Image showing grouping of points in the circle containing pairs of booleans](/assets/images/fibration/fibration_randFunc_boolPair_fib.png){: .center-image width="300"}

Finally, we write down the other encoding of this fibration to get a function `S : Bool → Type`.

![Image showing function from booleans to Types mapping each boolean to its associated fiber](/assets/images/fibration/fibration_randFunc_boolPair_indexEnc.png)

So we have turned a function `p : Bool × Bool → Bool` into a collection of types indexed by `Bool` (i.e. our function `S : Bool → Type`).

And that's basically what fibrations are up to (ignoring all that path business).


