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

- Here our `lexical analyzer` face difficulty in differentiating between else keyword from else identifier. It is done by looking at forward character. It means that we can say that a word must be an identifier if it is followed by an equal length.

`example : declare(args1,args2,...., argsn)`
- This time our `lexical analyzer` cannot distinguish keyword from identifier until right paranthesis get read. Here we need push back. another example : `differ in max() or max`.

<br>

### LET'S UNDERSTAND HOW PUSHBACK IS NECESSARY EVEN IN RECOGNIZING SINGLE TOKEN

**_Lets suppose there are 3 tokens : `xxyy , xx and y`. if we give analyzer `xxy` as input. It should return xx token followed by y. To distinguish between tokens, it reads atleast 4 character and then push back two y's back into input stream._**

- Another choice is to use `UNIX Lex` for this, but `LEX` not make any assumption anout structure of lexeme which makes it worst choice.

### HOW `LEX` WORKS

- It creates a layer around `getc()` using stack to get more pushbacks, then it get next next input from either stack or input stream. Here we cannot use normal buffer system like `ungetc()` as it only gives a single character pushback.

## CODE FOR USING STACK FOR MULTIPLE CHARACTER PUSHBACK

```c
#include<stdio.h>
#define SIZE 128     /* Maximum number of pushback characters */

              /*         About Variables

                *    Pbackbuff -> name of stack.

                *    Pbackp    -> stack pointer.

                *    NOTE ->  Stack grows down.
                      *  push ->  *--Pbackp =c.
                      *  pop  ->  c = *Pbackp++.
                *   get() ->  evaluates next input character, either by popping or getc().
                *   unget(c)  -> pushes c back (return -1 for unsuccesful operation and c for sucessful operation ).
              */

int Pbackbuff[ SIZE ];
int *Pbackp = &Pbackbuff[ SIZE ];

#define get( stream ) (Pbackp < &Pbackbuff[SIZE] ? *pbackp++ : getc(stream) )
#define unget( c ) (Pbackp <= Pbackbuff  ? -1  : *--Pbackp = (c) )

void ungets(char * start , int n){
  //Push back last n characters by traversing backward through string.
  char * p = start + strlen( start ); // get end of string.

  while(--p && --n >= 0 ){
      if( unget(*p) == -1){
          fprintf(stderr,"pushback stack overflow \n");
      }
  } 
}
```

## EXAMPLE INPUT SYSTEM

- Let's understand input system used by `LEX-generated lexical anlayzer`. It is not used by `LEX` itself.
- It has many desirable qualities :
  - Routine should be as fast as possilbe.
  - Several characters of `pushback` and `pushahead` must be available.
  - Both previous and current `lexeme` must be avilable.
  - Disk access should be efficient.

**_We had already  discussed that input system is highly concerned area in compiler cnstruction. Lets understand How disk is accesed?_**

## UNDERSTANDING HOW DISK IS ACCESED ? 

- OS `operating system` store disk and organize them into sectors.
**We can say that `sector` is the fundamental unit for disk storage. Disk read is performed in sector size chunks.**

- `Example : `If your disk is 512 byte sector, then you must read 512 at a time. This cause the problem with hardware.

## PROBLEM IS WITH HARDWARE ..😵💭

- Suppose you request a byte, then the `operating sytem` will read entire sector into buffer, then return single byte to you from buffer. This entire porcess is done eyerytime until entire buffer gets exhausted.Then it will read new sector. Now you understand that how it is decreasing efficiency and why we had said that `problem is with hardware`😖.

**Some Other approaches OS perform for disk access**

**CASE 1**
- Some `operating system` works on `allocation unit`. `Allocation Unit` can be termed as the minimum byte that can be read.
- OS `operating system` will perform all read operations in terms of `allocation unit`.
- It means `the number of bytes read from disk at any one time should be the multiple of allocation unit`.
**_"Larger the buffer, Shorter the read time"_**

