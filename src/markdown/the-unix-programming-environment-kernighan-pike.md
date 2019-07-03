# The UNIX™ Programming Environment
## by Brian W. Kernighan & Rob Pike

## Contents

1. UNIX for Beginners
    - 1.1 Getting started
        - Some prerequisites about terminals
        - A Session with UNIX
        - Logging in
        - Typing Commands
        - Strange terminal behavior
        - Mistakes in typing
        - Type-ahead
        - Stopping a program
        - Logging out
        - Mail
        - Writing to other users
        - News
        - The manual
        - Computer-aided instruction
        - Games
    - 1.2 Day-to-day use: files and common commands
        - Creating files - the editor
        - What files are out there?
        - Printing files - `cat` and `pr`
        - Moving, copying, removing files - `mv`, `cp`, `rm`
        - What's in a filename?
        - A handful of useful commands
        - A summary of file system commands
    - 1.3 More about files: directories
        - Changing directory - `cd`
    - 1.4 The shell
        - Filename shorthand       
        - Input-output redirection 
        - Pipes                    
        - Processes                
        - Tailoring the environment
    - 1.5 The rest of the UNIX system
2. The File System
    - 2.1 The basics of files
    - 2.2 What's in a file
    - 2.3 Directories and filenames
    - 2.4 Permissions
    - 2.5 Inodes
    - 2.6 The directory hierarchy
    - 2.7 Devices
3. Using the Shell
    - 3.1 Command line structure
    - 3.2 Metacharacters
        - A digression on `echo`
    - 3.3 Creating new commands
    - 3.4 Command arguments and parameters
    - 3.5 Program output as arguments
    - 3.6 Shell variables
    - 3.7 More on I/O redirection
    - 3.8 Looping in shell programs
    - 3.9 `bundle`: Putting it All Together
    - 3.10 Why a Programmable Shell?
4. Filters
    - 4.1 The `grep` Family
    - 4.2 Other filters
    - 4.3 The stream editor `sed`
    - 4.4 The `awk` pattern scanning and processing language
        - Fields
        - Printing
        - Patterns
        - The `BEGIN` and `END` patterns
        - Arithmetic and Variables
        - Control flow
        - Arrays
        - Associative arrays
        - Strings
        - Interaction with the shell
        - A calendar service based on `awk`
        - Loose ends
    - 4.5 Good files and good filters
5. Shell Programming
    - 5.1 Customizing the `cal` command
    - 5.2 Which command is `which`?
    - 5.3 `while` and `until` loops: watching for things
    - 5.4 Traps: catching interrupts
    - 5.5 Replacing a file: `overwrite`
    - 5.6 `zap`: killing processes by name
    - 5.7 The `pick` command: blanks vs. arguments
    - 5.8 The `news` command: community service messages
    - 5.9 `get` and `put`: tracking file changes
    - 5.10 A look back
6. Programming with Standard I/O
    - 6.1 Standard input and output: `vis`
    - 6.2 Program arguments: `vis` version 2
    - 6.3 File access: `vis` version 3
    - 6.4 A screen-at-a-time printer: `p`
    - 6.5 An example: `pick`
    - 6.6 On bugs and debugging
    - 6.7 An example: `zap`
    - 6.8 An interactive file comparison program: `idiff`
    - 6.9 Accessing the environment
7. UNIX System Calls
    - 7.1 Low-level I/O
        - File descriptors
        - File I/O - `read` and `write`
        - File creation - `open`, `creat`, `close`, `unlink`
        - Error processing - `errno`
        - Random access - `lseek`
    - 7.2 File System: directories
    - 7.3 File System: inodes
        - `sv`: An illustration of error handling
    - 7.4 Processes
        - Low-level process creation - `execlp` and `execvp`
        - Control of processes - `fork` and `wait`
    - 7.5 Signals and Interrupts
        - Alarms
8. Program Development
    - 8.1 Stage 1: A four-function calculator
        - Grammars
        - Overview of `yacc`
        - Stage 1 program
        - Making changes - unary minus
        - A digression on `make`
    - 8.2 Stage 2: Variables and error recovery
    - 8.3 Stage 3: Arbitrary variable names; built-in functions
        - Another digression on `make`
        - A digression on `lex`
    - 8.4 Stage 4: Compilation into a machine
        - A third digression on `make`
    - 8.5 Stage 5: Control flow and relational operators
    - 8.6 Stage 6: Functions and procedures; input/output
    - 8.7 Performance evaluation
    - 8.8 A look back
9. Document Preparation
    - 9.1 The `ms` macro package
        - Displays
        - Font changes
        - Miscellaneous commands
        - The `mm` macro package
    - 9.2 The `troff` level
        - Character names
        - Font and size changes
        - Basic `troff` commands
        - Defining macros
    - 9.3 The `tbl` and `eqn` preprocessors
        - Tables
        - Mathematical expressions
        - Getting output
    - 9.4 The manual page
    - 9.5 Other document preparation tools
