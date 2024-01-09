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


## PARSERS

- `Commpilers` are used for translation of high level language to machine level language like `8086` and `assembler`. In this process `parser` helps compiler to decompose a sentence into its component parts.
-  The decomposition of sentence is represented in tree form rather than sentence daigram `where sentence is entire program`.
-  Sentence daigram have the property to show their syntatic relationship betwee part of sentences itself. This representation is often called  `Syntax Daigram/Syntax Tree`.

- We can expand Syntatic Tree to visualize both grammatical and syntatic structure.


    **_A PARSER IS A GROUP OF SUBROUTINES USED FOR CONVERSION OF INPUT STREAM _TOKENS_ INTO A PARSED TREE_**

- `example : ` A * B + C * D
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/b4caac0b-07a3-4ee1-9ade-639b0678d2fa">
</p> 

## CODE GENREATOR

- Last part of `Compiler` which is somewhat misleading as it is represented as a seperate component from parser, while actually most compilers generate code as the parser progress.
- Code was generated  by sam subroutines which perform parsing.
    - `CASE 1 : ` Either `Parser` creates the parse tree for entire input file which further traversed by distinct code genreators.
    - `CASE 2 : ` Parse generates an Intermediate language of input file from which syntax tree can reconstructed using optimal pass.

- `Code Generator` translates input file into an intermediate langauge rather than machine code directly. Its then translated to machine code by `back end`.

#### WHY USING INTERMEDIATE LAGAUAGE CONVERSION ? 💭💭🧐

| **Advantages**                                       | **Disadvantages**                                         |
|------------------------------------------------------|-----------------------------------------------------------|
| It provides optimizations like `simple constant folding`. | Compiling times may double due to the intermediate conversion. |
| Simple constant folding evaluates constant expressions at compile time. |                                                           |
|                                                       |                                                           |


## GRAMMER AND PARSE TREE

- Generally `Formal Grammer` method is used to describe a programming lnaguage.
- `Formal Grammer` most often represented in modified `BNF (Backus Normal Form)`. Strict BNF representation starts with set of tokens ` ( called terminal symbols) ` and set fo symbols ` ( called non-terminal symbol ) `.
-  One operator is supported which is ` ::= operator ` which means **_is defined as_** or **_goes to_**.
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/1e02290e-4518-4c9c-8daa-d58b8432f173">
</p> 
- Each rule of this type is called `Production`. we can also termed it as ` ( **Left side of Production** ) ::= ( **Right side of Production** ) `.

## HOW REAL GRAMMER WORKS

- `Grammer` works in such a way that it continue with further defination until all the terminal symbol are accounted.
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/47495bcf-8a2d-4c67-8da5-85e684762da7">
</p> 

- Sometimes mulitple chaoices in Rule production is shown by ` OR operator  ( | ) `.
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/250821bf-38cf-4049-b12b-10066c6f13ed">
</p> 

- `NOTE : ` Make `Grammer` as much flexible as you can.
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/b468cb02-1628-4523-aeb7-70b1ec5da7db">
</p> 

- Optionality in Rules is shown by using  `epsilon production ( ε ) `
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/6bb7bdd0-97e0-491c-b1f7-19a13a0b0383">
</p> 

- **PARSER determines whcihc production to apply by examining the next input symbol.**  
  - Suppose that we had defined a grammer below that recognize a limited set of English sentences below.
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/9890b74e-82b0-42c6-86d5-b6d4fd8ff905">
</p> 

  - Lets take an example : ` JANE SEES SPOT RUN `. and check how input series is recognized.
    - Steps ivolved in series recognition are :
      - There must be a series of replacement.
      - Start out with topmost symbol `( goal symbol )`.
      - Replace that symbol with one in right hand side.
      - Replace all leftmost non-terminals with its right-hand side.    
**_STRUCTURE OF PARSE TREE_**
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/f05c0066-928b-45c0-a32f-fad843242980">
</p> 

## AN EXPRESSION GRAMMER
- Some grammer recognizes a list of one or more statements.
- Statements are made up of series of semicolon delimeter expression.
  - It means multiple expression as semicolon as its delimeter.
  - It either comprises of number seperated by non terminal symbols like (+, *) etc.
  - `NOTE : ` `Grammer` is Recursive in nature. Thsi means resursion is used to parse grammer. Even a well recursion makes it possible for a finite grammer to recognize an infinite number of sentences.

