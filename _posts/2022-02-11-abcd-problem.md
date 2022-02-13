---
layout: post
title: "ABCD problem"
date: 2022-02-11
categories: python
excerpt_separator: <!--more-->
---

### ABCD
This question is taken from [Art of Problem Solving](https://artofproblemsolving.com/community/c1105845h2728519_diabolical_digits).
The question asks us:

> Can you find a four-digit number ABCD satisfying ABCD = C * D^A?
<!--more-->

### Maths
With 4 variables that could range from 0-9 (except A), it is obvious that we should try to reduce the solution space. By spotting that the value ABCD is mostly determined by D^A, the number of possible pairs (D, A) is greatly reduced by limiting it between 999 < D^A < 10,000. Then, the number of possible values of C for each pair (D, A) is further reduced. Then the solution space is small enough for simple brute force checking.

### Python
Instead, we can brute force check every single possible 4 digit number to see if the equation holds. First, enumerate all the possibilities. One way to do this is through nesting 4 for loops, but recursion can provide more concise code (No efficiency improvements though, since the number of distinct 4 digit numbers is constant, and each number is checked).

```python
permutations = []

def permute(currentpermutation, length):
    if length == 0:
        permutations.append(currentpermutation.copy())
        return
    else:
        for num in range(0, 10):
            if num not in currentpermutation:
                currentpermutation.append(num)
                permute(currentpermutation, length-1)
                currentpermutation.pop()

permute([], 4)
```

The length variable allows us to generalize to n-digit numbers shall we ever need it. Then we just have to check if the equation is satisfied for each possibility.

```python
for permutation in permutations:
    num = permutation[0] * 1000 + permutation[1] * 100 + permutation[2] * 10 + permutation[3]
    if num == permutation[2] * (permutation[3] ** permutation[0]):
        print(num)
```

The python code correctly outputs 4375, the only solution to the equaion.


[Back](/)
