---
layout: post
title: Monads Part 2 (Applicatives)
date: 2021-03-03
---

# Why Applicatives?
We have covered functors in a previous post, and have found them to be very useful in functional programming.
However, there are cases where we wouldn't be able to apply functors in ways that we intuintively expect.  
For some functor `f` and objects `f (a -> b)` and `f a`, the functor laws don't offer a method in which to act on `f a` with our function.
Or what if we had a function with multiple arguments `a -> b -> c` and we want to apply it to arguments of type `f a` and `f b`?  
For instance, if we apply `fmap` to arguments of type `a -> b -> c` and `f a`, we would obtain an output of `f (b -> c)`.
From here, we are stuck as we have mentioned.  

Applicative functors are beefier functors that provide the solution to our dilemma.

## Programming Definition
As usual, examples will be in Haskell since it's the language of choice amongst people who care too much about this topic.  

Below is the definition of an applicative functor, which is a normal functor along with functions
`pure` and `<*>`.
```haskell
class (Functor f) => Applicative f where
    pure  :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
```

We can define `fmap` as `fmap f x = pure f <*> x`, so applicatives are clearly more expressive than regular functors.

The `<*>` operator satisfies the problem we were facing in the introduction 
where we were unable to use functions of type `f (a -> b)` with argument `f a`.  
Furthermore, if we have function with multiple arguments as `a -> b -> c`, we can apply it on functiorial types `f a` and `f b` via `pure f <*> x <*> y`.
The second form even has an even more compact form `f <$> x <*> y` where `f <$> x = pure f <*> x`.  
i.e. For functions with numerous arguments, we can apply them on functorial values like `f <$> a <*> b <*> c <*> ...`  
This is the advantage of the applicative over traditional functors: **We can lift regular functions to act on multiple functorial values as argument.**

However, this is not the only definition of an applicative functor.  
Instead of `<*>`, we can define the applicative functor as:  
```haskell
class (Functor f) => Applicative f where
    pure   :: a -> f a
    liftA2 :: (a -> b -> c) -> f a -> f b -> f c  
```

Defining just one of `<*>` or `liftA2` is sufficient to cement the 
entire behavior of the applicative since we can define one as the other.

### `<*>` and `liftA2`
Either of these functions being defined will explicitly define the behavior of an applicative, meaning that defining one should define the other.

Take a look at the type signatures again:
```haskell
<*> :: f (a -> b) -> f a -> f b
liftA2 :: (a -> b -> c) -> f a -> f b -> f c
```
Using `<*>` to define `liftA2` is easier to derive than the reverse direction since this is exactly the way we expect to use applicatives:
`liftA2 f x y = pure f <*> x <*> y = f <$> x <*> y`

The reverse direction is a bit abstruse: `<*> = liftA2 id` where `id` is the identity function of type `id :: a -> a`.  
Looking back at the type signature, `liftA2` takes as first argument `(a -> b -> c)` which can be curried to the form `(a -> (b -> c))`, meaning that when `id` is taken as the first argument, it really has as its type signature: `(b -> c) -> (b -> c)` and the type signature of `liftA2` really is `((b -> c) -> (b -> c)) -> f (b -> c) -> f b -> f c`.
i.e. `liftA2 id` as a function has type signature `f (b -> c) -> f b -> f c`, which is exactly what we want.

Consider the following table of hypothetical functions

| function | function type                     | arguments           | output  |
| -------------- | --------------------------------- | ------------------- | -----  |
| \\(fmap_0\\)   | `a -> F a`                          | `a`                   | `F a`      |
| \\(fmap_1\\)   | `(a -> b) -> F a -> F b`            | `a->b, F a`          | `F b`  |
| \\(fmap_2\\)   | `(a -> b -> c) -> F a -> F b -> F c`| `a->b->c`, `F a`, `F b` | `F c`  |

\\(fmap_0\\) is wrapping a value for regular functors and is the `pure` function for applicative functors.  

\\(fmap_1\\) is the `fmap` function that we all know from the definition of functors.  

\\(fmap_2\\) is actually something that does not exist for regular functors.  

Applicatives are functors that do have this capability since \\(fmap_2\\) is the function `liftA2`.  
Furthermore, the higher functions such as \\(fmap_3\\) (AKA `liftA3`) and onward 
are also available for applicatives.
i.e. `liftA3 f x y z = liftA2 (liftA2 f x y) z`  

The ability to lift functions to take multiple functorial values as argument is possible whether we define the applicative using `<*>` or `liftA2`.


### Applicative Functor Laws
Here are the functor laws that must be obeyed by applicatives.  
I won't go as far as to justify or explain them, but if you were to analyze them these laws should feel intuitive.  
```haskell
pure id <*> v = v                            -- Identity
pure f <*> pure x = pure (f x)               -- Homomorphism
u <*> pure y = pure ($ y) <*> u              -- Interchange
pure (.) <*> u <*> v <*> w = u <*> (v <*> w) -- Composition
```

