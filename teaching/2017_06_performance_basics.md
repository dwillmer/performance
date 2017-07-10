
# Performance Basics

*30th June 2016*

This is the first in a series of posts designed to convey the basic concepts and tools for writing performant code. An intermediate programming ability in one of the major languages is assumed.

### Intro to assembly

To start with, you will need to ensure you have `NASM` installed - `NASM` is the netwide assembler, this lets us write code at almost the lowest level possible, and is useful in order to understand the reasons for performance limitations in current CPU architectures.

To install NASM, go to `www.nasm.us` and download the latest version. This should work on any major operating system, and the code we write will be OS-independent.

Open a command line and type `nasm` or `./nasm` in the correct directory, and confirm that you get an output like this:

```bash
nasm: error: no input file specified
type `nasm -h' for help
```

This confirms that `nasm` is installed correctly.

Here's a complete example 32-bit program in nasm:

```asm
global start

section .text

start:
    push dword msg.len
    push dword msg
    push dword 1
    mov eax, 4
    sub esp, 4
    int 0x80
    add esp, 16

    push dword 0
    mov eax, 1
    sub esp, 12
    int 0x80


section .data
msg db 'My first program', 0xf
msg.len equ $ - msg
```

If you type this into a text editor and save it as `first.asm`, you should be able to 'assemble' it using

| MacOS                   | Linux                 | Windows                 |
| ----------------------- | --------------------- | ----------------------- |
| nasm -f macho first.asm | nasm -f elf first.asm | nasm -f win32 first.asm |

This will output an object file called `first.o`, this then needs linking into an executable using `ld` (Mac/Linux) or `link.exe` (Windows):

```
ld first.o -o first
```

This will output an executable which you can run using:

```
./first
```

The output should be:

```
My first program
```

Make sure you get this output at the command line, and are comfortable running the nasm-ld-run commands (you can put them in a script to make things easier).

#### Structure of an Assembly program

- Assembly programs contain 3 parts: `data`, `bss` and `text` sections.
- The `data` section is for immutable constants.
- The `bss` section is for variables.
- The `text` section is for code.
- Functions are declared with the name followed by a colon. The `start:` block above is a function.
- `start` is a special function, equivalent to the `main` function in C/C++ code - this is executed by default, and is the start of all assembly programs.
- `int 0x80` is the 32-bit command to execute a system call - `int` tells the hardware to perform an interrupt, `0x80` is integer 128 in hex, which on Linux/Mac tells the kernel to perform a system call.
- The exact `syscall` is determined by the value in the `eax` register. In the example above we have put `4` in the `eax` register with `mov eax, 4`, which is the syscall number for `sys_write`.

> In this way, assembler instructions may look backwards to people used to the major imperative languages (Java/C/C++/Python); in assembler, you push arguments onto the stack or into registers, then call the function name.

