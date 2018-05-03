# DataFlow Analysis

## Available Expressions
  The goal is that we can define for every program point the set of available expressions, which is the set of all nontrivial expressions whose value we have already computed by that point.
  Then you can replace those non-trivial expressions at that further point. 

## Formalizing the Analysis
  1. Introduce a variable i_n for the set of expressions avaiable before node n, and a variable o_n for the set of expressions availble after node n
  2. Define equations between those variables
  3. solve those equations

## Solving Equations
  Solved by iteration:
  1. Initialize all sets i1,...,ig, o1,...,og, to the set of all nontrivial expressions in the program
  2. viewing the equations as assignments compute the "new" value of those sets
  3. iterature until fixed point is reached.
  Initialization is done that way because we are interested in finding the larger sets satisfying the equations, the larger a set is, the more information it gives

  To simplify the equations we can first replace all i_k variables by their value to obtain a simpler system, and then solve that system. You end up solving the system of o_i equations.

## DataFlow Analysis
  Available expressions is one example of dataflow analysis. Dataflow analysis is a global analysis framework to approximate various properties of programs such as
  - CSE
  - Dead Code Elimination
  - Constant Propogation
  - Register Allocation
  - etc.

## Analysis Scope
  We will only be working with analysis that works on a single function at a time, while working with the fucntion as a control-flow graph (CFG).

  The nodes of the CFG are the statements of the function and the edge is the flow of control. 

## Analysis #2: Live Variable
  A variable is said to be live at a given point if its value will be used later. While liveness is undecidable, a conservative approximation can be computed by dataflow analysis.

  This  approximation can then be used to allocate registers; a set of variables never live at the same time can share a register

### Intuitions
  A variable is live after a node if it is live before any of its successors. Moreover, a variable is live before node n if it is either read by n, or life after n and is not written by n.
  No variable is live after an exit node. 

### Equations
  Work backwords from the final node to find a solution by iteration. 

### Using Live Variables
  The previous analysis on slide 43 can be used to show that the variables x and y are never live at the same time as z therfore z can be replaced by x or y, thereby removing one assignment

## Analysis #2: Reaching Definitions
  The reaching definitions for a program point are the assignments that may have defined the values of variables at that point. 

  Dataflow analysis can approximate the set of reaching definitions for all program points. These sets can then be used to perform constant propogation for example.

### Intuitioins
  Intuitively a definition reaches the beginning of a node if it reaches the exit of any of its predecessors. 
  - A definition contained in a node n always reaches the end of n iteslf. 
  - A definition reaches the end of a node n if it reaches the beginning of n and is not killed by n itself. (A node n kills a definition iff n is a definition and defines the same variables as the other definition)
  - As a first approximation, we assume no definition reaches the beginning of the entry node.

### Equations
  This can be used to build an equation set which can be solved fairly trivially and can decide where the constant can be propogated. If you allow unititialized variables this can break however, so
  you should mark unitizialized variables by a question mark and initizialize all variables at the beginning of the first node. 

## Analysis #4: Very Busy Expression
  An expression is *very busy* at some program point if it will definitely be evaluated before its value changes

  Dataflow analysis can approximate the set of very busy expressions for all program points. The result of that analysis can then be used to perform code hoisting and the very expression can be 
  performed at the earliest point where it is busy

### Intuitions
  We associate ever node n with a pair of variables (in, on) that give the set of expressions that are very busy when the node is entered or exited respectively.
  
  i_n = gen_VB(n) \cup [(is \cap is2 \cap ... \cap isk) \ kill_VB(n)]

### Equation Sovling
  We are interested in finding the largest sets of very busy expressions so to solve the equations by iteration we initialize all sets with the set of all non-trivial expressions appearing in the 
  program. 

  Again iterate backwards from the end result.

## Taxonomy
  - Forward Analyses: Analyses for which the property of a node depends on those of its predecessors (Available Expresions, Reaching Definitions)
  - Backward Analyses: Analyses for which the property of a node depends upon the successors (Very Busy Expressions, Live Variables)
  - Must Analyses: Analyses for which a property must be true for all successors or predecessors for a node to be true in that node (very busy expressions)
  - May Analyses: Analyses for which a property msut be true in at least one successor or predecessor of a node to be true in that node (reaching definitions, live variables)

## Speeding Up Dataflow Analyses
  - An algorithm based upon a work-list can avoid useless computation
  - Equations can be ordered in order to propogate information faster
  - Analyses can be performed on smaller control-flow graphs, where the nodes are basic blocks instead of invidivudual instructions
  - Bit-Vectors can be used to represent sets

## Running Example
### Work-List Algorithm
  Computing the fixed point by simple iteration as we did works, but is 
  wasteful as the information for all nodes is recomputed at each iteration.

  It is possible to do better by remembering every variable v, the set dep(v)
  of the variables who depend on the value of v

  Then, whenever the value for some v changes, we only recompute the value of
  the variables that belong to dep(v)

### Node Ordering
  Using the work-list, "only" 11 computations were required in the example 
  instead of 18 in the original.

  This could be improved if the elements of the work-list were ordered in 
  reverse order. The goal of node ordering is to order the elements of the 
  work-list in such a way that the solution is computed as fast as possible.

### Reverse (Post-Order)
  For backward analyses, ordering the variables in the worklist according to a
  post order-traversal of the CFG nodes speeds up convergence. For forward
  analyses, reverse post order has the same characteristic

### Basic Blocks
  Until now CFG nodes were single instructions, in practice *basic blocks* tend
  to be used as nodes to reduce the size of the CFG. When dataflow analysis is
  performed on a CFG composed of basic blocks, a pair of variables is attached
  to every block not every instruction.

  Once the solution is known for basic blocks, computing the solution for 
  instructions is easy. 

### Bit Vectors
  All Dataflow analyses can be seen to work on sets of values. If these sets
  are dense a good way to represent them is to use bit vectors. A bit is
  associated to every possible element of the set and its value is 1 iff
  the corresponding element belongs to the set.

  On such a representation, set union is bitwise-or, set intersection is 
  bitwise-and, and set difference is bitwise-and composed with bitwise-negation

## Summary
  Dataflow Analysis is a framework that can be used to approximate various 
  properties about programs.

  Can approximate: 
    - Liveness
    - Available Expressions
    - Very Busy Expressions
    - Reaching Definitions
  Used for:
    - Dead-Code Elimination
    - Constant Propogation
    - etc.