-------------------------------------------------------------------------------------------------------------------------------
- Grammer is set of rules that defines structure of langauge.
- Recursion is used to analyze and parse the grammer.
- Grammer is designed to be intutive reflects how expression are put together.  
-------------------------------------------------------------------------------------------------------------------------------


## LEFT RECURSION CHALLENGE

### Understanding Left Recursion

Left recursion is a challenging aspect in grammar and parsing, particularly in the context of designing parsers for programming languages. Left recursion occurs when a non-terminal in a grammar produces a sentential form that starts with itself. This can lead to ambiguity and difficulties in parsing.

Consider a production rule of the form  `A -> Aα | B ` where `A` is a non-terminal, and `α` and `β` are sequences of terminals and/or non-terminals. This type of production introduces left recursion.

### The Challenge

Left recursion poses a challenge for recursive descent parsers, as they may fall into an infinite loop when trying to expand such recursive rules. To handle left recursion, it's essential to rewrite the grammar in a way that eliminates left recursion.

### Demonstrative Example

Let's take a simple example to illustrate the left recursion challenge:

Consider a grammar with the following production rule:
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/d5ea4c43-a4a1-4a41-837e-266ceed8a06c">
</p> 


**_Problem occurs when expression tries to expand and gets expression again which leads to an infinte loop_** To eliminate this we are using null production.
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/3872f655-30ab-41ea-9019-8de4e3737872">
</p>

<br>
## SYNTAX DAIGRAM

- It helps to represent entire syntatic structure of a parsed sentence.
- It is useful in writting recursive - decent compilers because they can directly translate to flow charts.
- _How to translate into syntax daigram_
  - Mulitple production get merged into single daigram.
  - ε production is represented by uninterrepted line that doesn't go through box.
  
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/2512ecb3-30fc-476c-9974-b9350990b555">
</p>

<br>
## A RECURSIVE DECENT EXPRESSION COMPILER

- **_Lets try to build a smaller compiler using the expression grammer_**.

`GOAL : Take Simple arithematic expression as input and generate code that evaluate those expression at run time`.
<p align="center">
  <image src="https://github.com/teche74/CompilerCrafting/assets/129526047/50a01d91-f34b-4b7c-a103-87db36379f67">
</p>

### LEXICAL ANALYZER
- First step is to defining a token set.
- Number and identifiers are exceptions, otherwise all lexemes are single character.
- `NOTE : `
  - `token : ` indiviseble unit disambiguited by lexemes.
  - `lexemes : ` Comprises of input string.
- `NUM_OR_ID` token is used for both nubers and identifiers. It is made up of series of contiguous characters. `example :  ( '0' - '9' ) , ( 'a' - 'z' ) , ( 'A' - 'Z' ) `.

**_ABOUT LEX.H FILE_**

- Tokens themselves defined with macros at top of lex.h file.
- Lexical Analyzer translates a seicolon into `SEMI` token.
- Series of digit into a `NUM_OR_ID` token and so on..
- Three external variables at bottom of lex.h are used by lexical analyzer to pass information of parser.
  - `yytext : points at current lexeme.`
  - `yylang : number of characters in lexemes.`
  - `yylineno : current input line number.`
- `Lexical Analyzer starts itself. It uses a simple, buffered, input system, which gets character a line at a time from standard input.`
<br>

**_Isolating tokens, one at a time, from the line_**

<br>

- Another input line is fetched when entire line is exhausted.

### ADVANTAGE OF USING BUFFER SYSTEM ?? 🤓😲

- Main advantage of using buffer system for input :
  - `Speed : ` Computers like to read data in large chunks because `larger the chunk higher the throughput`. It is noticible only when size of buffer > size of disk cluster.
  - `lookahead and pushback feature : ` It is easier to move back and predict next when using buffer.

**LOOKAHEAD and PUSHBACK ??**
- Sometimes lexical analyzer often have to know what newxt input character without reading it. It is done through `lookhead`.
- Sometimes analyzer had to move back as it gets too far, then we use `pushback`.

**_These two operations are easy to perform in biffers against characters. How lets understand ??_**
-------------------------------------------------------------------------------------------------------------------------------
#### UNDERTANDING CONCEPT OF LOOKAHEAD AND PUSHBACK

Imagine you are reading a story, and you want to predict next word without reading it. `This is lookhead`

Sometimes, you might accidently read a bit too far, then you need to "unread" or "go-back". `This is pushback`

**_Now think of doing this withone letter at a time, trying to predic next letter is very difficult. But if you have a whole word or sentence in from=nt of yo, it's easier to guess what comes next.If you read too far you can come back quickly. This is the reason buffer system is more preffered._**

