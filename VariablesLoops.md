# Var, Loops, Syntactical Sugar

## While Loops
  In assembly we will do it by having a unconditional jump to a cond tag
  then evaluate and jump back up to the loop, which runs back into the
  cond tag
  Do while will not have the unconditional jump

## Syntactical Sugar
  Syntax sugar constructs are constructs that can be syntactically 
  translated to other existing constructs. Syntactic sugar does not
  offer additional expressive power to the programmer; only
  convenience.

  x = x +1; => val dummy = x = x + 1;
  y = y + 1; 

  val tmp = if (x > 0)
    x = x - 1
  else
    0; 
  val y = x * 5; 
  y
  =>
  if (x > 0) 
    x = x - 1;
  val y = x * 5; 
  x
  
  
