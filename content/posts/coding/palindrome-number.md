---
title: "Palindrome Number"
date: 2026-04-05T15:49:22-05:00
draft: false
toc: true
images:
tags:
  - c
  - algorithm
  - palindrome
---

This post walks through how to tell if an integer is a palindrome.

>
> Given an integer `x`, return true if `x` is a palindrome, and false otherwise.
>
> ## Examples
>
> **Example 1:**
>
> - **Input:** `x = 121`
> - **Output:** `true`
> - **Explanation:** Same forward and backward.
>
> **Example 2:**
>
> - **Input:** `x = -121`
> - **Output:** `false`
> - **Explanation:** `-121` is not the same as `121-`.
>
> **Example 3:**
>
> - **Input:** `x = 10`
> - **Output:** `false`
> - **Explanation:** `01` is not `10`.
>
> ## Constraints
>
> - `-2^31 <= x <= 2^31 - 1`
>
> **Follow-up:** Solve it without converting the integer to a string.

## Understanding the problem

The first thing to notice is that a palindrome is about **digits**, not the value of `x` as text in code. Negative numbers fail because the minus sign breaks symmetry. Numbers that end in zero (except zero itself) fail as palindromes because there is no leading zero on the left.

Which means:

- We can reject some cases up front in the half-reverse solution (`x < 0`, or `x % 10 == 0 && x != 0`).
- The modulo approach below still gives the right answer for negatives because `while (x > 0)` never runs, `rev` stays `0`, and `0` does not equal a negative `og`.

## Modulo approach

We reverse the number with `x % 10` **(to get the last digit)** and `x / 10` **(to drop the last digit)**, then compare the result to a saved copy of `x`.

A simple way we can do it:

1. Save `og = x` before the loop, since `x` is destroyed inside it.
2. While `x > 0`:
    - `rem = x % 10`
    - `rev = rev * 10 + rem`
    - `x /= 10`
3. After the loop, `rev == og` means palindrome.

For `x = 121`, the steps look like this to better understand properly:

| Step | x (working) | rem | rev |
|------|-------------|-----|-----|
| start| 121         | —   | 0   |
| 1    | 12          | 1   | 1   |
| 2    | 1           | 2   | 12  |
| 3    | 0           | 1   | 121 |

```c
#include <stdbool.h>

bool isPalindrome(int x) {
    double rev = 0, rem, og = x;
    while (x > 0) {
        rem = x % 10;
        rev = rev * 10 + rem;
        x /= 10;
    }
    return rev == og;
}
```

Time is **O(number of digits)**, extra space **O(1)**.

## Half-reverse approach

You only reverse **half** the digits and compare to what is left of `x`. That satisfies the follow-up without building the full reverse and cuts overflow risk on large `x`.

Peel digits from the right of `x` onto `rev` until `rev >= x`. For an odd number of digits, the middle digit ends up on `rev`, `x == rev / 10` ignores that middle digit in the compare. Reject `x < 0` and `x % 10 == 0 && x != 0` up front.

```c
#include <stdbool.h>

bool isPalindrome(int x) {
    if (x < 0 || (x % 10 == 0 && x != 0)) return false;
    long long rev = 0;
    while (x > rev) {
        rev = rev * 10 + x % 10;
        x /= 10;
    }
    return x == rev || x == rev / 10;
}
```

Same asymptotic time and **O(1)** space, still no string conversion.

## Summary

- Modulo 10 will give us the last digit & integer division by 10 removes it.
- Save the original `x` before you destroy it in the loop.
- Half-reverse compares one half of the digits to the other and handles odd length with `rev / 10`.