10. Epilog
    - Appendix 1: Editor Summary
    - Appendix 2: `hoc` Manual
    - Appendix 3: `hoc`

---

## Chapter 8: Program Development

The UNIX system was originally meant as a program development environment.
In this chapter we'll talk about some of the tools that are particularly suited for developing programs.
Our vehicle is a substantial program, an interpreter for a program language comparable in power to BASIC.
We chose to implement a language because it's representative of problems encountered in large programs.
Furthermore, many programs can profitably be viewed as languages that convert a systematic input into a sequence of actions and outputs, so we want to illustrate the language development tools.

In this chapter, we will cover specific lessons about
- `yacc`, a parser generator, a program that generates a parser from a grammatical description of a language.
- `make`, a program for specifying and controlling the processes by which a complicated program is compiled.
- `lex`, a program analogous to `yacc`, for making lexical analyzers.

We also want to convey some notions of how to go about such a project - the importance of starting with something small and letting it grow; language evolution; and the use of tools.

We will describe the implementation of the language in six stages, each of which would be useful even if the development went no further.
These stages closely parallel the way that we actually wrote the program.

1. A four function calculator, providing `+`, `-`, `*`, `/`, and parentheses, that operates on floating point numbers. One expression is typed on each line; its value is printed immediately.
2. Variables with names `a` through `z`. This version also has unary minus and some defenses against errors.
3. Arbitrarily-long variable names, built-in functions for `sin`, `exp`, etc., useful constants like `π` (spelled `PI` because of typographic limitations), and an exponentiation operator.
4. A change in internals; code is generated for each statement and subsequently interpreted, rather than being evaluated on the fly. No new features are added, but it leads to version 5.
5. Control flow: `if-else` and `while`, statement grouping with `{` and `}`, and relational operators like `>`, `<=`, etc.
6. Recursive functions and procedures, with arguments. We also added statements for input and output of strings as well as numbers.

The resulting language is described in Chapter 9, where it serves as the main example in our presentation of the UNIX document preparation software.
Appendix 2 is the reference manual.

This is a very long chapter, because there's a lot of detail involved in getting a non-trivial program written correctly, let alone presented.
We are assuming that you understand C, and that you have a copy of the *UNIX Programmer's Manual*, Volume 2, close at hand, since we simply don't have space to explain every nuance.
Hang in, and be prepared to read the chapter a couple of times.
We have also included all of the code for the final version in Appendix 3, so you can see more easily how the pieces fit together.

By the way, we wasted a lot of time debating names for this language, but never came up with anything satisfactory.
We settled on `hoc`, which stands for "high-order calculator."
The versions are thus `hoc1`, `hoc2`, etc.

### 8.1 Stage 1: A four-function calculator

This section describes the implementation of `hoc1`, a program that provides about the same capabilities as a minimal pocket calculater, and is substantially less portable.
It has only four functions: `+`, `-`, `*`, and `/`, but it does have parentheses that can be nested arbitrarily deeply, which few pocket calculators provide.
If you type an expression followed by RETURN, the answer will be printed on the next line:
```
$ hoc1
4*3*2
      24
(1+2) * (3+4)
      21
1/2
      0.5
355/113
      3.1415929
-3-4
hoc1: syntax error near line 4
$
```

#### Grammars

Ever since Backnus-Naur Form was developed for Algol, languages have been described by formal grammars.
The grammar for `hoc1` is small and simple in its abstract representation:
```
list:   expr \n
        list expr \n
expr:   NUMBER
        expr + expr
        expr - expr
        expr * expr
        expr / expr
        ( expr )
```
In other words, a `list` is a sequence of expressions, each followed by a newline.
An expression is a number, or a pair of expressions joined by an operator, or a parenthesized expression.

This is not complete.
Among other things, it does not specify the normal precedence and associativity of the operators, nor does it attach a meaning to any construct.
And although `list` is defined in terms of `expr`, and `expr` is defined in terms of `NUMBER`, `NUMBER` itself is nowhere defined.
These details have to be filled in to go from a sketch of the language to a working program.

#### Overview of `yacc`
#### Stage 1 program
#### Making changes - unary minus
#### A digression on `make`
### 8.2 Stage 2: Variables and error recovery
### 8.3 Stage 3: Arbitrary variable names; built-in functions
#### Another digression on `make`
#### A digression on `lex`
### 8.4 Stage 4: Compilation into a machine
#### A third digression on `make`
### 8.5 Stage 5: Control flow and relational operators
### 8.6 Stage 6: Functions and procedures; input/output
### 8.7 Performance evaluation
### 8.8 A look back
