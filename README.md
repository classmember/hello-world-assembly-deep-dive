# hello world deep dive

The hello.c file contains the C code for hello world.

`hello.c`
```
#include <stdio.h>

int main(int argc, char **argv){
    printf("hello world\n");
}
```

To compile to assembly through llvm using intel syntax for the x86 architecture shell command:
```
$ clang -S -O2 -mllvm --x86-asm-syntax=intel hello.c -o hello.asm
```
The arguments for clang are:
`-S` Only run preprocess and compilation steps
`-O2`  Optimization level 2
`-mllvm` pass arguments to llvm
`---x86-asm-syntax=intel` set assembly syntax to intel style
`hello.c` the file to compile
`-o` specify file for output
`hello.asm` file specified for assembly language

The output:
`hello.asm`
```
	.section	__TEXT,__text,regular,pure_instructions
	.build_version macos, 10, 14
	.intel_syntax noprefix
	.globl	_main                   ## -- Begin function main
	.p2align	4, 0x90
_main:                                  ## @main
	.cfi_startproc
## %bb.0:
	push	rbp
	.cfi_def_cfa_offset 16
	.cfi_offset rbp, -16
	mov	rbp, rsp
	.cfi_def_cfa_register rbp
	lea	rdi, [rip + L_str]
	call	_puts
	xor	eax, eax
	pop	rbp
	ret
	.cfi_endproc
                                        ## -- End function
	.section	__TEXT,__cstring,cstring_literals
L_str:                                  ## @str
	.asciz	"hello world"
```
---
Now to break down each line of assembly using the following reference: https://sourceware.org/binutils/docs/as/
The following is an annotated breakdown:


Declare sections: `__TEXT`, `__text`, `regular`, and `pure_instructions`
```
	.section	__TEXT,__text,regular,pure_instructions
```
---

Define build version. (eg. macOS 10.14.)
```
	.build_version macos, 10, 14
```
---

Specify assembly syntax as intel with no prefixes for values in the assembly syntax.
```
	.intel_syntax noprefix
```
---

[.globl](https://sourceware.org/binutils/docs/as/Global.html#Global)  allows a label such as  `_main` to be visible to the [linker](https://lld.llvm.org/)

This is followed by a comment using the [#](https://sourceware.org/binutils/docs/as/i386_002dChars.html#i386_002dChars) character to mark the main function's starting point.
```
	.globl	_main                   ## -- Begin function main
```
---

[.p2align](https://sourceware.org/binutils/docs/as/P2align.html#P2align) pads the memory location. p2align takes the first operand, multiplies by a power of two (so 4^2 = 16) and uses that value as the number of bytes to pad. p2align's second operand is the data to fill the bytes with. `0x90` in 0x86 is a no operation or `nop`.
```
	.p2align	4, 0x90
```
---

define the [label](https://sourceware.org/binutils/docs/as/Labels.html#Labels) for the `main` function. Labels are used to mark code blocks which can be jumped to using [`jmp`](https://sourceware.org/binutils/docs-2.19/as/i386_002dJumps.html). For more information on control flow in assembly, check out the [x86 Assembly Wikibook](https://en.wikibooks.org/wiki/X86_Assembly/Control_Flow)
```
_main:                                  ## @main
```
---

[.cfi_startproc](https://sourceware.org/binutils/docs/as/CFI-directives.html#g_t_002ecfi_005fstartproc-_005bsimple_005d) declares the start of a [call stack]([https://en.wikipedia.org/wiki/Call_stack](https://en.wikipedia.org/wiki/Call_stack)) frame where the operations are stored.

**Note:** *cfi stands for _call frame information_*
```
	.cfi_startproc
```
---

Comment block of code as "[basic block](http://llvm.org/doxygen/classllvm_1_1BasicBlock.html#details) `0`" 
```
## %bb.0:
```
---

push the rbp register. The rbp register is the base pointer, which points to the base of the current stack frame.
```
	push	rbp
```
---

Set cfa offset to `16` bytes to reserve them from the stack.

**Note:** CFA stands for Canonical Frame Address.
```
	.cfi_def_cfa_offset 16
```
---

[.cfi_offset]([https://sourceware.org/binutils/docs/as/CFI-directives.html](https://sourceware.org/binutils/docs/as/CFI-directives.html)) of the base pointer register to `-16`
```
	.cfi_offset rbp, -16
```
---

Point the base pointer to the stack pointer.
```
	mov	rbp, rsp
```
---

cfi/cfa base pointer
```
	.cfi_def_cfa_register rbp
```
---
lea (load effective address) stores the value of the second argument into the first argument.
The sum of the `rip` register and L_str is stored in the `rdi` register.
```
	lea	rdi, [rip + L_str]
```
---
syscall for the `puts()` function
```
	call	_puts
```
---
Set eax to zero by doing a xor operation between eax and itself, storing the resulting value in the eax register.
```
	xor	eax, eax
```
---

pop the value from the base pointer
```
	pop	rbp
```
---
return from the function (`eax` is returned)
```
	ret
```
---


[.cfi_endproc](https://sourceware.org/binutils/docs/as/CFI-directives.html#g_t_002ecfi_005fendproc) marks the end of the instruction set in the call frame.
```
	.cfi_endproc
```
---

Comments end of `_main` function
```
                                        ## -- End function
```
---

Declare sections: `__TEXT`, `__cstring`, and `cstring_literals`
(Todo: why `__TEXT`, `__cstring`, and `cstring_literals` )
```
	.section	__TEXT,__cstring,cstring_literals
```
---

L_.str section for storing strings to use in the program
```
L_.str:                                 ## @.str
```
---

[.asciz](https://sourceware.org/binutils/docs/as/Asciz.html#Asciz) is like [.ascii]([https://sourceware.org/binutils/docs/as/Ascii.html#Ascii](https://sourceware.org/binutils/docs/as/Ascii.html#Ascii)). It stores an ascii formatted string into a memory space. The difference is that .asciz adds a zeroed-out byte (0x0000) at the end.
```
	.asciz	"hello world\n"
```


---
