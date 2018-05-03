# Tail Calls

## Functional Loops
  Several functional languages do not have looping, instead using recursion 
  as necessary.

## The Problem
  Recursion is not equivalent to looping statements, as they consume stack
  space while loops do not. 

## The Solution
  If the compiler can detect when to replace the call by a jump to the
  beginning of the function our problem would be solved.

  This is called *tail call elimination* 

## Tail Calls
  Calls in terminal position are called tail calls. If they call the function
  itself it is called a recursive tail call. 

## Tail Call Elimination
  When a function performs a tail call its own activation frame is dead. 
  Therefore it is possible to first free the activation frame about to perform
  such a call then load the parameters for the call, and then jump to the
  activiation point. This is TCE

## Tail Call Optimization?
  Without TCE writing a program that loops endlessly using recursion and 
  does not result in a stack overflow is simply impossible. Therefore it is
  required in many functional languages like Scheme. Other languages like C
  it is simply an optimization performed by some compilers.

## Translation of CPS/MiniScala Tail Calls
  When translating CPS/MiniScala to virtual machine code, tail calls must be
  identified and translated appropriately. This is trivial because a function
  call is a tail call iff it gets the return continuation of its enclosing 
  function.

## TCE in Various Environments
  When generating assembly language it is easy to perform TCE, as the target
  language is sufficiently low level to express the deallocation of the 
  activiation frame and following jump

  When targeting higher langauges this becomes difficult.

## Single-Function Approach
  The single funciton approach consists of compiling the program into a single
  function of the target language. This makes it possible to compile the tail
  calls as single jumps within that function and other calls recursive calls
  to it. This is rarely applicable in practice due to limitations in the size
  of functions of the target language. 

## Trampolines
  With trampolines functions never perform the tail call directly. They 
  instead return a special value to their caller, informing it that a tail
  call should be performed. the caller performs the call itself. For this to
  work you must check the return value of all functions to see if a tail call
  should be performed. The code which performs this check is the trampoline

### Extended Trampolines
  These trade space saving for speed. Instead of returning to the trampoline
  every single call the number of successive tail calls is counted at run time
  using a tail call counter passed to every funciton. When that number reaches
  a limit a non-local return is performed to transfer back to the waiting
  trampoline at the bottom of the chain, thereby doing several frames in one 
  go. 

### Non-Local Returns In C
  Extended Trampolines are effecient when a non-local return is used to stack
  dead frames. In C, non-local returns can be performed using the standard
  functions setjmp and longjmp, which can be seen as a form of goto that goes
  across functions
  - setjmp(b) saves its calling environment in b and returns 0
  - longjmp(b,v) restores the environment stored in b and proceeds like if the call to setjmp had returned v instead of 0

## Baker's Technique
  This consists of first transforming the whole program to CPS, where all 
  calls are tailcalls. It is then possible to shrink the whole stack using
  a non-local return. 
