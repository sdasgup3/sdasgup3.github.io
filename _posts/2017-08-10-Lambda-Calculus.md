---
layout: post
title:  Lambda Calculus
tags:
- Lambda Calculus
date: 12-09-2017
---

# Lambda Calculus
## syntax
  ```
  E → ID
  E → λ ID. E
  E → E E
  E → (E)
  ```
  - The grammar is ambiguous like
  ```
  xyz could be x(yz) or xy(z)
  λx.yz could be (λx.y)z  or λx.(yz)
  ```
  - The grammar rules are not changed to make it unambiguous, but some
  disambiguation rules are added outside of the grammar.
  ```
  E → E E is left assocative:  xyz == (xy)z
  λ x.yz == λ x.(yz)
  λx.λy.zw == λx.(λy.(zw))
  ```
  Note:
  ``let x = e in e'``
  is nothing but syntactic sugar for

 ``(λ x . e') e``

## Semantics

  - Every ID that we see in lambda calculus is called a ``variable``
  - E →  ID . E is called an abstraction
    - The ID is the variable of the abstraction (also metavariable)
    - E is called the body of the abstraction
  - E → E E
    - This is called an application
  - λ ID . E defines a new anonymous function
    - ID is the formal parameter of the function
    - E Body is the body of the function
  - E → E1 E2, function application, is similar to calling function E1 and
  setting its formal parameter to the actual parameter E2

### Examples
#### Expl I
  - λ x . + x 1 == λ x . (+ x 1)
    - Represents a function that adds one to its argument
  - (λ x . + x 1) 2
    - Represents calling the original function by supplying 2 for x and it would "reduce" to (+ 2 1) = 3
  - Computing with lambda expressions involves rewriting; for each application, we replace all occurrences of the formal parameter ``variable`` in the function body with the value of the actual parameter (a lambda expression). It is easier to understand if we use the abstract-syntax tree of a lambda expression instead of just the text. Here's our simple example application again:

``(λx.x+1)3``
And here's the abstract-syntax tree (where λ is the abstraction operator, and apply is the application operator):
```
        apply
        /   \
       λ     3
      / \
     x   +
        / \
       x   1
```
We rewrite the abstract syntax tree by finding applications of functions to arguments, and for each, replacing the formal parameter with the argument in the function body. To do this,
- we must find an apply node whose left child is a lambda node, since only lambda nodes represent functions.
- The right subtree of the apply node is the argument.
- The left subtree of the apply node (with a lambda at its root) is the function.
- The left child of the lambda is the formal parameter.
- The right child of the lambda is the function body.

There is only one apply node in our example; the argument is 3, the function is λx.x+1; the formal parameter is x, and the function body is x+1. Here's the rewriting step:
```
        apply      =>      +
        /   \             / \
       λ     3           3   1
      / \
     x   +
        / \
       x   1
```
Here's an example with two applications:
(λx.x+1)((λy.y+2)3)

```
        apply         =>   apply     =>  apply   =>  +  =>  6
       /     \             /   \         /   \      / \
      λ       apply       λ     +       λ     5    5   1
     / \       /  \      / \   / \     / \
    x   +     λ    3    x   + 3   2   x   +
       / \   / \           / \           / \
      x   1 y   +         x   1         x   1
               / \
              y   2
```
OR
```
apply         =>    +     =>  +   =>  +  =>  6
/     \             / \       / \     / \
λ       apply     apply 1     +   1   5   1
/ \       /  \      / \       / \
x   +     λ    3    λ   3    3    2
/ \   / \       / \
x   1 y   +     y   +
       / \       / \
y   2     y   2

```

#### Expl II
Note that the result of rewriting a non-pure lambda expression can be a constant (as in the examples above), but the result can also be a lambda expression: a variable, or an abstraction, or an application. For a pure lambda expression, the result of rewriting will always itself be a lambda expression. Here are some more examples:

(λf.λx.fx)λy.y+1
```
        apply      =>   λ        =>    λ        λx.x+1
       /     \         / \            / \
      λ       λ       x  apply       x   +
     / \     / \         /   \          / \
    f   λ   y   +       λ     x        x   1
       / \     / \     / \
      x  apply y  1   y   +
         /  \            / \
        f    x          y   1
```
Note that the result of the rewriting is a function. Also note that in this example, although there are initially two "apply" nodes, only one of them has a lambda node as its left child, so there is only one rewrite that can be done initially.

(λx.λy.x)(λz.z)
```
           apply            λ         λy.λz.z
          /     \          / \
         λ       λ    =>  y   λ
        / \     / \          / \
       x   λ   z   z        z   z
          / \
         y   x
```
(λx.λy.xy)(λz.z)
```
  apply      =>       λ     =>        λ      λy.y
 /     \             / \             / \
λ       λ           y   apply       y   y
/ \     / \             /     \
x   λ   z   z           λ       y
   / \                 / \
  y  apply            z   z
  /     \
 x       y
```

## Currying
  - Technique to translate the evaluation of a function that takes multiple arguments into a sequence of functions that each take a single argument
  - Define adding two parameters together with functions that only take one parameter:
    - λ x . λ y . ((+ x) y)
    - (λ x . λ y . ((+ x) y)) 1
      - λ y . ((+ 1) y)
    - (λ x . λ y . ((+ x) y)) 10 20
      - (λ y . ((+ 10) y)) 20
      - ((+ 10) 20) = 30
  - Example in the context of programming

```C++
  #include <iostream>
  using namespace std;

  int F(int a, int b, int c) { return a + b + c; }

  int F_curry() {
    auto f = [](int a) {
      return [a](int b) { return [a, b](int c) { return a + b + c; }; };
    };

    return ((f(1))(2))(3);
  }

  int main() {
    cout << F(1, 2, 3) << endl;
    cout << F_curry() << endl;
  }
```
## Problems with the naive rewriting rule
### Problem 1
We don't, in general, want to replace all occurrences of x.

To see why, consider the following (non-pure) lambda expression:
``(λx.(x + ((λx.x+1)3)))2``

This expression should reduce to 6;
```
the inner expression: (λx.x+1)3 takes one argument, the value 3, and adds 1, producing 4. The outer expression is now: (λx.(x + 4))2 i.e., it takes one argument, the value 2, and adds 4, producing 6.
```

However, if we rewrite the outer application first, using the naive rewriting rule, here's what happens:

```
  apply
   /\
  λ  2
 / \
x   +                        +
   / \                      / \
 x   apply     =>          2  apply     =>  +    => 5
      / \    (bad              / \         / \
     λ   3   application)     λ   3       2   +
    / \                      / \             / \
   x   +                    x   +           2   1
      / \                      / \
     x   1                    2   1
```
We get the wrong answer (5 instead of 6), because we replaced the occurrence of x in the inner expression with the value supplied as the parameter for the outer expression.

### Problem 2
Consider the (pure) lambda expression

``((λx.λy.x)y)z``

The expression λx.λy.x should simply return the first argument, so in this case the result of rewriting should be y. However, if we use the naive rewriting rule, replacing all occurrences of the formal parameter x with the argument y, we get:
``(λy.y)z``

and now if we rewrite that expression we get
``z``

i.e., we got the second argument instead of the first one!

This example illustrates what is called the ``"capture" or "name clash" problem``.

## Free variable
A variable is free if it does not appear within the body of an abstraction with a metavariable of the same name
- x free in λ x . x y z? No
- y free in λ x . x y z? Yes
- x free in (λ x . (+ x (No) 1)) x (Yes)?
- z free in λ x . λ y . λ z . z  y x? No
- x free in (λ x . z foo) (λ y . y x)? Yes

x is free in E if:
1. E = x
2. E = λ y . E1, where y != x and x is free in E1
3. E = E1 E2, where x is free in E1 or E2


- x free in x λ x . x == x Yes (λ x . x No) --> Yes,  from rule 3
- x free in (λ x . x y) x ? Yes
- x free in λ x . y x ? No

## Bound Variables
- If an occurrence of x is free in E, then it is bound by λ x . in λ x . E
- If an occurrence of x is bound by a particular λ x . in E, then x is bound by the same λ x . in λ z . E
  - Even if z == x
  - Example: λ x . λ x .  x
    - Can also be written as λ y . λ x .  x; So x is bound by λ x preceding it.
  - If an occurrence of x is bound by a particular λ x . in E1, then that occurrence in E1 is tied by the same abstraction λ x . in E1 E2 and E2 E1

Example
  - (λ x . x (λ y . x y z y) x) x y
    - (λ x . x (λ y . x y z y) x) x(Free) y(Free)
  - (λ x . λ y . x y) (λ z . x(Free) z)
  - (λ x . x λ x . z x)
    - (λ x . x) (λ x . z(Free) x)

## Alpha Reduction
To solve [Problem 2](#Problem-2), we use a technique called alpha-reduction. The basic idea is that formal parameter names are unimportant; so rename them as needed to avoid capture. Alpha-reduction is used to modify expressions of the form "λx.M". It renames all the occurrences of x that are free in M to some other variable z that does not occur in M (and then λx is changed to λz). For example, consider λx.λy.x+y (this is of the form λx.M). Variable z is not in M, so we can rename x to z; i.e.,
``λx.λy.x+y alpha-reduces to λz.λy.z+y``

## Beta Reduction
Defined by:
```
(λx. e1) e2 ⇒ e1[e2/x]
```
where the notation e1[e2/x] denotes the result of substituting e2 ``for all free occurrences of x in e1``.

(lambda z. (z z)) (lambda x. lambda y. (x y));
A: for apply

```
    A                        A                 λ        λ
                          /     \              /\       /\
   / \                   λ      λ             w  A     w λ
  /   \                  /\     /\               /\     /\
 λ     λ        ==>     x  λ   x  λ    ->        λ w -> y A
 /\   / \                  /\     /\            /\       /\
z  A  x  λ                w  A   y  A          x λ      w y
   / \    /\                  /\    /\           /\
 z   z   y A                 x w    x y         y A
            /\                                    /\
           x  y                                  x y
```
``λw.(λy.wy)``

## References
  - [Lecture notes of Prof. Adam Doupé](https://adamdoupe.com/teaching/classes/cse340-principles-of-programming-languages-s16/)
  - [ Lambda Calculus (Part I)](http://pages.cs.wisc.edu/~horwitz/CS704-NOTES/1.LAMBDA-CALCULUS.html#overview)
