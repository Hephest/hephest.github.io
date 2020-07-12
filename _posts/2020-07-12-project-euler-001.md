---
title:  "Project Euler. Problem 1"
layout: post
tags: python algorithms
excerpt_separator: <!--more-->
---

![Project Euler](https://projecteuler.net/themes/20200426/logo_default.png)

## Problem
If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.

Find the sum of all the multiples of 3 or 5 below 1000.

<!--more-->

## Solution

```python
def solve(limit):

    sum = 0

    for number in range(limit):
        if number % 3 == 0 or number % 5 == 0:
            sum += number

    return sum


if __name__ == "__main__":
    print(solve(1000))
```
## Tests

```python
def test_solve_10(self):
    assert problem001.solve(10) == 23

def test_solve_1000(self):
    assert problem001.solve(1000) == 233168
```