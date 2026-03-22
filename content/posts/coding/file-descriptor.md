---
title: "File Descriptor"
date: 2026-03-17T18:29:21-05:00
draft: false
toc: false
images:
tags:
  - c
  - descriptor
  - linux
---

> "Mommy! what is a file descriptor in lunix?"

We are given this program and our goal is to find the flag:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char buf[32];

int main(int argc, char* argv[], char* envp[]){
    if(argc < 2){
        printf("pass argv[1] a number\n");
        return 0;
    }
    int fd = atoi(argv[1]) - 0x1234;
    int len = 0;
    len = read(fd, buf, 32);
    if(!strcmp("LETMEWIN\n", buf)){
        printf("good job :)\n");
        setregid(getegid(), getegid());
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;
}
```

## What we notice

1. `argv[1]` gets converted with `atoi` and then subtracted by `0x1234` (4660).
2. The goal is clearly to hit the `strcmp("LETMEWIN\n", buf)` check.
3. `read()` uses that computed value as a **file descriptor**.

At first I thought this was something with `atoi` bugs or a buffer overflow, but that's actually not what's happening.

## The real trick

The real trick is **controlling the file descriptor**.

If we pass **4660**:

```text
fd = 4660 - 4660 = 0
```

`fd = 0` means **stdin**.

So now `read()` is reading directly from our input instead of some file:

```bash
./fd 4660
LETMEWIN
```

This fills `buf` with `"LETMEWIN\n"`, passes `strcmp`, and we get the flag.

## Summary

The vulnerability is basically: letting user input control the file descriptor passed to `read()`.
