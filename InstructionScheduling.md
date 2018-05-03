# Instruction Scheduling

## Instruction Ordering
  When a compiler emits instructions it imposes a total order on them, however
  that order is not the only valid one. It can be changed without modifying
  the program's behaviour. If two instructions i1,2 appear sequentially in 
  that order and are independent you can swap them.

## Instruction Scheduling
  Among all the valid premutations of the instructions, some are more 
  desirable. Once can be faster on some machine because of architectural 
  constraints. The goal is the optimize these on some metric like speed.

## Pipeline Stalls
  Modern architectures can usually issue at least one instruction per clock
  cycle. However an instruction can only be executed if the data it needs is
  ready, otherwise it stalls for one or more cycles. Stalls can appear 
  because some instruction, e.g. division, require more cycles to complete
  or data must be fetched from memory. 

## Scheduling Example:
  - Memory load or store: 3 Cycles
  - Multiplication: 2 Cycles
  - Addition: 1 Cycle

## Data Dependencies
  - True Dependence: i2 reads a value written by i1 (RAW)
  - Antidependence: i2 writes a value read by i1 (WAR)
  - Antidependence - i2 writes a value written by i1 (WAW)

### Antidependences
  Antidependencies are not real in the sense that they don't arise from the
  flow of data, but are due to a single location being used to store 
  different values.

  Most of the time these can be removed by renaming locations/registers

## Computing Dependences
  Instructions that access memory are harder to handle than those who access
  only registers. It is not known in general whether two instructions refer to
  the same memory location. Conservative approximations have to be used then

## Dependence Graph
  The *dependence graph* is a directed graph representing dependences among
  instructions. Its nodes are the instructions to schedule and there is an
  edge from node n1 to n2 iff the instruction n2 depends on n1. Any 
  topological sort of the nodes represents a valid way to schedule.

## Difficulty of Scheduling
  Optimal scheduling is NP-Complete, but this implies we can use heuristics
  to find a good but not guranteed optimal solution to the problem. List
  Scheduling is a technique to schedule the instructions of a single basic 
  block. It's basic idea is to simulate the execution of the instructions and
  try to schedule instructions only when all their operands can be used 
  without stalling the pipeline. 

## List Scheduling Algorithm
  Maintain Two Lists:
    - Ready: a list of instructions that could be scheduled without stall
    - Active: List of instructions that are being executed.
  The idea is that at each step the highest-priority ready instruction is 
  scheduled and moved to active where it stays for a time delay equal to its
  delay. 

  Before scheduling is performed renaming is done to remove all antidependence
  that can be removed.

### Prioritizing Instructions
  When prioritizing nodes in the ready list there are several schemes
  - The length of the longest latency-weighted path from it to a root of the dependence graph
  - The number of its immediate successors
  - The number of its decendents
  - its latency
  - etc. 

## Scheduling Conflicts
  It is hard to decide whether schedling should be done before or after 
  register allocation. If allocation is done first, it can introduce 
  antidependences when reusing registers. If scheduling is done first 
  acllocation can introduce spilling code, destroying the schedule.

  Solution: schedulue, allocate, schedule
