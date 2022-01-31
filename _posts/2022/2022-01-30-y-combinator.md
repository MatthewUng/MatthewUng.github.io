---
layout: post
title: Y Combinator
date: 2022-01-30
---

# Recursion with Lambdas
Every entry computer science student should have encountered recursion during their education.
It's just one of many abstract ideas we've come to understand that has no practical usage past graduation.

As an example, here is a possible implementation of the mathematical factorial function using recursion.
```py
def factorial(n):
    if n == 0: 
        return 1
    return n * factorial(n-1)
```

Now imagine a hypothetical computer science, being consumed by hubris, 
attempting to combine his understanding of recursion with his newfound learning of 
lambdas(anonymous functions) that he picked up that day.
```python
lambda n: 1 if n == 0 else n*factorial(n-1) # factorial is not defined 
```
Except the initial solution does not work.  Unlucky.


Eventually, the student stumbles upon the following:
```py
f = lambda g, n: 1 if n == 0 else n * g(g, n-1) # calls input param `g`
f(f, 5) # returns 120
```
The lambda does not have a name for itself, but it does not matter since a reference to itself 
is passed as a function argument.
By calling the first argument with the calling convention `g(g, ...)`, we have obtained a 
recursive implementation of the factorial function using lambdas.

It turns out that we have uncovered something 
reminiscent of the Y-Combinator.

# Fixed-point Combinator
Suppose we have a mathematical function \\(f\\).  
We define the **fix-point** \\(\text{Fix} f\\) of \\(f\\) as some value \\(x\\) such that \\(f(x)=x\\).  
i.e. \\(f(\text{Fix} f) = \text{Fix} f\\)

e.g. Consider the function \\(rot\\) that rotates the points on the x-y plane 90 degrees counter-clockwise.  
\\(rot\\) sends (1,0) to (0,1) to (-1, 0) to (0, -1) and finally back to (1,0).
\\(rot\\) has just one fix-point: the point (0,0).


A **combinator** is a lambda expression that has no [free variables](https://en.wikipedia.org/wiki/Free_variables_and_bound_variables).
A free-variable would be some variable that is not a local variable or a function parameter.
```py
def not_combinator():
    return x+1

def is_combinator(y):
    z = 3
    return y+z
```
The variable `x` in `not_combinator` is a free-variable, but `y` and `z` in `is_combinator` are not free variables. 

# The Y
The mythical Y-combinator that has been hinted at is a fixed-point combinator, meaning that 
it satisfies \\(Y(f) = \text{Fix} f\\) where \\(f\\) is some function.  
When combined with the earlier equality, we also have \\(Y(f) = f(\text{Fix} f) = f(Y(f))\\)

Explicitly, it is defined as \\(Y = \lambda f.(\lambda x.f(x\ x))(\lambda x.f(x\ x)) \\)

We can reduce the application of \\(Y\\) to some function \\(g\\)

$$
Y\ g = (\lambda x.g(x\ x))(\lambda x.g(x\ x))\\
Y\ g= (\lambda x.g(x\ x))(\lambda x.g(x\ x))\\ 
Y\ g= g((\lambda x.g(x\ x))(\lambda x.g(x\ x)))\\ 
Y\ g = g(Y\ g)
$$

The last equation confirms that \\(Y\\) is a fix-point combinator.

Notice that multiple applications of the last equality gives us
\\( Y\ g = g(g(....(g(Y\ g))\\)
which is suggestive of recursion.

In fact, we can use the definition of `Y` to simulate recursion in programming languages 
where functions are objects (the language supports first-order functions).

Let's rehash the equation \\(Y\ g = g(Y\ g)\\) in python.
```py
def Y(g):
    def wrapped(x): # manually curry g(Y(g), - )
        return g(Y(g), x)
    return wrapped

factorial = Y(lambda f, x: 1 if x==0 else x * f(x-1))

factorial(5) # returns 120
```

If we compare this code with the initial code snippet of implementing factorial recursively with lambdas, 
we see that it's semantically identical:  
1. The lambda used in the implementation has been embellished to accept a function as the first argument 
2. The lambda is actually passed in as the first argument, allowing for recursion.

At their cores, both recursive solutions with lambdas leverage the same idea!


# Money
There are actually infinitely many fix-point combinators existing, but none have gained as much attention as Y.
In fact, it's cool enough that a very successful and well known startup accelerator is named after it.
So, if you ever hear of a new venture fund called \\(\Theta\\)-combinator, beware.


#### References
1. [https://en.wikipedia.org/wiki/Fixed-point_combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator)
1. [https://mvanier.livejournal.com/2897.html](https://mvanier.livejournal.com/2897.html)
