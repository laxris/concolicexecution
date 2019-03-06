# Concolic Execution with Z3

For this task, we will apply concolic execution to the first RERS challange problem. In principle we are following the guide at 

> http://shell-storm.org/blog/Binary-analysis-Concolic-execution-with-Pin-and-z3/

with slight updates as explained below.

## Setting up
You can find a fully prepared virtual box with the correct environment for at

> URL

user "stre" has the password "CS4110". Alternatively, you can set up a Ubuntu 14.04/Linux with Kernel 3.13 and compile PIN-tools with gcc 4.4. Because new versions of PIN do not allow linking external libraries, we use the newest of the version 2, which in turn requires an older kernel. 

You can find the crackme example from the blog post in 

~/Downloads/pin-2.14-71313-gcc.4.4.7-linux/source/tools/ManualExamples 

```C++
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <fcntl.h>

char *serial = "\x30\x39\x3c\x21\x30";

int main(void)
{
  int fd, i = 0;
  char buf[260] = {0};
  char *r = buf;

  fd = open("serial.txt", O_RDONLY);
  read(fd, r, 256);
  close(fd);
  while (i < 5){
    if ((*r ^ (0x55)) != *serial)
      return 0;
    r++, serial++, i++;
  }
  if (!*r)
    printf("Good boy\n");
  return 0;
}
```

You can compile this normally, without any options

gcc crackme1.c -o crackme1

Next, we need to complile the PIN tool for concolic execution. We already prepared all the code and the depenencies for Z3 described in the blog post. 

The code, similar to the blog post is in ConcolicExecution.cpp. You can compile it via the PIN tool makefiles (make obj-intel64/filename.o) and manually add the libraries for Z3, or use the compile.sh shell script we provide

./compile.sh

to both compile and link the binary for concolic execution:

```bash
cat compile.sh 
g++ -DBIGARRAY_MULTIPLIER=1 -DUSING_XED -Wall -Werror -Wno-unknown-pragmas -fno-stack-protector -DTARGET_IA32E -DHOST_IA32E -fPIC -DTARGET_LINUX  -I../../../source/include/pin -I../../../source/include/pin/gen -I../../../extras/components/include -I./z3/src/api/c++ -I../../../extras/xed2-intel64/include -I../../../source/tools/InstLib -O3 -fomit-frame-pointer -fno-strict-aliasing    -c -o obj-intel64/ConcolicExecution.o ConcolicExecution.cpp

g++ -shared -Wl,--hash-style=sysv -Wl,-Bsymbolic -Wl,--version-script=../../../source/include/pin/pintool.ver    -o obj-intel64/ConcolicExecution.so obj-intel64/ConcolicExecution.o  -L../../../intel64/lib -L../../../intel64/lib-ext -L../../../intel64/runtime/glibc -L../../../extras/xed2-intel64/lib -lpin -lxed -ldwarf -lelf -ldl -lz3
```

To execute the crackme under concolic execution, simple run:

./run-crackme.sh ./crackme1

```bash
cat run-crackme.sh
#!/bin/bash

sudo ../../../pin.sh -t ./obj-intel64/ConcolicExecution.so -taint-file serial.txt -- $1
```

(the password for the user is CS4110). 

Upon each execution, PIN tool will execute the binary until it eaches a comparison (CMP) and builds the equation to fulfill the jump. It writes the sequence of discovered values to make each statement true into serial.txt.
Then can be restarted to execute the program again, reading and using the values found and stored in serial.txt until it hits the next condition.

## RERS Challange

To apply this to the RERS challange, we need to add a small modification: The problems read input from stdin, so we modify it to read the input from serial.txt:

```C++
...
    int fd, i = 0;
    char buf[260] = {0};
    char *r = buf;

    fd = open("serial.txt", O_RDONLY);
    read(fd, r, 256);
    close(fd);

    printf("starting now\n");

    // main i/o-loop
    while(i < 100)
...
```

Unfortunately, that is not enough: the way memory tainting is implemented, we do not catch all the comparision with the parameters/local variables in the *calculate_output* functions. Our work-around (without having to implement much more thorough tainting) is to make all comparisions with a global variable by making *input* global and then the functions from

>  void calculate_outputm22(int input)

to 

> void calculate_outputm22(int nonput)

essentially changing all comparisions to comparisons with comparisions with global variables. The file with all modifications is Problem1.c which again can be compiled normally via

gcc Problem1.c -o Problem1

and then be used for concolic execution. We implemented taining more registers in MyConcolicExecution.c, which can be built and used using the following two commands:

> ./mycomile.sh
> ./run.sh ./Problem1


