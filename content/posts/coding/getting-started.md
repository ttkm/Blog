---
title: "Getting Started with Python The Very Basics"
weight: 0
date: 2026-03-01T11:10:22-06:00
draft: false
toc: true
images:
tags:
  - beginner
  - python
  - simple
---

This post is for someone who's never coded before. We're gonna go from zero what a variable is, what an "if" does, what a loop is, and why indentation matters in Python. No prior knowledge needed.

---

## What you need

- Python installed on your computer ([python.org](https://www.python.org/downloads/) — get the latest 3.x).
- A way to run code: either a simple text file saved as `something.py` that you run from the terminal, or an editor like VS Code / PyCharm.

If you prefer to simply follow along without installing anything, that is completely fine. You can run Python directly in your browser using an online interpreter such as [online-python.com](https://www.online-python.com/) It allows you to write and execute Python code instantly without any setup.

To run a file locally: open a terminal in the folder where your .py file is and type:

```bash
python yourfilename.py
```

That's it for setup. Rest is just writing code and running it.

---

## Variables

A variable is a name that holds a value. You set it once and can use it later.

```python
x = 1
```

This single line is saying: *I'm setting `x` to 1.* So whenever we use `x` in the code, it means 1 (until we change it).

We can use that name to do stuff:

```python
x = 1
print(x)   # prints 1
x = 5
print(x)   # now it prints 5
```

So: **variable = a box with a name. You put a value in it and can read or change it.**

---

## Conditionals (if)

Sometimes you only want to do something when a condition is true. That's what `if` is for.

```python
x = 1
if x == 1:
    print("x is one!")
```

Here we're setting `x` to 1. The second line is **checking if `x` is 1** and if so, it runs the `print` message. If `x` was something else (e.g. 2), that `print` would never run.

- `if` = "only do the next block when this is true"
- `==` = "is equal to" (comparing two things)
- The line under `if` that runs when the condition is true has to be **indented** (we'll get to that in a second).

So: **if the condition is true, run the indented code. Otherwise skip it.**

---

## Indentation (why spaces/tabs matter in Python)

In a lot of languages you use curly braces `{ }` to show "this code belongs to the if" or "this is inside the loop". In Python we don't. We use **indentation** the spaces (or tab) at the start of a line.

Whatever is indented under an `if` (or under a loop) **belongs** to that `if` or loop. So:

```python
if x == 1:
    print("this runs when x is 1")
    print("this too, same block")
print("this always runs, it's not inside the if")
```

If you forget to indent, Python will complain. So: **indent = "this line is part of the thing above it".** Keep it consistent (e.g. always 4 spaces).

---

## Loops what they are

A **loop** is code that runs over and over until a certain condition is met (or until we've gone through a list a set number of times).

Simple analogy: *The loop keeps rolling the dice until the result is 6. So the condition that makes us roll again is "dice is not 6".* That's what a loop is repeat something until a condition is satisfied.

We'll look at two kinds: `while` and `for`.

---

## While loop

A `while` loop runs **while** a condition is true. When the condition becomes false, we stop.

```python
import random
dice = 0
while dice != 6:
    dice = random.randint(1, 6)   # roll 1–6
    print("Rolled:", dice)
print("We got a 6, we're done!")
```

So: **while "dice is not 6", keep rolling and printing. When we get 6, we leave the loop and run the last line.**

---

## For loop

A `for` loop is often used to "go through" a range of numbers or a list, one by one.

In other languages you might see something like:

- Start with `i = 0`
- Keep going **while** `i < n` (i is below n)
- After each run, do `i++` (increase i by 1)

So that one line is saying: *I'm setting `i` to 0; then "i < n" means "if it's below n, keep going"; and we keep increasing i until the threshold is met.* The rest of the logic inside the loop can get more complex, but that's the idea of the loop itself.

In Python we usually write it like this:

```python
n = 5
for i in range(n):
    print(i)
```

This prints 0, 1, 2, 3, 4. So `i` takes each value in that range, one after the other. Same idea: we're "looping" through a set of values.

If we're looping through a list:

```python
fruits = ["apple", "banana", "cherry"]
for f in fruits:
    print(f)
```

So: **a for loop = "do this block once for each item (or each number in a range)".**

---

## Putting it together

- **Variables** — names that hold values (`x = 1`).
- **Conditionals** — run code only when something is true (`if x == 1:`).
- **Indentation** — in Python, what's indented under an `if` or loop belongs to it.
- **Loops** — repeat code until a condition is met (`while`) or for each item in a range/list (`for`).

That's the very basics. From here you can start writing small scripts, then look up things like lists, functions, and reading input. If something here was confusing, re-read that section and try changing the examples (e.g. change `x` to 2 and see what happens). Messing with the code is the best way to get it. Don't be afraid to make mistakes and run failed codes part of learning.
