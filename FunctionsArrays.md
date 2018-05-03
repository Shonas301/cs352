# Functions - Arrays

## Function - Type Checking
  A function type is well-formed if all of its argument types and its
  return types are well-fromed
  a function type tp conforms to type pt if all of the following hold
    1. pt is a function type or unknown type
    2. pt has the same number of arugments as tp
    3. the type of pt argument #n conforms to tp argument #n
    4. the return type of tp conforms to the return type of pt

  EX:
    (Int, Boolean) => Int conforms to ??? (Int)
    Int => Int conforms to Int => Int
    Int => Into does not conform to Boolean
    ??? => Int conforms to Int => Int 
    Int => Int does not conform to ??? => Int (rule #3)
    Int => Boolean does not conform to Int => Int (rule # 4)
    ??? => Boolean conforms to Int => (???) (result Int => Bool)

  [FunDef]:
    \gamma,a1:t1,...an:Tn -> fb: T
    \gamma -> FunDef(f, a1:T1...an:TN, T, fb) : (T1...Tn) => T
  [App]:
    \gamma -> f: (T1,...TN) => T, \gamma -> a1: T1, etc
    \gamma -> App(f, a1...an): T
  [LetRec]:
    \gamma f1:FT1,...fn:FTN -> f1: FT1, ... n, \gamma f1-fn:T1,N -> b:T
    \gamma -> LetRec(f1...fn,b): T

## Arrays
  Arrays are well form if tp is, that is
    pt is an array type and
    inner type tp conforms to pt inner type
  An array type tp conforms to a pt if:
    pt is a funciton type with one argument
    the function argument's type conforms to IntType
    The inner type of tp conforms to the return type of pt
  Array[T] conforms to Int => T
  
  [ArrayDec]:
    \g -> size: Int
    \g -> ArrayDec(size,T): array[T]
  [ArrayGet]:
    \g -> arr: Array[T] \g -> i : Int
    \g -> Prim("block-get", List(arr, i)): T
  [ArraySet]:
    \g -> arr: Array[T] \g -> i : Int \g -> v: T
    \g -> Prim("block-set", List(arr,i,v)): Unit
