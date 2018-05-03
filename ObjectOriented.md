# Object-Orientated Languages

## Object-Orientated Languages
  In a class-based language all values are objects belonging to a class.
  Objects encasulate both state, stored in fields, and behaviour, ie. the
  methods of the class. Two of the most important features are inheritance
  and polymoprhism. 

## Inheritance
  A code reuse mechanism in which a class inherits all the fields and methods
  of its parent class (superclass). This is just code copying but is usually
  implemented in such a way as to prevent code explosion. 

## Subtyping and Polymorphism
  In typed OO languages, classes usually define types. These are related
  through subtyping (conformance) relations. A type T1 that is a subtype of
  T2 has atleast the capabilities of T2. When T1 =[ T2 then T1 can be used
  whenever a type of T2 is expected. This is called inclusion polymorphism. 

### Subtyping != Inheritance
  These are not the same thing, but many OO languages tie them together by
  stating that every class defines a type and every type of class is a
  subtype of it's superclass. This is a design choice not an obligation.
  Several langauges allow you to seperate this out. Java interfaces make it
  possible to define subtypes but that are not subclasses. C++ has private
  inheritance that allowes the definition of subclasses that are not
  subtypes. 

## "Duck Typing"
  In OO like ruby inheritance is used only to reuse code, no notion of static
  type even exists. Whether an object can be used in a context is dependent
  soley upon methods implemented and not on class in the inheritance
  hiearchy.

## Polymorphism Challenges
  The following are caused by inclusion polymorphism:
  - Object Layout - arranging object fields in memory
  - Method Dispatch - Finding which concrete implementation to call
  - Membership Tests - Is an object an instance of some type?

### Object Layout
  Inclusion Polymorphism forces the layout of different object types to be
  compatible in some way. A field defined in a type T should appear at the
  same offset in all subtypes of T. 

#### Case 1: Single Inheritance
  Here the object layout problem is easily solved. The fields of a class are
  laid out sequentially, starting with those of the super class. 

#### Case 2: Multiple Inheritance
  Here it is much more difficult, if you use a unidirectional layout then
  there is wased space. 
  
##### Bidirectional Layouts
  However with Bidirectional layout you can avoid
  wasted space by using negative offsets. There does not always exist such a
  layout, and finding the optimal one is shown to be NP-Complete. In addition
  its computation requires the entire hiearchy to be known and computed at
  link time. 

##### Accessor Methods
  Only allow modification to fields through accessor methods. The fields can
  then be laid out freely. Whenever the offset of a field is not the same as
  its parent the accessor method is redefined. This reduces the method
  dispatch problem as well.

##### Other Techniques
  Bidrectional Layout wastes space, but field access is very fast. Acccessors
  don't waste space but take more cycles to complete. Two-Dimensional
  bidirectional layout slows down access slightly but it never wastes space,
  however it also requires the hiearchy to be known. 

##### Object Layout Summary
  The object layout problem can be solved trivially in single inheritance but
  multiple inheritance generates more problems in which you must make
  tradeoffs between speed and space. 

### Method Dispatch:
  Gien an object and a method, identify which piece of code to execute. IP
  makes the problem harder since it prevents static evaluation at compilation
  time. Efficient Dynamic methods have been devised. 

#### Case 1: Single Subtyping
  Method pointers are stored sequentially, starting with those of the
  superclass, in a virtual methods table shared by all instances of the
  class. This ensures that the implementation for a given method is always at
  the same postition in the VM and can be extracted. 

##### Dispatching with VMTs
  1. The VMT of the selector is extracted
  2. The code pointer for the invoked method is extracted from the VMT
  3. The method implementation is invoked

  Each of these steps requires a single, expensive, instruction

##### VMTs Pros and Cons
  This provides very efficient dispatching and does not use much mem
  unfortunately they do not work for dynamic languages or in the presence of
  any kind of "multiple subtyping" - e.g. multiple interface inheritance in
  Java. 

#### Case 2: Multiple Subtyping
  A trivial way to solve the problem is to use a global dispatching matrix 
  containing the code pointers and indexed by classes and methods. This makes
  dispatching very fast however it can occupy so much memory it is never used
  as is in practice. 
  
##### Dispatching Matrix
  You can compress the matrix but you trade dispatching
  efficiency for reduced memory usage by taking advatage of:
  1. The matrix is sparse since any given class implements only a limited
     subset of methods
  2. It contains a lot of redundancy since many methods are inherited

##### Column Displacement
  A technique for null elimination. If you transform the matrix into a linear
  array by shifting either its columns or its rows. Many holes of the matrix
  can be filled int he proccess if you choose the amount of columns (rows)
  are shifted. In practice column displacement is better than row
  displacement. 
  
##### Dispatcing with CD
  1. Add the offset of the method being invoked to the offset of the class of
     the receptor to extract the pointer
  2. Invoking the method referenced by that point. 

##### Duplicate Elimination
  Apart from being sparse much of the information is repeated, so we can
  eliminate even more. The idea of duplication elimination is to try to share
  as much information as possible. 

##### Compact Dispatch Tables
  Split the dispatch matrix into small sub-matrices called chunks. Each chuck
  will tend to have duplicate rows which can be shared by representing each
  chunk as an array of pointers to rows

##### Dispatching with CDTs
  1. Use the offset of the class of the reciever and the offset and chunk of
     the method being invoked to extract the code pointer
  2. Invoking the method referenced by the pointer
  Because of the additional indirection due to the chunk array, this is
  slightly slower however it tends to compress the matrix better in practice. 

##### Hybrid Techniques
  Java Implementations use VMTs to dispatch when the type of the selector is
  a class type, and more sophisticated and slower techniques when it is an
  interface. The JVM even has different instructions for the two kinds of
  dispatch. Invokevirtual and invokeinterface. 

##### Inline Caching
  Even when efficient dispatching structures are used, the cost of performing
  a dispatch on every method call can become important. In practice many
  potentially polymorphic calls are infact monomorphic so you can record at
  every call site the target of the latest dispatch and assume the next one
  will be the same. This is inline caching. 

  At first, all method calls are compiled to call a dispatching function.
  When this function is invoked, it computes the target of the call, and then
  patches the original call to refer to the computed target. All methods have
  to handle the potential mispredictions of this technique and invoke the
  dispatching function when they hapen. 

##### Polymorphic Inline Caching
  While inline caching replaces the call to the dispatch function by a call
  to the lastest method, PIC replaces it with a call to a specialized
  dispatch routine, generated JIT. That routine handles only a subset of the
  possible reciever types - those encountered previously at that call site. 

##### PIC Reciever Type Test
  The generated dispatcher should be able to test very quickly whether an
  object is of a given type by class tagging. This test does not check
  whether the type of the reciever is a subtype of a given type, but instead
  for equality. This means several entries can actually call the same method
  if that's inherited. 

##### PIC Optimizations
  PIC methods can be inlined into it, and the tests can be arranged so that
  the ones corresponding to the most frequent appear first

##### Method Dispatch Summary
  Method dispatch problem can be solved by virtual method tables while in the
  precense of multyple subtyping a compressed global distpatch matrix is
  used. Inline caching and its polymorphic variant can dramatically reduce
  the cost of dispatching. 

### Membership Test
  Checking if a member is of a certain type.

#### Case 1: Single Subtype
  Can be solved in two ways
  - Relative numbering,
  - Cohen's Encoding

##### Relative Numbering
  Number the types in the hiearchy during a preorder traversal. The numbers
  attributated to all descendents of a given type form a continuous interval.
  Testing can therefore be perfomred very efficiently by checking whether the
  number belongs to a certain interval. 

##### Cohen's Encoding
  Partition the types according to their level in the hiearchy. The level of
  a type T is definied as the length from the root to T (depth). Then all
  types are numbered so that no two types at a level have the same number.
  Finally a display is attached to all types T, mapping all levels L smaller
  or equal to T to the number of the ancestor of T at level L. 

##### Global Identifiers
  If globally-unique identifiers are used for the types instead of
  level-unique ones. Then the display bounds check can be removed if the
  displays are stored consecutively in memory with the longest one at the
  end. 

##### Comparison
  Cohen's requires more complication and memore it has the advantage of being
  incremental so it is easier to ass new types to the hiearchy without
  recomputation. This is important when new types can be added at run time,
  e.g. Java

#### Case 2: Multiple Subtyping - Membership Test
  In this setting neither of the above can be directly applied. 
  - Range compression
  - Packed Encoding
  - PQ Encoding

##### Range Compression
  A generilzation of relative numbering. Uniquiely number all tpes by
  spanning forests chosen judiciously, then each type caries the number of
  all its decendents as a list of disjoint intervals. 

##### Packed Encoding
  A generilzation of Cohen's. Partition types into slices, as few as
  possible, so that all ancestors of all types are in different slices. Types
  are then numbered uniquely. A display is then attached to every type T. 

##### PQ Encoding
  Partition types into slices, then number the types such that for all types
  T in a slice S, all descendenants of T are numbered consecutively in S

##### Membership Test Summary
  In a single subtyping context, two simple solutions to the membership test
  exist: relative numbering and cohen's. Only the latter is incremental.

  Generalizations of these techniques exist in the multiple subtype context:
  range compression, packed, and PQ encoding. Thsee techniques enable the
  test to be solved efficiently but the building and supporting structure is
  complicated. None of these are Incremental. 
