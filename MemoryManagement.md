# Memory Management

## Memory Management
  The memory of a computer is finite. Typical programs use a lot over a life
  but not all of it at the same time. The goal is the use that finiteness as
  efficiently as possible. In general, you dynamically allocate memory from 
  either the stack or heap. Since the management of the stack is trivial 
  this term usually designates heap management.

## The Memory Manager
  Its job consisits of maintaining the set of free memory blocks and to use
  them to fulfill allocation requests from the program.

  Memory deallocation:
  - Explicit: When the program asks for a block to be freed
  - Implicit: Automatically try to free unused blocks when there is not enough memory to satisfy allocation requests

### Explicit Deallocation:
  1. Memory can be freed to early which leads to dangling pointers which can cause data corruption, crashes, or security issues
  2. Memory can be freed to late or never which causes leaks

  Due to these problems most modern languages are designed to provide 
  implicit deallocation or automatic memory management, better known as
  garbage collection. Though garbage collection refers to a specific kind of 
  automatic memory management. 

### Implicit Deallocation
  If a block of memory is reachable, then it will be used again in the future
  and cannot be freed. Only once blocks can not be reached can they be freed.
  Since this assumption is conservative it is possible to have memor leaks. 
  This happens whenever a reference to a memory block is kept but the block is
  not accessed anymore. This does eliminate the problem of dangling pointers
  however.

### Garbage Collection
  This is a technique used to reclaim objects that are not reachable anymore.
  1. Reference Counting,
  2. Mark & Sweep Collection
  3. Copying Garbage Collection

#### Reachable Objects
  The set of reachable objects are objects immediatley accessible from global
  variables, the stack / registers, whcih are called the root set, and the 
  objects reachable from other reachable objects by following pointers.
  
  These form the *reachability graph*

##### Imprecision
  To compute the reachability graph at run time it must possible to identify 
  unambigiously pointers in registers, stack, and heap. If this is not 
  possible you must approximate the graph. For example when it is used to free
  memory it is only safe to over-approximate it. 

### Memmory Manager Data Structures - Free List
  If you maintain a collection of free blocks called the free list (even 
  when it is not actually a list) you can keep track of the heap trivially. 

#### Block Header
  The block header contains information stored just before the actual info 
  such as size

#### BiBoP
  BiBoP (Big Bag of Pages) are used to decrease the overhead of headers.
  This groups objects of idential size into contiguous areas of memory called
  pages. All pages have the same power of two size and start at an address
  which is a multiple of the size. the size of all objects on a page is stored
  at the page's beginning and can be retrieved by masking the power of two
  least significant bits of an objects address.

#### Fragmentation
  This is used to designate two different but similar problems:
  - External Fragmentation: Fragmentation of free memory in many small blocks
  - Internal Fragmentation: Refers to the waste of memory due to the use of a free block larger than required to satisfy an allocation request. 

##### External Fragmentation
  This can cause some requests to be fulfilled (smaller memory blocks) but 
  not larger memory blocks.

#### Internal Fragmentation
  For reasons such as alignment constraints the memory manager somtimes 
  allocates slightly more memory than requested. This results in small wasted
  memory scattered across the heap. 

### GC Technique 1: Reference Counting
  Every object carries a count of the number of pointers that reference it. 
  When this count reaches zero, the object is unreachable and can be 
  deallocated. This requires collaboration between the compiler and/or 
  programmer to make sure that refrence counts are properly maintained

#### Pros and Cons
  This is easy to implement even as a library. It impacts space consumption 
  and speed of execution as every object must contain a counter and every 
  pointer write must update it. 

##### Cyclic Structures
  The reference count of objects in a cycle never reach zero, even when they
  are unreachable. 

##### Uses of Reference Counting
  Due to the problem with cyclic structures it is seldom used. It is still 
  interesting for systems that do not allow cyclic structures to be created
  like hard links in Unix File systems. It can be used in combination with
  Mark and Sweep, the latter being used infrequently to collect cyclic 
  structures.

### GC Technique #2: Mark and Sweep
  1. In the marking phase, the reachability graph is traversed and reachable objects are marked.
  2. In the sweeping phase all allocated objects are examined, and unmarked ones are freed.

  GC is triggered by a lack of memory and must complete before resuming. This
  is necessary to ensure the reachability graph is not modified by the program
  as GC traverses it. 

