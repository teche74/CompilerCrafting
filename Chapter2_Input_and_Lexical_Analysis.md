------------------------------------------------------------------------------------------------------------------------
# **INPUT AND LEXICAL ANALYSIS**
------------------------------------------------------------------------------------------------------------------------

- In this module, we understand some input strategies.

- Then we program a model to demonstrate how `regular expression` is translated into a `lexical analyzer`.


## LEXICAL ANALYZER AS PART OF COMPILER

- `Lexical Analyzer` translate input stream into a form that is more managable by `parser`.
- It translates input strings `lexemes` into tokens  ansd then into `arbitary integer value` that represent `lexeme`.
`example : SEMI -> 2 , EOI -> 1`

- `Lexical Analyzer` contain various features :
  - It strip out all commnets.
  - It skip over white spaces.
 
- Isolating `lexical anayser` from parser leads to easy implementation and simplify designing process.
- `Analyzer` keep track of current line so that intellegent error message can be thrown as output by `parser`.
- `Lexical Analyzer` is an independent compilation phase.
- It communicates with parser over well defined and simple interface.

**_parser calls a single lexical analyzer subroutine when it needs a new token and that subroutine return `token` and the associated `lexeme`._**

<p align ="center">
  <image src ="https://github.com/teche74/CompilerCrafting/assets/129526047/00759483-87c1-4b54-89e6-07a7b3bcf9c0">
</p>
- `Lexical Analyzer` is an independent phase which means any modification and updation doent affect any other unit of compiler.
- Sometimes we need more complex interaction between `lexical analyzer` and `parser`.
**example : `typedef is used to create new keyword after parser processed the statement.`**.While it requires high level communication which is done through `symbol table`.

> NOTE : Its best to restrict the lexical analyzer to simple pattern recognition task in order to make it easier to maintain.


## ERROR RECOVERY IN LEXICAL ANALYZER

- Error can occur in both `lexical analyzer` and `parsing` phase.
**example : using of `@` and `'`` is illegal when used outside string.**

- How lexical analyzer recoverfrom these errors ??
  - lexical analyzer have multiple choices :
    - Either discard the entire lexeme.
    - It can discard the first character and try to rescan. 

- Sometime some `lexical anlyzer` can correct it by itself, but only in case when there is one mispelled character.

**_Before talking about errors, lets undesratnd the very first concept of input where data is read with help of `operating system`_**

-  Most of good compilers are formed by optimizing their lexical anlayzer giving it the most time in creation phase as most of system-dependent operations are concentrated to input layer.

## WHY USING A C STANDARD BUFFER INPUT SYSTEM IS A BAD CHOICE ? 

**ISSUE1**
- Most buffer system copy input characters at least 3 times before program can use them.
  - Copied from disk to file buffer maintained by operating system.
  - Copied from file buffer to second buffer.
  - Copied from secodn buffer to strign that holds the lexeme.
- This process takes both time and space. Also the buffer space is not optimized.

**ISSUE2**
-  Other issue is with pushback or lookahead. To distinguish between token and keyword `lexical analyzer` must have to lookahead several characters. If identified then must pushback the extra characters to input.
`example : if then then then = else ; else else = then`

