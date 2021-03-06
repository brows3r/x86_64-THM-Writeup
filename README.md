# THM x86_64 Assembly Writeup
My THM writeup on Assembly.

# Writeup (Below This)

URL: https://tryhackme.com/room/introtox8664
-----------------------------------------------------------

=================================================================================

[Introduction]

=================================================================================

Computers execute machine code, which is encoded as bytes, to carry out tasks on a computer. Since different computers have different processors, the machine code executed on these computers is specific to the processor. In this case, we’ll be looking at the Intel x86-64 instruction set architecture which is most commonly found today. Machine code is usually represented by a more readable form of the code called assembly code. This machine is code is usually produced by a compiler, which takes the source code of a file, and after going through some intermediate stages, produces machine code that can be executed by a computer. Without going into too much detail, Intel first started out by building 16-bit instruction set, followed by 32 bit, after which they finally created 64 bit. All these instruction sets have been created for backward compatibility, so code compiled for 32 bit architecture will run on 64 bit machines. As mentioned earlier, before an executable file is produced, the source code is first compiled into assembly(.s files), after which the assembler converts it into an object program(.o files), and operations with a linker finally make it an executable. 


The best way to actually start explaining assembly is by diving in. We’ll be using radare2 to do this - radare2 is a framework for reverse engineering and analysing binaries. It can be used to disassemble binaries(translate machine code to assembly, which is actually readable) and debug said binaries(by allowing a user to step through the execution and view the state of the program). 


The first step is to execute the program intro by running

./intro

Which then just shows the following output

