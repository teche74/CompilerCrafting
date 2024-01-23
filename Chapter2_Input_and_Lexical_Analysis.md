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

## `INTITALIZATION ROUTINES CODE`
```c
void ii_io( int (*open_funct)() , int * (close_funct)(), int * (read_funct)() ){
  Openp = open_funct;
  Closep = close_funct;
  Readp = read_funct;
}

int ii_newfile(char * name ){
  int fd;
  MS( if( strcmp(name, "/dev/tty") == 0) )
  MS( name =  "CON" )

  if( (fd = !name ? STDIN : (*Openp) (name, O_RDONLY|O_BINARY)) != -1){
      if( Inp_file != STDIN ){
          (*Closep)( Inp_file );
      }
      Inp_file = fd;
      Eof_read = 0;

      Next = END;
      sMark = END;
      eMark = END;
      End_buf = END;
      Lineno = 1;
      Mline = 1;
  }
  return fd;
}
```

`ii_oo function` is used to change the low-level input functions that are used to open the file and fill the buffer. It is used only when you directly getting input fro hardware. This way we can use a `LEX` generated lexical analyzerin an unusual situtation without having to rewrite the input sysytem.

`ii_newfile()` is a normal mechanism for opening a new input file. It takes file name as argument and return file descripter. ( -1 if file ot get opened ) It actualy does'nt read the first buffer .It only setup various pointers so that buffer is loaded propeny when first character is requested. This way a program will neevr call `ii_new_file()`.

#### PROBLEM WITH THIS APPROACH  

- You must read atleast 1 character before you can look ahead in input.

> **SOLUTION** : If you need to `lookahead` befoer `advance` we use `ii_advance()` and `ii_pushback(1)`.  

`ii_advance` : Read first buffer full of input.
`ii_pushback` : Put back first chracter.

- Default input stream is standard input `used if ii_newfile never called`. We can reassign the input to standard buffer input by calling : `ii_newfile( NULL )`.

`open( ) `uses the O_BINARY input mode in `MS-DOS` system. A CR-LF `carriage-return, linefeed` pair is not translated into single **\n** when binary mode input is active.
  
  - Most `LEX` appilcations trat LF and CR as whitespace. It prevents them for wasting time on translation. It amy cause problem in case if explicit `\n` is given.

`NOTE : ` `ii_new_file()` not reads the input buffer.It initializes various pointers to the end of the buffer. The input system reads input in same way as `NEXT pointer crosses the DANGER`. It shifts buffer tail to left.


### `IMPLEMENT SMALL ACCESS ROUTINES AND MARKER MOVEMENTS`

```c
PUBLIC char *ii_text        () { return ( sMark         );  }
PUBLIC int   ii_length      () { return ( eMark - sMark );  }
PUBLIC int   ii_lineno      () { return ( Lineno       );   }
PUBLIC char *ii_ptext       () { return ( pMark        );   }
PUBLIC int   ii_plength     () { return ( pLength      );   }
PUBLIC int   ii_plineno     () { return ( pLineno      );   }

char *ii_mark_start(){
  Mline = Lineno;
  eMark = sMark = Next;
  return ( sMark );
}

PUBLIC char *ii_mark_end(){
  Mline = Lineno;
  return ( eMark = Next );
}

PUBLIC char *ii_move_start(){
  if( sMark >= eMark) return NULL;
  else return ++sMark;
}

PUBLIC char *ii_to_mark(){
  Lineno = Mline;
  return (Next = eMark);
}

char *ii_mark_prev()
{
  pMark   = sMark;
  pLineno = Lineno;
  pLength = eMark - sMark;
  return ( pMark ); 
}
```

- We introduced some access functions to limit extrnal access of global variable. Linker assumes the two global variable with same name as same variable.

- Suppose if you had done careless declaration of two variables with same name as same variable.One of them will seems to get magically change its value when a subroutine access the other one. To avoid this problem we use `static` ( limiting scope to the current file ).

- It is necessary for external subroutine to acess these variable and to do this in safest way is to acess them with small routines.These subroutine are used for maintainence reason only.

`NOTE :` **_Two Subroutines with same name must cause error but two variable with same name that are globally define does'nt cause any error as they are silently merged._**


