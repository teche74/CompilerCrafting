---------------------------------------------------------------------------------------------------------------------------------

# UNDERSTANDING THE KEY COMPONENTS OF A COMPILER 😲

----------------------------------------------------------------------------------------------------------------------------------



## INTRODCTION
- Compilers are complex programs.
- Compilers can be broken down into distict chunks called 'PASES'. Pases communicate with each other via temporary files. They are also the part of compilation process.


## `COMPILER`
- `Compiler` is a program or group of programs that translate one language into another language.

> IN OUR CASE WE ARE TRANSLATING HIGH LEVEL LANGUAGE INTO MACHINE UNDERSTANDABLE LANGUAGE.


## STRUCTURE OF A TYPICAL FOUR PASS COMPILER 

<p align="center">
  <image src ="https://github.com/teche74/CompilerCrafting/assets/129526047/38ae6d3a-9517-403f-ae0e-99387429873b">
</p>

- FIRST PASS ( `THE PREPROCESSOR` )
  -  The Preprocessor do macro substitutions.
  -  It also strips comments.
  -  It also handles various housekeeping tasks to prevent compiler from burdens.
 
- SECOND PASS ( `HEART OF COMPILER` )
  -  It includes `lexical analyzer`, `parser` and `code generation`.
  -  It is used to convert source code into intermediate language that is more similar to assembly.

- THIRD PASS ( `THE OPTIMIZERS` )
  -  It improves qulaity of generated intermediate code.

- FORTH PASS ( `THE BACK END` )
  -  It translates the optimized intermediate code into real assembly code or some form of binary executable file.
 
> `NOTE` : There are many variations of this structure , for examples - there are compiler with no preprocessors, some compilers can directly generate asssembly language in second pass and the modify the assembly language, some compiler can directly generate binary instructions.
> A data stucture called `symbol table` is also used which is shared and used by all passes.


## THE LEXICAL ANALYZER

- The first `phase` of a compiler.   `Phase refers to independent tasks used in compilation process.`
- Multiple phases are combined in single pass. `Example : second phase of compiler` 
     <p align="center">
         <image src = "https://github.com/teche74/CompilerCrafting/assets/129526047/0168f622-38e5-4346-8464-60dde752f1bf">
     </p>
- It also termed as `Scanner` and `Tokenizer`.
- It converts/translates input stream to a form that is more usable for rest of compiler.
- `Lexical Analyzer` convert input streams as a collection of basic language element called tokens. `Token : indivisible lexical unit`
- Examples  : `while`, `for`, `<=` , `>=`, `<<`, `&` etc..
- Original strings from which tokens are constructed is called `Lexeme`.

#### LEXEME VS TOKENS

`Lexeme :` The raw sequence of characters in your code.
`Token :` The recognized unit or category assigned to a lexeme by the compiler.

| Lexeme   | Token         |
|----------|---------------|
| `123`    | Number        |
| `foo`    | Identifier    |
| `+`      | Plus Operator |
| `if`     | If Keyword    |
| `==`     | Equality      |
| `= `     | Assignment    |
| `while`  | While Keyword |
| `23.45`  | Number        |
| `bar`    | Identifier    |
| `*`      | Multiply      |

#### CONDITION OF OVERLAPPING

- Sometimes it get difficult for compiler to assign tokens and token class. There are several scenarios when compiler faces difficulty : 
  - Multiple Interpretations
     - **Example:** Consider the characters `>>` in a programming language.
     - **Scenario:** It might represent either the "greater-than" operator (`>`) or the "shift" operator (`>>`).
     - **Challenge:** Deciding which interpretation to choose based on the context.

  - Token Priority
    - **Consideration:** Different tokens may have varying priorities.
    - **Resolution:** The "longest match rule" is often employed.
    - **Principle:** Choose the token associated with the longest matching sequence.
    - **Example:** Resolving `>>` as a "shift" operator rather than two "greater-than" operators.

  - Ambiguous Grammar
    - **Situation:** The language's grammar might permit ambiguous situations.
    - **Importance:** Resolving ambiguity is critical to ensure correct interpretation during lexical analysis.


#### RESOLVING THE PROBLEM
- To avoid confusion, `Compiler` use a simple rule. It chooses the longest interpretation rule. It means ` while reading codes, the compiler looks for meaningful chunks (`lexemes`) and then assign them category (`tokens`). Sometimes when character can be part of different categories, the compiler chooses the meaningful chunk to avoid confusion `.
- Tokens internally represented as unique integersor an enumerated type.
- Generally Arithemetic Operators with same precedence and associativity can be grouped together, similary Type declaration keywords are gruped together.  
- `Lexical Analyzer` is a self contained unit that communicates with rest of compiler through subroutines and global variable.

> `NOTE  1 : `  `Parser` call the anlayer every time it needs a new token. It returns token with associated lexemes.
<p align="center">
         <image src = "https://github.com/teche74/CompilerCrafting/assets/129526047/c8fecb09-b005-49b1-87ce-d1f700f60295">
</p>

> `NOTE  2 : ` Implementation of Analyzer is hidden from parser, this means you can modify or replace it without affecting rest of compiler. 
