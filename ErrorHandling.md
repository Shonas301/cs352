# Error Handling - Semantics - Branches

So far we have focused on learning how to accept _valid_ programs. One other aspect of the parser is to reject invalid programs as well

What can be added to improve the error handling? 
  Repeat more Information such as line number, give hints
  Try to recover and continue parsing
  Test integer overflow

Types of errors:
  Syntax: val 1 = 2;
  Semantic: val x = z; undefined identifier 'z'
  Synatx: unexpected character '&'
  Semantic: undefinied operator ++

Handling:
  Parser handles syntax errors, semantic analyzer handles the semantic errors
  After these two phases any problem is a compiler bug

How our token position works:
  Each token is assigned a position which is line start, stop, column start, column end

Solutions to Syntax error:
  Report and exit
  Try to recover and continue
    Panic Mode
    Phase-Level Recovery
    Context-Specific Look-Ahead
    Burke-Fisher
      Attempts to continue parsing by backup up to k parse
      toekns before the rror point and attempting to sub
      in all possible tokens to keep finding future errors.