`ii_text : ` pointer to current lexeme.
`ii_length : ` length of current lexeme.
`ii_lineno : ` line no for last character in lexeme.

`ii_ptext : ` pointer to previous lexeme.
`ii_plength :` length of previous lexeme.
`ii_plineno :` line no for last character in previous lexeme.


`ii_mark_start() :` routine moves the `sMark` to the current input position. It also make's sure that the end of lexeme `eMark` is not to the left of start mark.

`ii_mark_end() :`  routine moves the `eMark` to the curren input position. It also make's sure that the end of lexeme `sMark` is not to the right of end mark. It also saves current line no in Mline.

`ii_move_start() :` It moves the statr marker one space to the right.It return new start marker on success.

`ii_to_mark() :` restores input pointer to last end mark.

`ii_mark_prev() :` modfiy the previous lexeme markerto reference the same lexeme as current lexeme marker.    


## `IMPLEMENTING ADVANCE FUNCTION`
```c
int ii_advance(){
  static int been_called = 0;
  if( !been_called ){
    Next  = sMark = eMark = END -1;
    *Next = '/n';
    --Lineno;
    --Mline;
    been_called =1;
}
if(  NO_MORE_CHARS() ) return 0;
if( !Eof_read && ii_flush(0) < 0 ) return -1;
if( *Next == '\n') Lineno++;
return( *Next++ );
}
```
`ii_advance() :` It past the character in input system .

`NO_MORE_CHARS :` It is used to detect end of file.

  `#define NO_MORE_CHARS() (Eof_read && next >= End_buff)`

## `IMPLEMENTING BUFFER FLUSH`


<p align ="center">
  <image src= "https://github.com/teche74/CompilerCrafting/assets/129526047/01e37d65-1874-4dd4-aa4d-739340d35deb">
</p>



```c
  int ii_flush( int force ){
  
    int copy_amt,shift_amt;
    char * left_edge;

    if( NO_MORE_CHARS() ) retur 0;
    if( Eof_read ) return 1;

    if(Next >= DANGER || force )
    {  
      left_edge = pMark ? min(sMark, pMark) : sMark;
    
      shift_amt = left_edge - Start_buf;
    
      if( shift_amt < MAXLEX){
          if( !force ) return -1;
          leftedge = ii_mark_start();
          ii_mark_prev();
          shift_amt = left_edge - Start_buf;
        }
        copy_amt = left_edge - Start_buf;
      
        COPY( Start_buf, left_edge, copy_amt );
      
        if( !ii_fillbuf( Start_buf + copy_amt) )
          ferr("INTERNAL ERROR", ii_flush: Buffer full, can't read.\n");
      
        if( pMark )
          pMark -= shift_amt;
          sMark -= shift_amt;
          eMark -= shift_amt;
          Next  -= shift_amt;  
    }
    return 1;
}
                                   /*   ------------------------------------------------------------------------------------------------------------------------------- */

PRIVATE int ii_fillbuf( unsigned char *starting_at ){
  register unsigned need, got;

  need = ( (END - starting_at) / MAXLEX ) * MAXLEX;

  D( printf( "Reading %d bytes\n",need ); )

  if( need < 0 ) ferr("INTERNAL ERROR (II_FILLBUF) : bad read-request starting adr. \n");
  if( need == 0 ) return 0;
  if( (got = (*Readp) (Inp_file, starting_at, need)) == -1 )
    ferr("Can't read iinput file\n");

  End_buf = starting_at + got;

  if( got < need ) Eof_read = 1;

  return got;
}
```

`ii_flush() :` it flushes the buffer if necessary.

### **IMPLEMENTING IILOOK AND IIPUSHBACK**

```c
int ii_look( int n ){
  uchar *p;
  p = Next + (n-1);

  if( Eof_read && p >= End_Buf) return EOF;

  return ( p < Start_buf || p >= End_buf ) ? 0 : *p ;
}

int ii_push_back( int n ){
  while( --n >= 0  && Next > sMark){

      if(*--Next == '\n'  || !*Next ) --Lineno;

  }

      if( Next < eMark ){
          eMark = Next;
          Mline = Lineno;
      }

  return (Next > sMark);

}
```
`ii_look` returns character at the offset from the current character that's specified in the argument. `ii_look(0)` returns character at that was return by most recent `ii_advance()` call.

