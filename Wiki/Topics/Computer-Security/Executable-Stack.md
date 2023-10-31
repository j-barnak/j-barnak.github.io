# The Executable Stack

**[WIP]**

An **executable stack**, as the name implies, allows for instructions to be executed on the stack. This simple program illustrates how this might work:

```c
#include <stdio.h>

void print_foo() { puts("foo"); }

void print(char *str, void (*function_ptr)()) {
  printf("String that should have been passed: %s", str);
  function_ptr();
}

int main() {
  char string[1024];
  fgets(string, 20, stdin);

  print(print_foo, string); // Arguments switched
}
```

Compiling and running with  `Ya` as the input yields

```
segmentation fault (core dumped)
```

which is what you'd exect to happen. The program attempted to execute an illegal memory location, hence an error is raised. However, if you compile the same code with the `-z execstack` flag and keep the same input, you'll encounter a slightly different error.

```
illegal hardware instruction (core dumped)
```
Why the different error messages? Let's explore what happended in both versions of the same program. The program compiled without the `-z execstack` flag will hereon forth be referred to as `NonExecStackBinary` and the one without, `ExecStackBinary`.

Running `NonExecStackBinary` through gdb and dumping assembly of `print` shows

```
Reading symbols from NonExecStackBinary...
(No debugging symbols found in NonExecStackBinary)
(gdb) b print
Breakpoint 1 at 0x11cb
(gdb) r
Starting program: /home/jared/Workshop/NonExecStackBinary
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Ya

Breakpoint 1, 0x00005555555551cb in print ()
(gdb) disas $rip
Dump of assembler code for function print:
   0x00005555555551c3 <+0>:     endbr64
   0x00005555555551c7 <+4>:     push   rbp
   0x00005555555551c8 <+5>:     mov    rbp,rsp
=> 0x00005555555551cb <+8>:     sub    rsp,0x10
   0x00005555555551cf <+12>:    mov    QWORD PTR [rbp-0x8],rdi
   0x00005555555551d3 <+16>:    mov    QWORD PTR [rbp-0x10],rsi
   0x00005555555551d7 <+20>:    mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555551db <+24>:    mov    rsi,rax
   0x00005555555551de <+27>:    lea    rax,[rip+0xe2b]        # 0x555555556010
   0x00005555555551e5 <+34>:    mov    rdi,rax
   0x00005555555551e8 <+37>:    mov    eax,0x0
   0x00005555555551ed <+42>:    call   0x5555555550a0 <printf@plt>
   0x00005555555551f2 <+47>:    mov    rdx,QWORD PTR [rbp-0x10]
   0x00005555555551f6 <+51>:    mov    eax,0x0
   0x00005555555551fb <+56>:    call   rdx
   0x00005555555551fd <+58>:    nop
   0x00005555555551fe <+59>:    leave
   0x00005555555551ff <+60>:    ret
End of assembler dump.
```

If you recall, according to the System V ABI, parameters to functions are passed in  the registers: rdi, rsi, rdx, rcx, r8, r9, and further values are passed on the stack in reverse order. Therefore, the registers `rdi` and `rsi` are the arguments `string` and `function_ptr` respectively. 

We see that these arguments are passed into local variables

```
   0x00005555555551cf <+12>:    mov    QWORD PTR [rbp-0x8],rdi
   0x00005555555551d3 <+16>:    mov    QWORD PTR [rbp-0x10],rsi
```

The argument `string` is passed into `rax` and the subsequently passed into `rsi` denoting that this is the second argument. Then we see that see the address `[rip+0xe2b]` (gdb graciously tells us this is `0x555555556010`) is passed into `rax` and then `rdi`. We have our two arguments for `printf` and then we call `printf` itself.

```
    0x00005555555551d7 <+20>:    mov    rax,QWORD PTR [rbp-0x8]
    0x00005555555551db <+24>:    mov    rsi,rax
    0x00005555555551de <+27>:    lea    rax,[rip+0xe2b]        # 0x555555556010
    0x00005555555551e5 <+34>:    mov    rdi,rax
    0x00005555555551e8 <+37>:    mov    eax,0x0
    0x00005555555551ed <+42>:    call   0x5555555550a0 <printf@plt>
```

Undefined behavior is invoked because passing a function pointer when the format specifier expected a string. As such, anything can happen and in our case, nothing is printed to the screen.

The undefined behavior is interesting, but far more so, is the second part of `print` where we invoke user supplied input as if it were a function. Before we explore that, it's necessary to understand how `call` works.

There are actually two type of `call` instructions -- near and far. Near calls transfer control to procedures with the code segment (i.e., `.text segment`), and far calls transfer control to procedures in different segments. 

When executing a near call, the processor does the following:
   
1. Push the instruction pointer (`rip`) onto the stack
2. Loads the address of the called procedure into the instruction pointer
3. Continue execution

While discussing `call`s, I think it is worthwhile to explain the mechanisms of the return instruction `ret`

When executing a `ret` instruction:

1. Pop the top of the stack into into `rip`
2. If `ret` has an `N` operand, incremenent the stack by `N` bytes to release the pushed parameters from the stack
3. Resume execution of the stack of the calling procedure

