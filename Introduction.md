#Introduction to Compilers

## What is a Compiler
  A compiler is a program which translates one progarmming language into
  another. This should be human-friendly to machine friendly and hopefully
  uses the target machine efficiently. 

## Simple Arithmetic
  Abstract Syntax in BNF Notation:
    ~~~
    <exp> := <num>
          | <exp> + <exp>
          | <exp> - <exp>
          | <exp> * <exp>
          | <exp> / <exp>
    ~~~
    BNF = Backus-Naur Form