`ii_pushback` pushes back the nth most recent read chracters.


`NOTE :` It's ocassionaly useful to have a terminator on the string.

### IMPLEMENTING SYSTEM SUPPORT FOR '\0'- TERMINATED STRING

```c
void ii_term(){
  Termchar = *Next;
  *Next = '\0;
}

void ii_unterm(){
  if( Termchar ){
      *Next = Termchar;
      Termchar =0;
  }
}

int ii_input(){
  int rval;
  if( Termchar ){
      ii_unterm();
      rval = ii_advance();
      ii_mark_end();
      ii_term();
  }
  else{
      rval = ii_advance();
      ii_mark_end();
  }
  return rval;
}

void ii_input( c ){
    if( Termchar ){
        ii_unterm();
        if( ii_pushback(1) ){
              *Next = c;
        }
        ii_term();
    }
    else{
        if( ii_pushback(1) ){
            *Next = c;
        } 
    }
}

int ii_lookahead( n ){
  return ( n==1 && Termchar) ? Termchar : ii_look(n);
}

int ii_flushbuf(){
    if( Termchar ){
        ii_unterm();
    }
    return ii_flush(1);
}
```
`ii_advance()` : used to past characters given by buffer_flush (return next character).
`NO_MORE_CHARS` : used to detect end of file.
`ii_flush()` : it flushes the buffer if neccessary.
`ii-look()` : return charcaters at offset fron the current character.

<br>

-------------------------------------------------------------------------------------------------------------------------------------------------------------
### LEXICAL ANALYSIS
-------------------------------------------------------------------------------------------------------------------------------------------------------------

- After developing set of input routines, we need to apply it on lexical-analysis application.

### 2 approaches for lexical-analysis :

- We can `hard code the analyzer` recognizing lexemes with nested if-else,switch or statements. If lexemes are not too long, one effective approach to use series of lookup table to recognize token.
  - Advantages :
    - It tends to be very efficient.
  - Disadvantage :
    - They are difficult to maintain.     


**_LOOKUP TABLE FOR CHARACTER RECOGNITION_**

```c
#define LESS_THAN 1
#define GREATER_THAN 2
#define EQUAL 3
#define NOT 4
#define LESS_THAN_OR_EQUAL 5
#define NOT_EQUAL 6
#define ASSIGN 7
#define ERROR -1
#define CONTINUE 0
#define SIZE_OF_CHARACTER_SET 128

char first[SIZE_OF_CHARACTER_SET];
char second[SIZE_OF_CHARACTER_SET];

int s,f;

memset(first,-1,SIZE_OF_CHARACTER_SET);
memset(second,-1,SIZE_OF_CHARACTER_SET);

first['>'] = GREATER_THAN;
first['<'] = LESS_THAN;
first['!'] = NOT;
first['='] = ASSIGN;
second['='] = EQUAL;

c = get_char();

if( f == first[c] == ERROR) return ERROR;

if( s== second[c] == ERROR){
    ungetchar();
    return f;
}
else{
    if(s == EQUAL){
        switch(f){
            case ASSIGN: return EQUAL;
            case LESS_THAN: return LESS_THAN_OR_EQUAL;
            case GREATER_THAN : return GREATER_THAN;
            case NOT : return NOT_EQUAL;
        }
        return ERROR;
    }
}
```


### LANGUAGES

`Alphabet` : Finite set of symbols.
  - example : set of ASCII , set of binary {'0','1'}.

`String` : Sequence of alphabetical symbols.
  - NOTE : empty string must be given by ε.


**_Difference between ε, null and empty string_**

`ε string` : when an empty string is given.
`null string` : string with absence of valid refernce.
`empty string` : character sequence with ero length.


`Language` : set of strings that can be formed by using alphabets.

`Grammer` : Ordering of strings within sentence is defined by collection of syntatic rules called a grammer.

`Kleen Closure ` : **( * )** If `L` is a language , then `kleen closure` of `L` is `L` repeated zero or more times. It is represented by `L*`.

`Positive Closure` : **( L+ )** `Positive Closure of L` is L repeated one or more times.

<br>

-----------------------------------------------------------------------------------------------------------------------------------
### **Regular Expression**
-----------------------------------------------------------------------------------------------------------------------------------

