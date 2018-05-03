# Register Allocation

## Memory Hiearchy
  CPU holds the registers, communicates with the cache which is MBs large, 
  which communicates with the Main Memory which is GB-TBs large which can
  communicate with the Disk,Cloud which is >= TB.

  `CPU <-> Caches <-> Main Memory <-> Disk, Cloud`

## Register Allocation
  The problem of *register allocation* consists of rewriting a program that 
  makes use of an unbounded number of local variables (*virtual registers*)
  into one that only makes use of machine registers.

  You must "spill" stored variables into memory if there are not enough
  registers to store all of the variables.

  Register Allocation is one of the very last phases of the compilation process
  with only instruction scheduling coming after it. It is performed on the
  intermmediate language closes to machine code. 

## Setting the Scene
  Register allocation can only be performed on a language with the following
    - Apart from n machine registers R0, ..., Rn-1, an unbounded number of
      virtual registers v0, v1,... are available before register allocation
    - Machine Registers that play a special role, like the link register, are identified with a non-numerical index

## Techniques
  - Graph Coloring: Slow but produces very good results
  - Linear Scan: Fast but produces worse results
  Because of slowness graph coloring is used in batch compilers, while linear
  scans are used in JIT Compilers

  - Batch Compiler: Compiles all functions at once (Ahead of Time Compiler)
  - JIT: (Just-In-Time) compiler compiles code at run-time, i.e as the program is running. Therefore the cost of compilation is part of the execution time so it should be minimized. 

  Both techniques are global in that they allocate registers for functions at
  a time. 

### Technique #1: Allocation by Graph Coloring
  - The interference graph is built. One node per register, real or virtual, and two nodes are connected by an edge iff their registers are simultaneoulsy live
  - The interference graph is colored with K colors (the number of available registers) so that all nodes have a different color than their neighbors
  
  Problems:
    - For an arbitary graph, the coloring problem is NP-Complete
    - A K-Coloring might not exist

#### Coloring By Simplification
  This is a heuristic technique to try to color a graph G with K Colors.

  If the graph G has at least one node n with less than K neighbors, n is 
  removed from G, and that simplified graph is recursively colored. Once this
  is done, n is colored with any color not used by its neighbors

  There is always at least one color available for n, because its neighbors
  use at most K-1 colors. If the graph does not contain a node with less than
  K neighbors K-Coloring might not be feasible

##### Spilling
  During Simplification it is possible to reach a point where all nodes have
  at least K neighbors. When this occurs a node n must be chose to be spilled.
  As a first approximation, we assume that the spilled value does not interfere
  with any other value, remove it's node from the graph, and color the graph
  as per usual. After it's been colored it is possible that the neighbors of n
  do not use all the possible colors, in this case n is not spilled, otherwise
  we commit the spilling.
##### Spill Costs
  The node to spill could be chosen at random but it is better to favor values
  not used frequently or values that interfere with others.

  Spill Cost formula: cost(n) = (rw0(n) + 10 rw1(n) + ... + 10^k rwk(n)) / degree(n)

  where rwi(n) is the number of times the value of n is read or written in a 
  loop depth i and degree(n) is the number of edges adjacent to n in the 
  interference graph.

##### Spilling of Pre-Colored Nodes
  As we have seen the interference graph contains nodes corresponding to the 
  registers of the machine. These are pre-colored, because the color of each of
  them is given by the machine register it represents. This can simplify the
  coloring process as the cannot be spilled. 

##### Consequences of Spilling
  Once a node is spilled the original code must be rewritten.
  - Before the spilled value is read, code my be inserted to fetch it from mem
  - Just after the spilled value is written, code must be inserted to write it to mem

  Since spilling code introduces new virtual registers, the whole register 
  allocation proccess must be restarted from the beginning. (Usually one or 
  two iterations)

