# Optimizations

## The goal of aptimizations:
  Rewrite a progam being compiled to a new program 
  Behaviourally equivalent to the original
  Better in some respect - faster, smaller, more energy efficient, etc...

## Two Classes:
  Machine-Independent optimizations: constant folding or dead code elimination - high level not dependent on target architecture
  Machine-Dependent optimizations: register allocation or instruction scheduling - low level and dependent on target architecture

## The Importantance of IRs
  The Intermediate representation on which rewriting optimzation are performed can have a dramatic impact on their ease of implementation.
  Rewriting optimizations work in two steps:
    The program is analyzed to find rewrite oppurtunities
    The Program is rewritten based on the oppurtunities identified in the first step
  The IR should make both of these steps should make it easy to do 

## Types of Optimizations

### Case 1: Constant Propogation
  If you have an assignment like x := 7 we have to wonder if it is legal to just replace x with 7 at every single oppurtunity
  If multiple assignment to the same variable is allowed we have to do data-flow analysis to decide where x is no longer equal to 7 at a later time
  If it is prohibited then by all means go ahead and replace it with 7!

### Other Simple Optimizations
  Common Subexpression Elimination: avoiding the repeated evaluation of expressions
  Dead Code Elimination: removing assignments to variables whose value is not used later
  Etc.
  In All cases analyses are required to distinguish the various "versions" of a variable that appear in the program, so a good IR should not allow multiple assignments to a variable

### Case 2: Inlining
  Also called in-line expansion consists of replacing a function call by a copy of the body of that function with parameters replaced by the actual arguments. 
  This often opens up the door to further compiler optimizations
  IR also can affect this, and problems can arrise in examples like inlining directly to AST.

#### Naive Inlining: Problem #1
  ~~~
  def printReturn(x: Int) = { printInt(x): x};
  def twice(y: Int) = y + y; 
  def f(z: Int) twice(printReturn(z)); 
  ~~~
  When you direcetly inline this it will incorrectly inline to print the argument passed to z twice and add them. 
  ~~~
  def f(z:Int) = printReturn(z) + printReturn(z); 
  ~~~
  Instead of correctly printing the original argument a single time and then returning twice it's value.
  A possible solution is bind actual parameters to the variables using a Let to ensure they are evaluated at most once. 
  //TODO understand this solution

#### Naive Inling: Problem #2
  ~~~
  def first[T, U](t: T, u:U) = t; 
  def firstReturn(z:Int) = first[Int, Unit](z, printInt(z)); 
  ~~~
  This results in the incorrect inlining of first in printReturn:
  ~~~
  def printReturn(z:Int) = z; 
  ~~~
  Such that z is in fact never printed. 
  In this case the possible solution is to bind the actual parameters to variables using a let so they are again atleast evaluated once. 

#### Easy Inlining
  The solution in both the above cases is to bind the actual arguments to a variable using a let and using them in the body of the inlined function.
  A properly-designed IR can avoid these problems altogether by ensuring that actual pramaters are always atoms (variables/constants)
  So a good IR should only allow atomic arguments to functions, or should force them itself in the ir compilation stage

### IR Comparison
  Standard CTL/CFG is bad in that its variables are mutable; however it allows only atoms as functions which is good for inlining
  RTL/CFG in SSA Form, CPS/Miniscala and similar functional IRs are good in that variables are immutable and they only allow atoms as function arguments so it can do dead code elimination as well as
  inling

## Simple CPS Optimizations - Rewriting Optimizations
  The rewriting Optimizations for CPS/Miniscala are specified a s aset of rewriting reules of the form T->_opt_U
  These rules rewrite the CPS term T to a functional equivalent but hopefully more efficient U

### Optimization Contexts
  The rewriting rules cannot be applied just anywhere
  It would be inappropriate to rewrite the paramater list of a function. This constraint can be captured by specifiying all the contexts in which it is valid to rewrite a term by a single whole `[]`
  The whole of the context C can be plugged with a term T, written as `C[T]`
  I.E.
  if C is `if [] ct else cf` then `C[x < y]` is:
  ~~~
  if (x < y) ct else cf
  ~~~

  The optimization contets for CPS are generated by the following grammar:
  ~~~
  C_opt ::= []
    | val_l n = l; C_opt
    | val_p n = p(n1, ...); C_opt
    | def_c c1...; def_c ci(ni1,...) = {C_opt}; ...; defc ck ... ; e
    | def_c c1 ...; C_opt
    | def_f f1...; def_f fi(ci, ni1, ...) = {C_opt}; ...; def_f fk...; e
    | def_f f1...; C_opt
  ~~~