- Any well formed formulas over union, concatenation and kleen closure. We can match complex sequence by adding special called meta characters.
 
- Followinw are some rules used by lex to form a regular expression :
   - A single character that is not a etacharacter is a regular expression.
   - Two regular expressions concatenated forms a regular expression.
   - A period `.` matches any chracter except newline.
   - A dollar sign `$` anchor the pattern to end of line.
   - `[...][^...]` matches any of the characters embedded in the bracket.
   - `Negative charcter class` **^** It means any character except the one specified.
   - Only seven characters have special meaning inside a character class.
      - **{** : start of macro name.
      - **}** : end of macro name.
      - **[** : start of character class.
      - **{** : end of character class.
      - **-** : range of charcaters.
      - **^** : indicates negative character class. It doesnt match any newline character.
      - **" "** : takes away special meaning from charcter upto next quote mark.
      - **\** : takes away special meaning of next charcater.
    

A `regular expression` followed by a  **_*_** matches that expression repeated zero or many times.

`a+` : matches one or more repetations.
`a?` : matches zero or one repetation.

- Two `regular expression` seperated by vertical bar `|` recognizes a match of first expression **OR** a match of second.
- `Paranthesis` are used for grouping.

#### **LIMITATIONS OF REGULAR EXPRESSION**

- It can only be used to define sequence of character.
- It cannot do things like recognizing any number of properly nested paranthesis (Something that can be reconized grammatically). It is also the reason why parser and anlayzer are twoo seperate modules.
- Lexical anlayzer is responsible for recognizing simple sequence of characters and parser recognizes more complex combinations.

**REGULAR EXPRESSION OPERATOR PRECENDENCE**
<p align = "center">
    <image src = "https://github.com/teche74/CompilerCrafting/assets/129526047/c933b6af-cc2e-49fa-b891-1e85d1356d48">
</p>

**REGULAR DEFINATIONS**
- Regular definations build up a language specification using cobination of regular expression operator and production like specifiers.
  - keyword : long/int/double/while....
  - digit : 0/1/2/3/4....
  - digit_sequence : digit+
  - sign : +/-
  - exponent_part : e_sign ?  digit_sequence | E_sign ? digit_sequence.  


---------------------------------------------------------------------------------------------------------------------------------------------------
#### FINTIE AUTOMATA  
---------------------------------------------------------------------------------------------------------------------------------------------------

- A recognizer program, such as a lexical anlayzer ,reach a string as input and output "YES" if the string is a sentence in a lanaguage and "NO" if it's not.

- A extra layer is usually added around the recognizer itself so we can do more than "YES" and "NO".

- So basically first layer is used to recognize a string, while second layer perform an action associated  with that string.

- We translate regular expression that represent the lexemes into finite automation/ finite state machine.

-  `FSM` consist of follwing :
  - A finite set of states.
  - A set of transistions from 1 state to another.
  - A special start state.
  - A set of final or accepting states.


`Example of daigram that recognizes only 'he' , 'she', 'his' , 'him' and 'hers'`.
<p align = "center">
    <image src = "https://github.com/teche74/CompilerCrafting/assets/129526047/866ee2bd-4360-4d90-9b74-3c1e8d7fb091">
</p>

- State machine can be modelled using two datat structure.

- A single variable holding current state machine.
- A 2D array for computing next state.
  - One axis is indexed by the input charcter.
  - Other is indexed by current state and array holds the next state.

- We can represent previous machine using two arrays.
  - One array is used to hold state transitions.
  - Another array is used to tell whether a state is accepting or not.
  - Next state is determined from current state and input character.
```c
 next_state = Transistion_table[input_character][current_state];
 if( accepting[next_state] == 1 ){
    do_an_accepting_action(next_state);
 }
```

- Input character usulaaly called `lookahead character`. In this way we have implemented `DFA` as we can determined next state by using current state and current lookahaead charcter.

**_DFA is a state machine in which all outgoing edges are labeled with an input character._**

- While we can also create `NFA` which has no limitations on the number and type of edges. Unfortunately, `DFA's` are difficult to construct directly from regular expressions while `NFA's` arre easy to construct.
- `NFA's'` are more prefered because it is easier to represent in  computer program . `NFA's` can be an akward data
structure to use. It can have more states than equivalent DFA


**NFA model consist of :**
- A set of States **S**.
- A special state **S** called `start state`. Machine is initially at this state.
- A set of states inS called accepting states, entry into which denotes recognition of a string. Some sort of action is usually associated with each of the accepting states.
- A set of input symbols (an input alphabet). 


**DFA is an NFA with following restrictions :**
-  No state can have an outgoing ε transition .
-  There may be no more than one outgoing transition from any state that is labeled with the same character.

<br>

------------------------------------------------------------------------------------------------------------
### STATE MACHINE DRIVEN LEXICAL ANALYZERS
------------------------------------------------------------------------------------------------------------

- Let's understand how state machine are used for lexical-anlaysis y looking at high level.

- A simple table-driven lexical-analyzer that recognizes decimal and floating-point constants.

```c
[0-9]+                                                         return ICON;
([0-9]+1 [0-9]*\. [0-9]+1 [0-9]+\. [0-9]*) (e[0-9]+)?          return FCON; 
```


- This code is executed by the lexical analyzer when an input string that matches that expression is recognized.
  - The first expression recognizes a simple sequence of one or more digits.
  - The second expression recognizes a floating-point constant. The `( e [ 0- 9] +) ? ` at the end of the second regular expression is the optional engineering notation at the end of the number.

**We can write it in more simplified manner**
```c
( [0-9] + [0-9]*\. [0-9]+ [0-9] +\. [0-9] *) 
```
<p align = "center">
    <image src = "https://github.com/teche74/CompilerCrafting/assets/129526047/f371e96c-5935-4b87-86cc-b574556810a1">
</p>

- LEX uses a state-machine approach to recognize regular expressions, and a DFA that recognizes the previous expressions as shown above.
-  The next state is computed using that array with: `next_state = array[ current_state ] [ input ] `.
-  A dash indicates a failure transition (no legal outgoing transition on the current input character from the current state). This array is typically called a transition matrix or transition table. 

> The state machine itself and the driver program that uses that machine are distinct from one another. Two algorithms are commonly used in lexical analysis applications, and the same state machine (transition matrix) is used by both algorithms. A greedy algorithm is used by LEX (because that's what's required by most programming-language specifications). This algorithm finds the longest possible sequence of input characters that can form a token. The algorithm can be stated informally as follows: If there's an outgoing transition from the current state, take it. If the new state is an accepting state, remember it along with the input position. If there's no outgoing transition (the table has a a dash in it), do the action associated with the most-recently seen accepting state. If there is no such state, then an error has occurred (IJX just ignores the partially-collected lexeme and starts over from State 0, in this situation).

<p align = "center">
    <image src = "https://github.com/teche74/CompilerCrafting/assets/129526047/74d80642-7fbc-432d-8986-d1efedc6c504">
</p>

<br>

#### ALGORITHM USED BY LEX STATE-MACHINE DRIVER

```c
current_state = 0;
previously_seen_accepting_state = none_seen;

if( lookahead character is end-of-input )
  return 0;

while( lookahead character is not end-of-input )
{
  if( there is a transition from the current state on the current lookahead character)
  {
    current_state = that state;
    advance the input;
    if( the current state is an accepting state )
    {
        remember the current position in the input
        and the action associated with the current state;
    }
  }
else{
  if( no accepting state has been seen )
  {
      There's an error:
          Discard the current lexeme and input character.
          Current_state = 0;
  }
  else
  {
    back up the input to the position it was in when it saw the last accepting state
    perform the action associated with that accepting state;
  }
 }
}
```
- *Note that the greedy algorithm does have its disadvantages: It's tricky to implement and tends to be relatively slow. It can also cause the recognizer to behave in sometimes unexpected ways. (The LEX input expression ( \n 1 . ) * tries to absorb the entire input file, for example.) It is nonetheless the best (and sometimes the only) choice in most real
lexical-analysis applications.*

- The second type of algorithm (the nongreedy algorithm) is much simpler. Here, the shortest possible input string is recognized, and the machine just accepts as soon as an accepting state is entered. A nongreedy recognizer program is much simpler to implement than a greedy one, and is much faster as well. Nonetheless, this algorithm can be used only when all the accepting states in the machine are terminal nodes-when they have no outgoing transitions.