#### Marking Objects
  Since only one bit is required for the mark it is possible to store it in 
  the block header with the size. If all blocks have an even size then the LSB
  of the block size can be used for marking. It is possible to use external
  bit maps stored in a memory area that is private to the GC to store mark 
  bits

#### Free List
  Free Blocks are not contiguous in memory therefore you need to store it in a
  free list. This can simply be a linked list. Since the free blocks are by 
  definition not used by the program the links can be stored in the blocks 
  themselves.

#### Allocation Policy
  When a block of memory is requested, there are in general many free blocks
  big enough to satisfy it. 

  - First Fit: uses the first suitable block
  - Best Fit: uses the smallest block to satisfy

#### Splitting and Coalescing
  When the memory manager has found a big enough block it is possible for that
  block to be bigger than the size requested. It must then be split into two
  parts. One is returned to the client the other is reinserted to the free 
  list. 

  During deallocation if the freed block is next to a bigger freed block you
  should coallesce the two blocks into one larger block to avoid fragmentation
  of the list.

#### Reachability Graph Traversal 
  The mark phase requires a depth-first traversal of the graph which is 
  usually recursive. These use stack space since the depth of the graph is not
  bounded this can cause an overflow. 

#### Sweeping Objects
  Once the mark phase is terminated all allocated but unmarked objects can be
  freed. This is the job of the sweep phase which traverses the heap 
  sequentially looking for unmarked objects and adding them to the free list.
  Unreachable objects cannot become reachable again so it is possible to sweep
  objects on demand to only fulfill the current memory need. (Lazy Sweep)

#### Conservative MS GC
  One that is able to collect memory without unambiguously identifying 
  pointers at run time. This is possible because an approximation for the
  reachability graph is sufficient to collect some garbage, as long as the 
  approx. includes the actual reachability graph.

  It assumes that everything that looks like a pointer to a object is a 
  pointer to an object. This assumption is conservative in that it can lead
  to the retention of dead objects but it cannot lead to the freeing of live
  objects. It is important that the GC uses as many hints as possible to avoid
  considering non-pointers as pointers as this can lead to memory being
  saved for no reason. 

##### Pointer Identificatioin
  - Many architectures require pointers to be aligned in mem on 2 or 4 byte boundaries. Unaligned pointers can be tossed aside
  - If a object is reachable then there exists at least one pointer to its beginning in some architecture so some potentially interior pointers can be ignored. 

### GC Technique #3: Copying GC
  The idea of copying GC is to split the heap in two semi-spaces: the 
  from-space and the to-space. Memory is allocated in from-space while the
  two space is left empty. When from-space is full all reachable objects in 
  from-space are copied to to-space and pointers to them are updated.
  Finally the role of the two spaces is exchanged and the program resumed.

#### Allocation In A Copying GC
  The memory is allocated linereally in from-space. There is no free-list and
  no search to performed to find a free block. All that is required is a 
  pointer to the border between allocated and free area of fromspace.
  This is very fast, as fast as stack, allocation

#### Forwarding Pointers
  Before copying an object, a check must be made to see whether it has already
  been copied. If this is the case it must not be copied again. Rather the
  already copied version must be used. This can be down by storing a fowarding
  pointer in the object in from-space after it has been copied. 

#### Cheney's Copying GC
  The algorithm already presented does a depth-first traversal of the 
  reachable graph and uses recursion. Cheney's algorithm does a bfs of the  
  graph, requiring only one pointer as additional state.

  In any bfs, one has to remember the set of nodes that has been visited but
  whose children have not been. The basic idea is to use to-space to store
  this set of nodes, which can be represented using a single pointer scan.
  The pointer partitions to-space in two parts: the nodes whose children have
  been visited, and those whose children have not been visited. 

### Copy vs Mark & Sweep
  Pros of Copying:
    - No external Fragmentation
    - Very fast allocation
    - no traversal of dead objects
  Cons: 
    - Uses twice as much virtual memory
    - Requires precise identification of pointers
    - Copying can be expensive