### Optimization Relation
  The slides optimize writing `C_opt[T] => opt C_opt[U]` to `T ->_opt U`

## (Non-)shrinking rules
  There are two types of rewriting rules:
    Shrinking rules which rewrite a term to an equivalent but smaller one
    Non-Shrinking Rules rewrite the term to an equivalent but _potentially_ larger one
  Shrinking rules can be applied at will until the term is no longer reducable but nonshrinking rules could cause infinite expansion. 
  Heuristics must be used to determine when non-shrinking rules are appropriate
  Execept for non-linear inlining, all optimizations are shrinking (that we cover) 

## Dead Code Elimination 
  Goal: eliminate code that does not perform side effects or produce a later used varaible
  ~~~
  val_l n = l; e
  ->opt e [if n is not free in e]
  ~~~

  ~~~
  val_p n = p(n1, ...); e
  ->opt e [if n is not free in e and p \ne {byte-operations}]
  ~~~

  ~~~
  def_f f1...; def_f fi...; def_f fk...; e
  -> opt e def f1; ...; def_f fi-1; def_f fi+1; def_f fk...; e 
  [if f_i is not free in {f1, ..., fi-1, fi+1, ..., fk, e}]
  ~~~
  The rules for continuations is similar to the one for functions
  //TODO recover Free rules

  The rewriting rules to eliminate dead functions does not eliminate dead but mutually recursive functions
  To handle them you identify all functions transitively reachable from it. All unreachable functions are dead.
  The rule for continuations has the same problem with the same solution but the only difference is that, continuations being local to functions, the reachability 
  analysis can start in function bodies

## CSE (Common Subexpression Elimination)
  ~~~
  val_l n1 = l; C_opt[val_l n2 = l; e]
  ->opt val_l n1 = l; C_opt[e[n2 -> n1]]

  val_p n1 = a1 + a2; C_opt [val_p n2 = a1 + a2; e]
  ->opt val_p n1 = a1+a2; Copt[e[n2 -> n1]]
  ~~~
  etc. 

  These simple rules for common subexpression elimination are also a bit too naive and miss some things due to scoping
  I.e.
  ~~~
  def_c c1() = {
    val_p n1 = y + z; ... 
  }; 
  def_c c2() = {
    val_p n2 = y + 1; ...
  }; 
  ...
  ~~~

### Eta Reduction
  Variants of eta-reduction can be performed to remove redunant definitions
  ~~~
  def_c c1(...) = {e1}; ...;
  def_c ci(n1,...) = { d(n1,...)};  // where d is a continuation
  def_c ck(...) = {e_k}; e
  ->opt def_c c1(...) = {e1[c_i -> d]}; ...; def_c ck(...) = {ek [ci->d]}; 
    e[ci->d]

  def_f f1(...) = {e1}; ...;
  def_f fi(c, n1,...) = { g(c, n1,...)};  // where g is a continuation
  def_f fk(...) = {e_k}; e
  ->opt def_c f1(...) = {e1[fi -> g]}; ...; def_f fk(...) = {ek [fi->g]}; 
    e[fi->d]
  ~~~

### Constant Folding
  Constant folding is the act of replacing a constant expression by its value
  I.e. if you use two variables to store values, then add those two variables together to assign to a third variable, why not simply assign the addition of the the two values to the third variables
  after which you can eliminate the two stored variables as dead code and reduce memory calls. 

  Primatives appearing in conditional expressions are also amenable to constant folding 
  I.e.
  ~~~
  if (n == n) ct else cf -> opt ct()

  if (n != n) ct else cf -> opt cf()

  val_l n1 = l1;
    Copt[ val_l n2 = l2;
      Copt[ if (n1 == n2) ct else cf]]
  ->opt
  val_l n1 = l1;
    Copt[ val_l n2 = l2; 
      Copt[ ct() ]] //if l1 ==l2 else replace it with cf
  ~~~

### Neutral/Absorbing Elements
  Use of neutral and absorbing elements of arithmetic primitives can be simplified. For multplication this is expressed as follows:
  ~~~
  val_l n1 = 0; Copt[val_p n = n1 * n2; e]
  -> opt val_l n1 = 0; Copt[ e[n -> n1 ]]
  ~~~
  I.e. if the action always results in a certain outcome (multiplication by 0 is 0, multiplication by 1 is is the same value, (same for addition)) you do not need to actually perform the primitive 
  action