**CASE 2**
- Other OS `operating system` rewards you on block sized transfer. Due to block size transfer it elimiates one level of copying and decreasing read time. It means that `operatign system` will directly transfer buffer.


**CASE 3**
- Some `operating system` will also follow a design with single input buffer and several pointers.
  - `BUFSIZE :` The actual buffer size.
  - `MAXLEX :` maximum length of `lexeme`.  **NOTE : Diskread always done in multiple of MAXLEX**
  - `STARTBUFF :` mark the phyical start of the buffer.
  - `END :` mark the physical end of the buffer.
  - `ENDBUFF :` mark the logical end of the buffer.
  - `NEXT :` points next input character.
 
- Lets visulaize buffer after it is loaded for first time.
<p align ="center">
  <image src ="https://github.com/teche74/CompilerCrafting/assets/129526047/5337782e-8d3c-4748-9c8f-5bc18d2e7e15">
</p>

- Buffer normal state when lexical analyzer has processed several states.
<p align ="center">
  <image src ="https://github.com/teche74/CompilerCrafting/assets/129526047/abfb0643-32ac-428f-aadf-78233832093f">
    
  **pMark : beginning of previous lexeme**
  
  **sMark : beginning of current lexeme**
  
  **eMark : end of current lexeme**
</p>

- If `lexical analyzer` find longer lexeme than current one, it moves eMark to current input location.
- If it founds that it read too far in input, it can push back all extra characters by setting Next back to eMArk. `NEXT = eMark`
- `MAXLOOK :` number of lookahead characters that are supported.
- `DANGER :` marker that tells the input routine when next pointer is getting too close to the end of buffer.
**_NOTE :When next crosses danger point, a buffer flush is triggered. Due to which all character between pMark and last valid characterhas been shifted to far left of buffer. Now input routine fills the reaminder empty portion of buffer from disk by reading aas many `MAXLEX`sized chunks._**
<p align ="center">
  <image src ="https://github.com/teche74/CompilerCrafting/assets/129526047/2d53209c-3a6f-4391-aeea-1312d0afb50d">
</p>

#### ADVANTAGES OF THE SINGLE INPUT BUFER AND POINTERS APPROACH 
- Advantages of using thsi appproachas follow : 
  - Lack of copying.
  - Push back can be done by `NEXT = eMark` rather than series of push and pop.
  - Disk reads are done in block size lead to eliminate one extra stage of copying.

## IMPLEMENTATION OF INPUT SYSTEM

- This file contain all macros and Data structures
```c
#include<stdio.h>
#include<stdlib.h>
#include<fnctl.h>
#include<tools/debuh.h>

#include<tools/l.h>
#include<string.h>

#ifdef MSDOS
#   define COPY(d,s,a) memmove(d,s,a)
#else
#   define COPY(d,s,a) memcpy(d,s,a)
#endif

#define STDIN 0

#define MAXLOOK 16
#define MAXLEX 1024

#define BUFSIZE  ( (MAXLEX * 3) + (2 * MAXLOOK) )

#define DANGER (end_buf - MAXLOOK)

#define END (&Start_buf[BUFSIZE])

#define NO_MORE_CHARS() (Eof_read && Next >= End_buf )

typedef unsigned char uchar;

PRIVATE uchar Start_buf[BUFSIZE];
PRIVATE uchar *EndBuf = END;
PRIVATE uchar *Next = END; 
PRIVATE uchar *sMark  = END;
PRIVATE uchar *eMark = END;
PRIVATE uchar *pMark = END;
PRIVATE int  pLineno = 0;
PRIVATE int  pLenght = 0;

PRIVATE int  Inf_file = STDIN;
PRIVATE int  Lineno = 1;
PRIVATE int  Mline = 1;
PRIVATE int  Termchar = 0;
PRIVATE int  Eof_read = 0;

PRIVATE int open(), close(), read();

PRIVATE int ( *Openp )() = open;
PRIVATE int ( *Closep )() = close;
PRIVATE int ( *Readp)() = read;
```

