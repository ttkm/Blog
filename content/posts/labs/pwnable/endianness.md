---
title: "Endianness"
date: 2026-03-21T17:55:42-05:00
draft: false
aliases:
  - /posts/labs/endianness/

toc: true
images:
tags:
  - c
  - endianness
  - memory
  - pwnable.kr
---

> **Source:** **[pwnable.kr](https://pwnable.kr/)**: walkthrough notes from their **`collision`** challenge. You don’t need an account or to follow along to read this; it’s just context for where the problem came from.

This post will get you familiar with endianness, memory layout, and integer constraints.

We are given this C program running on a 32-bit architecture:

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;

unsigned long check_password(const char* p){
    int* ip = (int*)p;
    int i;
    int res = 0;
    for(i = 0; i < 5; i++){
        res += ip[i];
    }
    return res;
}

int main(int argc, char* argv[]){
    if(argc < 2){
        printf("usage : %s [passcode]\n", argv[0]);
        return 0;
    }
    if(strlen(argv[1]) != 20){
        printf("passcode length should be 20 bytes\n");
        return 0;
    }

    if(hashcode == check_password(argv[1])){
        setregid(getegid(), getegid());
        system("/bin/cat flag");
        return 0;
    } else {
        printf("wrong passcode.\n");
    }
    return 0;
}
```

## Understanding the program

The first thing to notice is this line in `check_password`:

```c
int* ip = (int*)p;
```

This doesn’t create arrays for each character like you might think. It just **reinterprets** the same 20 bytes as 5 integers, each 4 bytes.

So the 20 bytes are grouped like this:

| Bytes | Integer |
|-------|---------|
| `b0 b1 b2 b3` | `ip[0]` |
| `b4 b5 b6 b7` | `ip[1]` |
| `b8 b9 b10 b11` | `ip[2]` |
| `b12 b13 b14 b15` | `ip[3]` |
| `b16 b17 b18 b19` | `ip[4]` |

Then the loop sums them:

```c
res += ip[i];  // adds all 5 integers
```

Basically, the program is just adding 5 numbers that come from your input bytes.

The main function also has this check:

```c
if(strlen(argv[1]) != 20)
```

Which means:

- You must input **exactly 20 bytes**
- No `\x00` allowed, because `strlen` stops early

This is the length/integer constraint we need to respect.

## Strategy to pass the check

We need:

```text
ip[0] + ip[1] + ip[2] + ip[3] + ip[4] = 0x21DD09EC
```

A simple way to do it:

1. Pick 4 integers to be something simple, for example:  
   `A = B = C = D = 0x01010101`
2. Compute the 5th integer:  
   `E = 0x21DD09EC - (A + B + C + D)`

## Endianness

The system is 32-bit **little-endian**, so the last integer `E` must be reversed in memory.

For example, if:

```text
E = 0x1DD905E8
```

Memory layout (little-endian) becomes:

```text
E8 05 D9 1D
```

So the full input must be:

```text
\x01\x01\x01\x01
\x01\x01\x01\x01
\x01\x01\x01\x01
\x01\x01\x01\x01
\xE8\x05\xD9\x1D
```

Total = **20 bytes**.

## Key takeaways

- Input is raw bytes, not printable characters
- The program reinterprets bytes as integers
- Endianness determines byte order
- This challenge is about memory layout + arithmetic, not cryptography

## Check yourself

If someone inputs:

```text
\x1D\xD9\x05\xE8
```

The program will read it as:

```text
0xE805D91D
```

Because of little-endian ordering.
