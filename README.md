# MakeFiles
A guide to makefiles in Linux 


## C/C++ Compilation
Eventually you will need to start making multiple code files
In something like VSCode we create a project and fill it with files
* VSCode would worry about the linking and building process itself
In linux we are not provided with the luxury that VScode gives us

C/C++ makes use of the idea of separate compilation
* each code file is compiled separately
* then an exe is built linking them together

Multiple code files is better for organization
Each class will require a .h library file and accompanying .cpp code file
* we need to link them together with the main .cpp file


## .h files
Collections of prototype information
.h files allow us to collect forward declarations together to keep code organized
.ccp files are *"code"* files - things that will be compiled
.h files are the *"library"* files that allow us to collect these prototypes to be used in further compilation

#include
When we #include something, we are inserting code at that point
* you can think of this as a giant macro
* a pre-processor step
* pulls declarations from the library into our code so the compilation knows "what is going on" 

Generally, .h files do not include variables or function definitions
* include forward declarations and constants

### Why? 
This is a pre-processor directive, so it is open to many problems
* just pulls open the code, would be difficult to trace... remember Macros
* we usually want vars and functions to be compiled so they can be error checked before including them

## Protecting .h files
.h files can open errors if not protected properly
When we #include, that code is inserted into the file without checks
* this can lead to situations where the contents of the .h file leads to errors

We can further use pre-process directives to protect ourselves from these issues
```h
#ifndef HEADER_H //name this the same as your file
#define HEADER_H
...header contents...
#endif
```

The #ifndef method defines a macro and everything after but once defined it will skip to the end

Another solution here can also include `#pragma once` achieving the same result
* this may not work with all compilers

## Compiling and Makefiles

To create an object file, we use the -c directive to the g++/gcc compiler
* g++ -c testFile.cpp
    * -> testFile.o
The .o file here is an object file

The -c option lets us compile and get around certain restrictions
* such as not requiring a main() function

The .o file is compiled code that doesn't have enough information to run by itself and would not be executable
* we can try and `chmod` but this would still not work

When we compile a .cpp file we don't compile everything
When creating a .h file it will be filled with forward declarations
* a .cpp file will be created to define all functions/classes declared

When we compile a .cpp file and we #include a .h file
* the compiler does NOT compile the associated .cpp file
* it only checks the .h file to make sure we are using it correctly



The -c option is useful, but limited... it assumes the .cpp file is standalone
To do this we use a Makefile
* this consists of several entries
Each entry has a:
1. Target (usually a file)
2. Dependencies (files which the target depends on)
3. Commands (based on the target and Dependencies) 
 
```Makefile
circle: main.o circle.o
    g++ -g -Wall main.o circle.o -o circle

main.o:circle.h main.cpp
    g++ -g -Wall -c main.cpp

circle.o circle.h circle.cpp
    g++ -g -Wall -c circle.cpp

clean: 
    rm *.o circle *~ -v

```

We usually want the first directive to be our executable file

## Makefile Linking
We create a .o file for each item that belongs in the overall code project
Each .o will be a target in the Makefile
Then we have an executable target

## Makefile Macros
Once your file has been written you can use the make command
When you type a make, and there is a file names makefile or Makefile it will run the commands from the first target of that file
You can also type make <target> to make a specific target
* NOTE: Targets are essentially labels
* The commands that get executed can be anything
* It's common to add a **clean** target to remove excessive files generated

Below is a general macro setup

```Makefile
OBJS = <.o files> 
CC = g++ #compiler
DEBUG = -g
CFLAGS = -Wall -c $(DEBUG)
LFLAGS = -Wall $(DEBUG)

<exe>: $(OBJS)
    $(CC) $(LFLAGS) $(OBJS) -o <exe>

clean:
    \rm *.o *~ <exe>

tar:
    tar -cvf <code files>

```

Why use Macros?
1. We can use this to customize compilation quicker and easier

Example. 
Difficult time with debugging code, we can use this to debug with even more information to locate where our issues are

2. This is also reusable, most macros stay in place, just change directives and small other definitions

## Making it All
After a clean, all is the next most common target
* All is usually used as a command in the makefile to call the other targets

`all:<target list>`
* All is frequently used when our Makefile is responsible for building multiple executables


# Sample Makefile

```Makefile
OBJS = MovieList.o Movie.o NameList.o Name.o Iterator.o
CC = g++
DEBUG = -g
CFLAGS = -Wall -c $(DEBUG)
LFLAGS = -Wall $(DEBUG)

p1 : $(OBJS)
    $(CC) $(LFLAGS) $(OBJS) -o p1

MovieList.o : MovieList.h MovieList.cpp Movie.h NameList.h Name.h Iterator.h
    $(CC) $(CFLAGS) MovieList.cpp

Movie.o : Movie.h Movie.cpp NameList.h Name.h
    $(CC) $(CFLAGS) Movie.cpp

NameList.o : NameList.h NameList.cpp Name.h 
    $(CC) $(CFLAGS) NameList.cpp

Name.o : Name.h Name.cpp 
    $(CC) $(CFLAGS) Name.cpp

Iterator.o : Iterator.h Iterator.cpp MovieList.h
    $(CC) $(CFLAGS) Iterator.cpp

clean:
    \rm *.o *~ p1

tar:
    tar cfv p1.tar Movie.h Movie.cpp Name.h Name.cpp NameList.h \
            NameList.cpp  Iterator.cpp Iterator.h
```