### Mostly-Copying GC
  A copying GC can be used in situations where not all pointers can be
  identifed. This is the idea behind Bartlett's mostly-copying GC.

  These are partitioned into two classes:
    - Those for which some pointers are ambiguous, because they appear in the
        stack or registers
    - Those for which all pointers are known umabiguously

  Objects of the first case are left where they are (pinned) while the others
  are copied as usual. Objects cannot be pinned if the from and to spaces are
  two seperate areas of memory as fromspace must be completely empty after
  GC. Therefore GC organizes memory in pages of fixed size with the space to
  which they belong. Then during gc the pinned objects are left on their page
  whose tag is updated to ove them to to-space and all other objects are
  compacted as usual. 

### GC Technique #4: Generational GC
  The large majority of objects die young, so you should try to partition
  objects into generations based on their age and collect the young
  generations more than the older ones. This should improve the amount of
  memory collecty per objects visited. In a copying GC this also avoids
  repeatedly copying long-lived objects. In this GC objects are partitioned
  into a given number of generations. The younger a generation is the smaller
  amount of memory is resrved to it. All objects are initially allocated to
  the youngest generation. When it is full that generation is scanned for
  surviving memory blocks and promoted to the next generation based on a
  promotion policy. When a older generation itself is full a major collection
  is performed upon the entire from space. 

### Hybrid Heap Organization
  Instead of managing all generations using copying, it is also possible to
  manage the oldest using M&s.

#### Promotion Policies
  The simplest promotion policy is that all surivors are advanced, but this
  can proite very young objects. It is simple though as age does not to be
  recorded. To avoid promoting very young objects it is sufficient to wait
  until they survive a second collection before advancing them. 

#### Minor Collection Roots
  The roots used for a minor collection must also include all pointers from
  older generations to younger ones, otherwise reachable objects can get
  collected. 

#### Inter-Generational Pointers
  These can be handeled in two different was:
    1. By scanning - without collecting - older generations during a minor
       collection.
    2. By detecting pointer writes using a write barrier - either in software
       or through hardware -  and remembering inter-generational pointers. 

#### Remembered Set
  A set which contains all old objects pointing to young objects.
  The write barrier maintains this set by adding to it iff:
    - The object into which the pointer is stored is not yet in the set and
    - the pointer is stored in an old object and points to a young one

#### Card Making
  Another technique to detect inter-generational pointers. Memory is divided
  into small, fixed sized areas called cards. A card table remembers whether
  it potentially contains the pointers. On each write the card is marked in
  the table and marked cards are scanned for inter-generational pointers
  during collection. 

#### Nepotism
  Since old generations are not collected as often, it is possible for dead
  old objects to prevent the collection of dead objects. 

#### Pros and Cons
  Generational GC tends to reduce GC pause times since only the youngest are
  collected. In copying GCs the use of generations also avoids copying
  long-lived objects over and over again. The only problem is the cost of
  maintaining the remembered set and nepotism. 

### Other Kinds: Incremental / Concurrent GC
  An incremental GC can collect memory in small, incremental steps, thereby
  reducing the length of GC pauses. A very important characteristic for
  interactive apps. The must deal with modifications to reachability made by
  the program, called the mutator, during computation. This is usually
  achived by using a write barrier that ensures the graph is a valid
  approximation of the real one. 

### Other Kinds: Parallel GC
  Some parts of garbabe collection can be sped up by performing them in
  parallel on several processors. In M&S GC marking can easily be done in
  parallel. 

### Other Kinds: Virtual-Memory-Aware GC
  Bookmarking GC is an example that avoids the problem of poor performance
  when little physical memory is available. Its basic idea is to bookmark
  memory-resident objects referenced by evicted objects. These objects are
  then considered reachable and GC is performed without looking and therefore
  avoids loading the objects.

### Additional GC Features: Finalizers
  These are functions that are called when an object is about to be
  collected. The are used to free the external resources associated with the
  object. There is no guarantee when finalizers are invoked so the resource
  should not be scarce.

#### Finalizer Issues:
  - What should be done if a finalizer makes the object reachable again?
  - How do the interact with concurrency - which thread?
  - How can they be implemented efficiently in a copying GC?

### Flavors of References
  When GC encounters a ref, it treats it as a object as reachable and
  survives the colletion. It might be useful to have weaker reference kinds
  which can refer to an object without stopping colletion.

#### Weak References
  A reference that does not prevent an object from collection. During  GC a
  ref which is only weakly reachable is collected and all weak references
  referencing it are cleared. These are useful for caches, "canonicalizing",
  mapping, etc. 