(Link to the image -> https://lh5.googleusercontent.com/JjT_G7sF5ScGMJWTisYH3N49djt64Dx2_6CkOtXBSezoheO0uo7wlu0FQBLBLTyjA_PsRDHrRYTYrvqtA0NVFG0Kt2EGosxx7QvBf32cEjSMSYEOh85uRFJFKy2AxLhsovfUTT9O)

From the execution, it can be seen that the program is creating two variables and switching their values. Time to see what it’s actually doing under the hood!


Go to the introduction folder on the virtual machine and run the command: r2 -d intro

This will open the binary in debugging mode. Once the binary is open, one of the first things to do is ask r2 to analyze the program, and this can be done by typing in: aa

Which is the most common analysis command. It analyses all symbols and entry points in the executable.

The run

e asm.syntax=att

to set the disassembly syntax to AT&T.


The analysis in this case involves extracting function names, flow control information and much more! r2 instructions are usually based on a single character, so it is easy to get more information about the commands. For general help, run: ?

For more specific information, for example, about analysis, run: a?


Once the analysis is complete, you would want to know where to start analysing from - most programs have an entry point defined as main. To find a list of the functions run: afl

(Link to the image -> https://lh4.googleusercontent.com/OMdwgZHBcZxoBjRON-zmPmdlfeaCcZUstR0S5qev7mofmxTEGwVzkZAenUYlKXEy94wBWA8XoSsWQnXbwAroPPj2gq1rrrytoavs-Vc97PwK9eblUtGx-DBj3EMHS7xXN5Jn2_9f)

As seen here, there actually is a function at main. Let’s examine the assembly code at main by running the command

pdf @main

Where pdf means print disassembly function. Doing so will give us the following view

(Link to the image -> https://lh4.googleusercontent.com/HometWAQT4JO7lJN5-tipL_tiBL8T270njUm4bTTdIIXIXOm3oEb41YhuUcq1dl0oK5b_y5QfqbzZJlDsPQKQ-G7LMVqPADbpz1uvD6TfCM7UONbEAmAVn_bae7W2Rpj2dfZDJDV)


As we can see from above, the values on the complete left column are memory addresses of the instructions, and these are usually stored in a structure called the stack(which we will talk about later). The middle column contains the instructions encoded in bytes(what is usually the machine code), and the last column actually contains the human readable instructions. 

The core of assembly language involves using registers to do the following:

Transfer data between memory and register, and vice versa

Perform arithmetic operations on registers and data

Transfer control to other parts of the program

Since the architecture is x86-64, the registers are 64 bit and Intel has a list of 16 registers:


64 bit
------
%rax
%rbx
%rcx
%rdx
%rsi
%rdi
%rsp
%rbp
%r8
%r9
%r10
%r11
%r12
%r13
%r14
%r15

32 bit
------
%eax
%ebx
%ecx
%edx
%esi
%edi
%esp
%ebp
%r8d
%r9d
%r10d
%r11d
%r12d
%r13d
%r14d
%r15d


Even though the registers are 64 bit, meaning they can hold up to 64 bits of data, other parts of the registers can also be referenced. In this case, registers can also be referenced as 32 bit values as shown. What isn’t shown is that registers can be referenced as 16 bit and 8 bit(higher 4 bit and lower 4 bit). 

The first 6 registers are known as general purpose registers. The %rsp is the stack pointer and it points to the top of the stack which contains the most recent memory address. The stack is a data structure that manages memory for programs. %rbp is a frame pointer and points to the frame of the function currently being executed - every function is executed in a new frame. To move data using registers, the following instruction is used:

movq source, destination

This involves:

Transferring constants(which are prefixed using the $ operator) e.g. movq $3 rax would move the constant 3 to the register

Transferring values from a register e.g. movq %rax %rbx which involves moving value from rax to rbx

Transferring values from memory which is shown by putting registers inside brackets e.g. movq %rax (%rbx) which means move value stored in %rax to memory location represented by %rbx.

The last letter of the mov instruction represents the size of the data:



Intel Data Type Suffix Size (bytes)

Byte
----
b
1

Word
----
w
2

Double Word
-----------
l
4

Quad Word
---------
q
8

Quad Word
---------
q
8

Single Precision
----------------
s
4

Double Precision
----------------
l
8



When dealing with memory manipulation using registers, there are other cases to be considered:

(Rb, Ri) = MemoryLocation[Rb + Ri]
D(Rb, Ri) = MemoryLocation[Rb + Ri + D]
(Rb, Ri, S) = MemoryLocation(Rb + S * Ri]
D(Rb, Ri, S) = MemoryLocation[Rb + S * Ri + D]

Some other important instructions are:

leaq source, destination: this instruction sets destination to the address denoted by the expression in source
addq source, destination: destination = destination + source
subq source, destination: destination = destination - source
imulq source, destination: destination = destination * source
salq source, destination: destination = destination << source where << is the left bit shifting operator
sarq source, destination: destination = destination >> source where >> is the right bit shifting operator
xorq source, destination: destination = destination XOR source
andq source, destination: destination = destination & source
orq source, destination: destination = destination | source

Before understanding how programs work, it is important to understand registers, memory manipulation and some basic instructions. The next sections will have more hands on use of radare2.

=================================================================================

[If Statements]

=================================================================================

The general format of an if statement is

# START CODE EXAMPLE
---------------------------------------------------------
if(condition){

  do-stuff-here

}else if(condition) //this is an optional condition {


  do-stuff-here

}else {


  do-stuff-here

}
---------------------------------------------------------
# END CODE EXAMPLE


If statements use 3 important instructions in assembly:

cmpq source2, source1: it is like computing a-b without setting destination
testq source2, source1: it is like computing a&b without setting destination



Jump instructions are used to transfer control to different instructions, and there are different types of jumps:

==================================
| Jump Type | Description        |
==================================
| jmp       | Unconditional      |
| je        | Equal/Zero         |
| jne       | Not Equal/Not Zero |            
| js        | Negative           |
| jns       | Nonnegative        |
| jg        | Greater            |
| jge       | Greater or Equal   |
| jl        | Less               |
| jle       | Less or Equal      |
| ja        | Above (unsigned)   |
| jb        | Below (unsigned)   |
==================================

The last 2 values of the table refer to unsigned integers. Unsigned integers cannot be negative while signed integers represent both positive and negative values. SInce the computer needs to differentiate between them, it uses different methods to interpret these values. For signed integers, it uses something called the two’s complement representation and for unsigned integers it uses normal binary calculations. 

=================================================================================

[If Statements Continued]

=================================================================================

Go to the if-statement folder and Start r2 with r2 -d if1

And run the following commands:
aaa
afl
pdf @main

This analyses the program, lists the functions and disassembles the main function.

(Link to image -> https://lh4.googleusercontent.com/SWXZLnHK52fyB4BtLsq4b-YC0uucB8P219xVEc4ilFrGiFf0usbMzzzuzx1m3KEF94__4Ox9sCP256VVHkWUOx3DUhVcS9a03eG3FONST3C2gCD9Kt8pCmmM2r-6rl1TFOeMkLGk)

 We’ll then start by setting a break point on the jge and the jmpinstruction by using the command:

db 0x55ae52836612(which is the hex address of the jgeinstruction) 

db 0x55ae52836618(which is the hex address of the jmpinstruction)

We’ve added breakpoints to stop the execution of the program at those points so we can see the state of the program. Doing so will show the following:

(Link to image -> https://lh6.googleusercontent.com/9aI231aVGvJr4mWImailUL5Z0zjQ-IuOnHgKxybK2jX-bAXp2uHlqggTTdLtwyANTyq_Q1anXDgnUl1Goxe9WhFGi6n5QcKzef9vAfnRdfycB5Q2icI8ZOGrafnmP2PomCjOsOCk)

We now run dcto start execution of the program and the program will start execution and stop at the break point. Let’s examine what has happened before hitting the breakpoint:
The first 2 lines are about pushing the frame pointer onto the stack and saving it(this is about how functions are called, and will be examined later)
The next 3 lines are about assigning values 3 and 4 to the local arguments/variables var_8h and var_4h. It then stores the value in var_8h in the %eax register. 
The cmplinstruction compares the value of eax with that of the var_8h argument

To view the value of the registers, type in: dr

(Link to image -> https://lh5.googleusercontent.com/dIlnkagpBvm0pX7AFwYDStfJp4UqA48PUmOv2qf_BcZwVu7OsgYoKInNZ16iv_k4xbC3XqUxB8IbVbscKtnQ2TmdhHRIpWuTbvezdjd6ZcHfcSv3H1heeD05-K4Se9e_MCi9Qdw1)

We can see that the value of rax, which is the 64 bit version of eax contains 3. We saw that the jge instruction is jumping based on whether value of eax is greater than var_4h. To see what’s in var_4h, we can see that at top of the main function, it tells us the position of var_4h. Run the command: px @rbp-0x4
And that shows the value of 4. 

We know that eax contains 3, and 3 is not greater than 4, so the jump will not execute. Instead it will move to the next instruction. To check this, run the ds command which seeks/moves onto the next instruction.

(Link to image: https://lh4.googleusercontent.com/jGtJeqPVX_GL4UOwOjx0i7KigX69yt2MsROx5vO0k3l5IuM9-MwU7JyD67lbbkTohGZueOMnGYzpUatkXoMTKTtWMe0f8KhwSj7hXcEDcpWZ7I1Vu6-MbVBDG1msi2kya_95eOdt)

The rip(which is the current instruction pointer) shows that it moves onto the next instruction - which shows we are correct. The current instruction then adds 5 to var_8h which is a local argument. To see that this actually happens, first check the value of var_8h, run ds and check the value again. This will show it increments by 5.

(Link to image: https://lh5.googleusercontent.com/epOtH0brvnOKjW9GLbv8ZgcSUsREsGrMJrcMh0HJkXlBoR_kLhmJp4CDUBb8U6BWkNneIOEkteXP4wH69OgvI8h2Aq3Cufi_TD3huKkJ3FtYDMI47kWwh89IFhyfutypuvQTnf7R)

Note that because we are checking the exact address, we only need to check to 0 offset. The value stored in memory is stored as hex.
The next instruction is an unconditional jump and it just jumps to clearing the eax register. The popqinstruction involves popping a value of the stack and reading it, and the return instruction sets this popped value to the current instruction pointer. In this case, it shows the execution of the program has been completed. To understand better about how an if statement work, you can check the corresponding C file in the same folder.

=================================================================================

[Loops]

=================================================================================

Usually two types of loops are used: for loops and while loops. The general format of a while loops is:

while(condition){

  Do-stuff-here

  Change value used in condition

}


The general format of a for loop is

for(initialise value: condition; change value used in condition){

  do-stuff-here

}


Let’s start looking up loops by entering the loops folder, running r2 with the loops 1 file. After this, analyse everything, list the functions and disassemble the main function. 

(Link to image: https://lh4.googleusercontent.com/OKIkdD0MD_xvJ8zqJpR8LJBnffeOjeoWXRHFQ1uahwqmrfB-t6tctxc-8Nfm1t4gS_nwR61ekl1x4bVvY4mslLbjfaqbtKfs4onYHxaHr7dt1jAbfj59W7xdVtJOjAMXnqGFo67O)

Let start of by setting a break point at the jmp instruction using the command: db address-of-instruction
Doing this allows use to skip the first few lines of instructions, which as we saw using if statements, it just passing in values to local arguments(note that the constant showed by $0xa represents that value of 10 in hex). Once execution reaches the breakpoint at the jmp instruction, run ds to move to the next instruction. Since this is an unconditional jump, it will move to the cmpl instruction.

(Link to image: https://lh6.googleusercontent.com/mMG7wWP3vWB59e_2EtZuRgl8b_g4PwYklXzn8SaKCL5zLHUZmbmQhnO6JBUEiZvWG5JBQPR2WQUmkZFA5arFafm88l_c2rhjILTlhf064o7wO9Zmg99eq6iqvoqHYdDMebmZuURM)

Here the cmplinstruction is trying to compare what’s in the local argument var_ch with the value 8. To see what’s in var_ch, check the start of the disassembled function and check the memory. In this case, it is rbp-0xc

(Link to image: https://lh3.googleusercontent.com/KL9Y3euzWtQh-FqiylEiSpoEerjE8zHoetxHMqmbth5-mCw0ETwNubCaibWDXV7WIGo9IecXZPjFQ88xy-JC9dPkCTAFO-yBlRb5OG_Yy6jY87MFM-XqF1WI7PFzXDsXJgXIAaiI)

And shows that it contains 4. The next instruction is a jle which is going to check is the value is var-ch is less than or equal to 8. Since 4 is less than 8, it will jump to the addlinstruction. 

(Link to image: https://lh6.googleusercontent.com/W2rtR7Df_6PWd2FiKcixUJA92dPgYb3ISpwgdfA-ONMhfM_WpgrpRVziXBSjDy2fj3pWmHanGfr_Dhck7bIq9__lfH3IgGRJDl-PpRuCn761XBCWTRpaavCSHsTbthx_VTyx_kaL)

The addlinstruction will add 2 to the value of var-ch and continue to go to the cmplinstruction. Since 2 was added to var_ch, var_ch will now contain 6 which is still less than 8, and it will jump back to the addlinstruction. This can be seeing by continuing execution using the dsstatement. We know this is a loop because the addlinstruction is being executed more than once, and this is in combination with comparing the value of var_ch to 8. So we can infer the structure of the loop to be

while(var_ch < 8){

 var_ch = var_ch + 2

}

A quicker way to examine the loop would be to add a break point to cmplinstruction and running dc. Since this is a loop, the program will always break at the cmplinstruction(because this instruction checks the condition before executing what is inside the loop). You can check the loop1.c file to see the structure of the loop!

The end!
Hope you enjoyed my notes and happy learning! 
