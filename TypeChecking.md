# Type Checking / Inference - Functions

## Inference Rules
  We created a context mapping \gamma which maps a variable v to a 
  type T. This is a statement that can be True or False determined
  through the *Inference Rules*

  \gamma(Lit(i)): Int means that i is an int
  \gamma(Unary(op,e)): Int by rule [IntUnOp]

  Prim:
    op: [+, -, *, /]
    bop: [==, !=, <=, >=, < ,>]

  [IntOp]:
    \gamma(e1): Int, \gamma(e2): Int -> \gamma(Prim(op,e1,e2)): Int
  [BoolOp]:
    \gamma(e1): Int, \gamma(e2): Int -> \gamma(Prim,(bop,e1,e2)): Int
  [Int]:
    _ -> \gamma(Lit(i)): Int
  [Boolean]:
    _ -> \gamma(Lit(b)): Boolean
  [Unit]:
    _ -> \gamma(()): Unit
  [Let]:
    \gamma(e1): T1, \gamma(x:t1): e2: T2 -> \gamma(Let(x,T1,e1,e2))->T2
  [If]:
    \gamma -> c1 : Bool, \gamma -> e1: T, \gamma -> e2: T
    \gamma -> If(c1, e1, e2): T
  [VarDec]:
    \gamma -> e1: T1, \gamma,x:T1 -> e2: T2
    \gamma -> VarDec(x, T1, e1, e2): T2
  [VarAssign]:
    \gammma(x) = T1, \gamma -> e1: T1
    \gamma -> VarAssign(x, e1): T1
  [While]:
    \gamma -> c1: bool, \gamma -> e1: Unit, \gamma -> e2: T2
    \gamma -> While(c1, e1, e2): T2

## Compilation With Types
  Assembly does not have types, We need to make an implementation on how
  to represent types.
  Bool: 0, 1, but we still use the full register
  Unit: No need to store

  val x = 1 == 4;
    Can be done using branches, one to 0 or one to 1
    set<op> %al //like jump you use sete, setne, setl, etc..
    movbq %al, %rax
    ret

## AST
  FunType(args: List[String,Type], rte: Type) extends Type
  Arg(name: String, atp: Type, pos: Position)
  FunDef(name: String, args: List[Arg], rte: Type, fbody: Tree)
  LetRec(funs: List[Tree], body: Tree)
  App(fun: Tree, args: List[Tree])
  
