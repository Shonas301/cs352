# Operator Precedence and Tokenization

## A Better Grammar
  If you define a grammar with explicit operator precedence you can solve the
  issue of arithmetic operations being done out of order. 

## Generic Operator Precedence
  You can define grammars with less explicitness and be allowed to update it
  with new operators if you define a precedence level for each operator. You
  then parse and compute all of the high precedence level operators first and
  then the low level ones. 

  You need to also keep track of the associativity of each operator (left vs
  right)

## Error Handling
  A parser should also be able to reject invalid programs as well. Ideally it
  should be able to report more information as well such as line number or
  the error encountered, and possible try and recover and continue parsing,
  testing integer overflow. 