### Block Primitives
  These are harder to optimize because the can always be modified.
  However blocks used to implement closures are known constant so you can rewrite those
  ~~~
  val_p b = block-alloc-k(s);
    Copt[ val_p t = block-set(b,i,v); 
      Copt[ val_p n = block-get(b, i); e]]
  ->opt
  val_p b = block-alloc-k(s); 
    Copt[ val_p t = block-set(b,i,v); 
      Copt [ e[n->v] ]]
  ~~~
  //TODO perform excercise optimizing block primitives for all block operations

## Shrinking Inlining
  A non-recursive continuation or function that is applied exactly once, i.e. used in a linear fashion, can always be inlined without making code grow
  ~~~
  def_f f1...; def_f fi(ci, ni1,...) = {e}; ...; def_f fk...;
    Copt[fi(c, m1,...) ]
  ->opt
  def_f f1...; def_f fi-1...; def_f fi+1...; ...; def_f fk...;
    Copt[e1[ci->c, ni,1 -> m1]]
    //[when fi is not free in Copt, e1, ..., ek]
  ~~~
  The rule for continuations is similar. 

## General Inlining
  Non-linear inlining can also be performed trivially in CPS/Miniscala
  ~~~
  ...; def_f fi (c1, ni1,..) = {ei}; ...;
  Copt[ f(i, c,m1,...)]
  ->opt
  ...; def_f fi(ci, ni1,...) = {ei}; ...;
  Copt[ei[ci -> c, ni1, m1]]
  ~~~
  To preserve name uniqueness of names, fresh versions of bound names should be created during general inlining
  The problem of these rules is that they are not shrinking and rewriting does not even terminate with recursive continuations

## Inling Heuristics
  Since non-shrinking inlining cannot be performed always we need to decide when a candidate function should be inlined
  Several Factors:
    The size of the candidate function - smaller ones should be inlined more eagerly than bigger ones
    A function called only a few times should be inlined
    The nature of the candidate - not much gain can be expected from inlining a recursive function
    Constant arguments could lead to further reductions in the inlined candidate especially if it combines them with other constants

## Contification
  This is the name given to an optimization that transforms functions into local continuations
  When applicable the transformation is interesting because it transforms expensive functions - compiled as closures - to inexpensive closures compiled as code blocks

  A CPS/MiniScala function is contifiable iff it always returns to the same location because then it does not need a return continuation
  For non-recursive functions, this condition is satisfied iff that function is only used in App_F nodes in function position, and always passed the same return continuation
  For Recursive functions the condition is more involved

### Non-Recursive Contifaction
  f can be contified if it does not appear free in either of the optimization contexts and the second context is the smallest multi-hole context enclosing all applications of f.
  This ensures the scope of the continuation is as small as possible and therefore that m obeyes the scoping rules for continuations
  
### Recursive Contifiability
  A set of mutually recursive functions is contifiable iff every function in F is always used in one of the following ways:
    applied to a common return continuation, or
    Called in tail position by a function in f.
  This ensures all functions in F eventually return throught the same common continuation
  //Example on slide 39

  This can only be true if all the non-tail calls to functions in F appear either
    In the term e, or
    In the body of exactly one function f_i \ne F
  Therefore two sepearate rewriting rules must be defined

  ~~~
  def_f f1(c1, n1, ...) = {e1}; ...; def_f fk...;
    Copt[e]
  -> opt
  def fi+1(ci+1, ni+1,1,...) = {ei+1}; ...; fk...;
    Copt[def def_fc m1(n1,1...) = {e1^*[c1->c0]}; e^*]
    // where f1, ..., fi, do not appear free in Copt and e is minimal
  ~~~
  ~~~
  def_f f1(c1, n1,1 , ...) = {e1}; ..; 
  def_f fk(ck, nk,1, ...) = { Copt[ ek] }; e
  ->opt
  def_f fi+1(ci+1, ni+1,1, ...) = {ei+1}; ...; 
  def_f fk(ck, nk,1, ...) = {
    Copt[def_c m1(n1,1,...) = {e1^*[c1->c0]}; def_c ...; ek^*]};
    e
    //where f1,...,f does not appear free in Copt and ek is minimal
  ~~~

### Contifiable Subsets
  Given a LetRec term defining a set of functions F = {f1, ..., fn}, the subsets of F of potentially contifiable functions are obtained by the following
    Builidng a tail-call graph of it's functions, identifying the fuctions that call each-other in tail position
    Extracting the strongly- connected components of that graph

  As reminder: strongly-connected functions of F_i in F is then either contifiable together, i.e. Cnt(F_i) or not contifiable at all, non of it's subsets of functions are contifiable

