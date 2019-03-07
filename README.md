# Concolic Execution with Z3

For this task, we will apply concolic execution to the first RERS challange problem. In principle we are following the guide at 

> http://shell-storm.org/blog/Binary-analysis-Concolic-execution-with-Pin-and-z3/

with slight updates as explained below.

## Setting up
You can find a fully prepared virtual box with the correct environment at

> https://www.dropbox.com/s/b1pgxgla9cah6bp/old%20sytem.ova?dl=0

user "stre" has the password "CS4110". Alternatively, you can set up a Ubuntu 14.04/Linux with Kernel 3.13 and compile PIN-tools with gcc 4.4. Because new versions of PIN do not allow linking external libraries, we use the newest of the version 2, which in turn requires an older kernel. 

You can find the crackme example from the blog post in 

~/Downloads/pin-2.14-71313-gcc.4.4.7-linux/source/tools/Tasks 

```C++
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <fcntl.h>

int main(void)
{
  int   fd;
  char  buff[260] = {0};

  fd = open("serial.txt", O_RDONLY);
  read(fd, buff, 256);
  close(fd);

  if (buff[0] != 'a') return -1;
  if (buff[1] != 'b') return -1;
  if (buff[2] != 'c') return -1;

  printf("Good boy\n");

  return 0;
}
```

You can compile this normally, without any options

gcc easy.c -o easy

Next, we need to complile the PIN tool for concolic execution. We already prepared all the code and the depenencies for Z3 described in the blog post. 

The code, similar to the blog post is in ConcolicExecution.cpp. You can compile it via the PIN tool makefiles (make obj-intel64/filename.o) and manually add the libraries for Z3, or use the compile.sh shell script we provide

./compile.sh

to both compile and link the binary for concolic execution:

```bash
g++ -DBIGARRAY_MULTIPLIER=1 -Wall -Werror -Wno-unknown-pragmas -fno-stack-protector -DTARGET_IA32E -DHOST_IA32E -fPIC -DTARGET_LINUX  -I../../../source/include/pin -I../../../source/include/pin/gen -I../../../extras/components/include -I../../../extras/xed-intel64/include -I../../../source/tools/InstLib -O3 -fomit-frame-pointer -fno-strict-aliasing   -c -o obj-intel64/ConcolicExecution.o ConcolicExecution.cpp

g++ -shared -Wl,--hash-style=sysv -Wl,-Bsymbolic -Wl,--version-script=../../../source/include/pin/pintool.ver    -o obj-intel64/ConcolicExecution.so obj-intel64/ConcolicExecution.o  -L../../../intel64/lib -L../../../intel64/lib-ext -L../../../intel64/runtime/glibc -L../../../extras/xed-intel64/lib -lpin -lxed -lpindwarf -ldl -lz3
```

To execute the crackme under concolic execution, simple run:

./run.sh ./easy

```bash
sudo ../../../pin.sh -t ./obj-intel64/MyConcolicExecution.so -taint-file serial.txt -- $1
```

(the sudo password for the user is CS4110). 

Upon each execution, PIN tool will execute the binary until it eaches a comparison (CMP) and builds the equation to fulfill the jump. It writes the sequence of discovered values to make each statement true into serial.txt, assuming each check is done on a different variable.
TThe tool can be re-run and executes the program again, reading and using the values found and stored in serial.txt until. Each time, it will get past one more condition.

## The task

Compile task.c, outline the different control flow paths (list or visualization as a graph), and use the pin tool to provide 

* Inputs generated to explore the paths
* Path constraint constructed
* Code executed causing ithe path constraints

Compare the path constraint to the if-conditions in the code and explain the differences.