## Mathematical Definition
In the language of category theory, an applicative is a **lax monoidal functor**.  Applicative functors were actually conceived in programming-land, so unlike functors and monads, its name is not borrowed from category theory.  
Let's build up our intuitive understanding to what this means.

A **monoid** is generally defined as a set with an associative binary operation with an identity element.  
If you're comfortable with groups, a monoid is just a group lacking invertability.  
An example of a monoid is the non-negative integers under addition.
We can add numbers together, but if we add the number `b` to `a`, there may not be a way to obtain `a` from `a+b` using just addition.
`0` will be our identity element since adding it to a sum does not change it.  
Another familiar example of a monoid is string concatenation with the empty string being the identity element.

A **monoidal category** is a category that has monoidal properties.  
Suppose \\(\mathcal{C}\\) is some category.
Along with basic requirements of being a category, \\(\mathcal{C}\\) has a binary operation that acts on pairs of objects in \\(\mathcal{C}\\) and some unit object \\(1\_\mathcal{C}\\) that will act as our "identity".  
Unlike our earlier examples of monoids, categories do not have a notion of equality.  
Instead the notion of "equality" is captured through isomorphisms.
For instance, the binary operations of a monoid is associative, so we expect \\(a\otimes (b \otimes c)\simeq (a\otimes b)\otimes c\\), which is exactly one of the rules of a monoidal category.

A **lax monoidal functor** from \\(\mathcal{C}\\) to \\(\mathcal{D}\\) is a functor between two monoidal categories that has two additional components:
1. A morphism from \\(1\_\mathcal{D}\\) to \\(F(1\_\mathcal{C})\\)
2. A natural transformation: 
\\(\mu_{x,y}: F(x)\otimes_\mathcal{D}F(y)\to F(x\otimes_{\mathcal{C}}y)\\)

*Aside: a strong monoidal functor is just a lax monoidal functor, but having isomorphisms for 1 and 2.*

Suppose we are operating in the category \\(\mathcal{H}\\) of Haskell types and have some lax monodoidal functor \\(F\\) from \\(\mathcal{H}\\) to itself.  
Consider the diagram below

$$
\require{AMScd}
\begin{CD}
(a, b)\in \mathcal{H}\times\mathcal{H} @>>> a\otimes b\in\mathcal{H}\\  
@V F VV                               @V F VV \\  
(F(a), F(b))\in\mathcal{H}\times\mathcal{H} @>>> F(a)\otimes F(b), F(a\otimes b)\in\mathcal{H}\\  
\end{CD}
$$

We start with some elements \\((a,b)\in \mathcal{H}\times\mathcal{H}\\).  
From these two elements we can either apply the funtor \\(F\\) before combining them with \\(\otimes\\), or we can do the reverse by constructing \\(a\otimes b\\) before applying \\(F\\).
We end up obtaining both \\(F(a\otimes b)\\) and \\(F(a)\otimes F(b)\\).
A lax monoidal functor establishes a relationship between these two quantities (via the natural transformation \\(\mu\\)).  
When dealing in \\(\mathcal{H}\\), \\(a\otimes b\\) is actually just the product \\((a,b)\\), so an applicative establishes a relationship between \\((F(a), F(b))\\) and \\(F(a,b)\\).

### Equivalency in Definition
Discussed earlier, an applicative in programming can be completely defined by the function `liftA2`, which converts a function of type `a -> b -> c` to a function of type `F a -> F b -> F c`.
We will define such a function using the mathematical definition of an applicative.
Suppose we have a function of type `a -> b -> c`.
We can uncurry it to a function of type `(a, b) -> c`, from which we can apply the functor \\(F\\) (AKA `fmap`) to to obtain `F (a, b) -> F c`.
Since we are working with an applicative functor, we already have a way to convert `(F a, F b)` to `F (a, b)` using our natural transformation \\(\mu\\).
Thus, by composing this function with the one derived earlier and currying again, we obtain the desired function of type `F a -> F b -> F c`.

# Parting Words
Before we go out and conquer the monad, let's reassess what we have covered thus far.
Functors allow us to "wrap" existing types to form "enhanced" type, as well as provide us a way via `fmap` to lift single-variable functions to act on those functorial values.
Applicatives are an enhancement to our conception of the functor by giving us the capability to lift functions to act on a variable number of functorial values.
In fact, a monad are just applicative functor with more properties and applicatives are often described as living between a monad and a regular functor.
With this knowledge, we are ready to taking a deep look into monads.

#### References
1. [https://en.wikibooks.org/wiki/Haskell/Applicative_functors](https://en.wikibooks.org/wiki/Haskell/Applicative_functors)
2. [https://begriffs.com/posts/2015-08-30-applicative-functors.html](https://begriffs.com/posts/2015-08-30-applicative-functors.html)
3. [Applicative Programming with Effects](https://www.google.com/search?q=applicative+programming+with+effects)  
4. [https://ncatlab.org/nlab/show/monoidal+functor](https://ncatlab.org/nlab/show/monoidal+functor)