-------------------------------------------------------------------------------------------------------------------------------


## _CREATING LEX.H FILE_

- **_Lex.h_ file contain token defination and extern statements**

```c
#define EOI              0          /*         end of input                 */
#define SEMI             1          /*             ;                        */
#define PLUS             2          /*             +                        */
#define TIMES            3          /*             *                        */
#define LP               4          /*             (                        */
#define RP               5          /*             )                        */
#define NUM_OR_ID        6          /*   decimal number or identifier       */

        /*   in lex.c    */
extern char *yytext;               /*     Lexeme (not '\0' terminated)      */
extern int yyleng;                 /*     Lexeme length.                    */
extern int yylineno;               /*     Input line number.                */
```

## _IMPLEMENTATION OF LEX.C_

- Implementation of `lexical analyzer` is done in this module.
- Steps of implemenattion as follow:
  - Including header files and macros and custom files.
  - Declaring global variable.
  - Implement lex() function.
    - Initialize buffer and pointer to buffer.
    - Skip leading whitespace characters.
    - Character Processing.
    - Token Recognition Cases.
    - Handling whitespaces newline or tabs.
    - Handling alphanumeric token.
    - Maintain lexeme information.


```c
#include "lex.h"                /*  It contains all macros and declaration    */
#include <stdio.h>              /*  It includes standar input output library  */
#include <ctype.h>              /*  It includes standard library for character classification and manipulation function */

char  *yytext = "";             /*     Lexeme (not '\0' terminated)      */
int    yyleng = 0;              /*     Lexeme length.                    */
int    yylineno = 0;            /*     Input line number.                */


lex()
{
    /*
        * Declares a static array of type char with size 128.
        * Declares a character pointer.
    */
    static char input_buffer[128];
    char  * current = NULL;
    /*
        ! Why static ?
            * Static keyword helps in static storage duration which means memory allocated to array remains allocated throughout the entire execution.
            
            * Static keyword restricts the scope, this mean input_buffer is only accesed and visible within file.
            
            * It helps input_buffer to initialized only once with all elements set to zero.
            
            * It is also useful to maintain or remember current state of input_buffer beteween calls to lex function.    

        ! Purpose of current ?
            * It is used to keep track character arrray.
    */

    current = yytext + yyleng;      /*  Skip  current  lexeme  */

    /* Getting the next one */
    while(1){
        while( !*current ){
            /*
                1 > Get new lines.
                2 > Skip leading spaces on line, until a nonblank line is found.
            */

            current = input_buffer;         /*   Set current pointer to beginning of input_buffer            */
            
            if( !gets( input_buffer ) )     /*              If End of File is encountered                    */
            {   
                *current = '/0';            /*  It sets end last lexeme character of input buffer to  '\0'   */
                
                return EOI;                 /* Return EOI to signify that there is no more tooken to process */
            }

            ++yylineno;     /* Increment line number */


            /*  Skip all leading spaces*/
            while( isspace( *current ) ){
                current++;
            }

            /*  Iterate over characters until  a null character encountered */
            for( ;*current;++current){
                /*  Get next Token */
                yytext = current;            /*  yytext represent the beginning of the lexeme */ 
                yyleng=1;                    /*  Set length of lexeme to 1                    */ 

                /*  Switch to get token against lexeme encountered  */
                switch( *current ){

                    /*
                        * If a specific character is encountered ( ;, +, *, (, ) ), the corresponding token is returned (SEMI, PLUS, TIMES, LP, RP).
                    */
                    case EOF : return EOF;
                    case ';' : return SEMI;
                    case '+' : return PLUS;
                    case '*' : return TIMES;
                    case '(' : return LP;
                    case ')' : return RP;

                    /*
                        * If the character is a whitespace character (\n, \t, ' '), it is ignored.
                    */
                    case '\n' : 
                    case '\t' :
                    case ' '  : break;

                    default : 
                        /*  Checks  for alphanumeric character  */
                        if( !isalnum( *current ) )
                            fprintf(stderr, "Ignoring illegal input <%c> \n", *current);    /* If not alphanumeric return error statement */
                        else{
                            /* If it is alphanumeric iterate until a non - alphanumeric encountered */
                            while( isalnum( *current ) ){
                                ++current;
                            }

                            yyleng = current - yytext;              /*               Calculating Length of Lexeme                             */
                            return NUM_OR_ID;                       /*     Token is returned to signify that a numeric or identifier is found */
                        }
                        break;
                }
            }
        }
    }
}
```
