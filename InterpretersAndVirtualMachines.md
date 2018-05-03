# Interpreters and Virtual Machines

## Interpreters
  An interpreter is a program that executes another program represented as 
  some kind of data structure.

  Common Representations:
    - Raw Text
    - Trees (AST)
    - Linear Sequence of Instructions

  Interpreters enable the exectution of a program without requring its 
  compilations to native code. The simplify the implementation of programming
  languages and, on modern hardware, are efficient enough for many tasks.

## Text-Based Interpreters
  Directly interpret the textual source of the program. These are very seldom
  used except for trivial languages where every expression is evaluated at
  most once. (no loops or functions). 

  Plausible Example: A calculator

## Tree-Based Interpreters
  Walk over the AST of the program. These allow name/type analysis if 
  applicable, to only be computed once. 

  Plausible Example: Graphing problem, which evaluates functions multiple 
  times to plot it.

## Virtual Machines
  These resemble real processors but are software implemented. They accept
  programs as input as a sequence of instructions. The provide more than
  simple interpretation and abstract away managing memory, threads, and 
  can even do I/O.

  Used in: SmallTalk, Lisp, Forth, Pascal, Java, C#

### Why Virtual Machines? 
  Since the compiler has to generate code for some machine why bother with a
  VM?

  - portability: A compiled VM code can be run on many actual machines
  - simplicity: A VM is usually more high-level than a real machine
  - simplicity, redux: A VM is easier to monitor and profile, which eases debugging

### Drawbacks
  They tend to be slower. This is because of the overhead associated with 
  interpretation: ie. fetcing and decoding, executing, etc. In addition they
  can cause higher numbers of indirect jumps which causes pipeline stalls.
  This can be mitigated by the tendency of VMs to compile the program being
  executed to and to perform optimzations then.

### Kinds of Virtual Machines
  - Stack-Based: use a stack to store intermediate results, variables, etc
  - Register-Based: Mimic a real CPU with a limited set of registers

  It is usually easier to target a stack-based VM than a register based one,
  as register-allocation can be avoided. The JVM is stack based, while Lua is
  register based. 

### Virtual Machine Input
  As the program is a series of instructions the instruction can be identified
  by its opcode which is a single number, and is often called byte code.(java)
  Some instructions have additional arguments which appear after the opcode.

### VM Implementation
  1. The next instruction to execute is fetched from memory and decoded
  2. Operands are fetched, the result compared, and state updated
  3. Repeat

  Many VMs are written in C(++) because they are fast, portable, and have the
  right level of abstraction. 

### VM Optimization
  Techniques:
  - Threaded Code
  - Top-Of-Stack caching
  - Super-Instructions
  - JIT Compilation

#### Threaded Code
  In a switch-based interpreter each instruction requires two jumps
    - one indirect jump to the branch handling the current instruction
    - one direct jump from there to the main loop.
  It would be better to avoid the second one by jumping directly to the code
  handling that specific instruction. This is threaded code.

##### Implementing Threaded Code
  1. Indirect Threading: Instructions index an array containing pointers to the code handling them
  2. Direct Threading: Instructions are pointers to the code handling them.

  Direct Threading could potentially be faster than indirect but on modern 64
  bit architections representing each opcode by a 64-bit pointer is expensive.

##### Threaded Code in C
  To Implement threaded code, it must be possible to manipulate code pointers.  In ANSI C, the only way to do this is with function pointers, but GCC allows
  labels as values which is much more efficient.

##### Direct Threading in ANSI C
  This is easy, but not very efficient. If you define one function per VM 
  instruction, the program can be represented as an array of function pointers
  . Some code is inserted at the end of every function to call the function
  handeling the next VM Instruction. 

  This has a major problem. It can lead to a stack overflow very quickly 
  unless the compiler implements TCE. In our interpreter the function call
  appearing at the end of add (slide 19) and all other VM instructions is a 
  tail call and should be optimized. GCC does TCE but not all compilers do so.
  In these cases you must create your own trampolines or using Baker's Tech.

##### Direct Threading with GCC
  Using labels as values, gcc lets direct threading be done much easier as 
  goto can jump to a computed label. The program can then be represented as
  an array of labels, and jumping to the next instruction is achieved by goto
  instead of function call. 

  This is by far the fastest. 

#### Top-of-Stack Caching
  In a stack-based VM the stack is an array in memory. Since all instructions
  access the stack you can store some of the topmost elements in registers.
  This keeps a fixed number of stack elements in registers which can be a bad
  idea. 

  This can complicate the whole endeavour because you need a vm instruction
  per cache state as defined by the number of stack elements currently cached
  in registers. 

### Static Super-Instructions
  Since instruction dispatch is expensive one way to reduce its cost is to 
  dispatch less. If you group several instractions that often occur in 
  sequence as a super-instruction this can be done.

  i.e. mul is followed by add the two can be made into a madd super 
  instruction.

  Profiling is done to determine which sequences should be transformed and
  the instruction set is modified as such. It is also possible to generate
  super instructions at run time. This is dynmamic super-isntructions. This
  technique can be pushed to its limits by generating one super instruction
  for every basic block of the program. 

### MiniScala VM
  - 32 bit
  - Register Based
  - Only 32 Instructions

#### Memory
  Code is stored at address 0 and the rest is available memory for the heap.
  The address space is virtual and not the same as the host

#### Registers
  - PC: program counter containing the address of the instruction
  - Ib, Lb, and Ob: input, local, output base registers each of which contains either 0 or the address of a heap-allocated block

#### Register Slots
  The slots of the blocks pointed by Ib, etc, are reachable through pseudo 
  registers. I.e. O3 points to index three in Ob. There are 32 input/output
  pseudo registers and 192 local pseudo registers.

#### Function Call and Return
  A function gets its arguments through its input registers, stores local 
  variables in its local registers, and uses output registers to pass 
  arguments. The CALL instruction takes care of saving the caller's context 
  imposed of its base registers. They are Saved in I0-I4 and are implicit 
  arguments to the callee. RET takes care of restoring the caller's context