#### Coalescing
  Given an instruction of the form `v1 = v2` and provided that v1 and v2 do not
  interfere, it is possible to replace all instances of v1 and v2 by instances
  of a new virtual register v1&2. Once this has been done, the move instruction
  becomes useless and can be removed. This is known as coalescing as v1 and v2
  can become a single node.

#### Coalescing Issue
  Coalescing is not always a good idea. The new node can have a higher degree
  than the original nodes which might make the graph impossible to color with K
  colors and force spilling.

#### Coalescing Heuristics
  1. Briggs: Coalesce nodes n1 and n2 to n1&2 iff n1&2 has less than K neighbors of significant degree (degree greater or equal to K)
  2. George: coalesce nodes n1 and n2 iff all neighbors of n1 either already interfere with n2 or are of insignificant degree.

  Both heuristics are safe, in that they will not turn a K-color graph into a 
  non-k-colorable one. But they are also conservative in that they might 
  prevent a coalescing that would be safe.

### Summary Iterated Register Coalescing
  To get the best results, the phases of simplifaction and coalescing should
  be interleaved. This technique is known as iterated register coalescing (IRC)

  1. Interference graph nodes are partitioned into two classes: move-related and not
  2. Simplification is done on not move-related nodes (move-related can be coalesced)
  3. Conservative coalescing is performed.
  4. When neither simplification nor coalescing can proceed some move-related nodes are marked as frozen or non-move-related
  5. Restart on 2. 

### Assignment Constraints
  Until now we have assumed that a virtual register can be assigned to any 
  physical register so long as its free. 

  This is not always the case due to architectural characteristics.
  - Some architectures divide the registers into several classes 
  - Some instructions require some of their arguments - or their result - to be in specific registers
  - calling conventions require function arguments and results to be in specific registers
  
  A realistic register allocator has to satisfy these

#### Register Classes
  Most architectures seperate registers into classes for floating-point and
  other types of values. These can be accounted for in coloring based 
  allocators by making nodes of all pre-colored nodes of a certain class 
  interfere with a variable of not its class. 

#### Calling Conventions
  Many calling conventions pass arguments in registers. 

  At the beginning of all functions, move instructions have to be inserted to
  copy the arguments to new virtual registers.

  Similarly before any function call, move instructions have to be inserted to
  load the arguments in the appropriate registers.

  Whenever possible, these move instructions will be removed by coalescing. 

##### Caller/Callee-Saved Registers
  Calling Conventions can be distinguished into two kinds:
  - Caller-Saved: saved by the caller before a call and restored after exit
  - Callee-Saved: Saved by the callee at function entry and restored before exit
  Ideally all virtual registers that have to survive at least one call should 
  be assigned to callee-saved registers, while others should be caller-saved.

  The contents of caller-saved registers do not survive a function call. To 
  model this, edges are added tot he interference graph between all virtual
  registers that are live across at least one call and physical caller-saved
  registers

  These edges ensure virtual registers that are live across at least one call
  will not be assigned to caller-saved registers and will be spilled or 
  allocated to callee-saved registers

##### Saving Callee-Saved Registers
  Since these must be preserved by all functions, this can be achieved by 
  copying them to fresh registers at function entry and restoring them before
  exit.

  If the register pressure is love, then the two can be coalesced and the 
  move instructions can deleted, otherwise the temp register is spilled and
  the original register can be utilized in the function body

### Technique #2: Linear Scan
  Very Simple:
  - The program is linearlized, a linear sequence of instructions not as a graph
  - A unique live range is computed for every variable 
  - Registers are allocated by iterating over the intervals sorted by increasing starting point. Each time an interval starts, the next free register is allocated to it, and once it ends the register is freed
  - If no register is available the active range ending last is chose to have its variable spilled.

#### Linear Scan Improvements
  The Basic linear scan algorithm is very simple but procudes reasonable code.
  It can be and has been improved in many ways:
  - Liveness information about virtual registers can be described using a sequence of disjoint intervals instead of a single one
  - Virtual registers can be spilled for only a part of their whole life
  - More sophisiticated heuristics can be used to select the register to spill
  - etc.

