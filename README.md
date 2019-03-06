# Concolic Execution with Z3

For this task, we will apply concolic execution to the first RERS challange problem. In principle we are following the guide at 

> http://shell-storm.org/blog/Binary-analysis-Concolic-execution-with-Pin-and-z3/

with slight updates. Because newer versions of the PIN-tool do not work with external libraries anymore, we will use the latest version 2 that still allows to compile external libraries with it (2.14).

You can find a fully prepared virtual box with the correct environment for at

> URL

user "stre" has the password "CS4110". Alternatively, you can set up a Ubuntu 14.04/Linux with Kernel 3.13 and compile PIN-tools with gcc 4.4.

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
The code, similar to the blog post is in ConcolicExecution.cpp. You can compile it via the PIN tool makefiles (make obj-intel64/filename.o), or use the compile.sh shell script we provide

./compile.sh

to both compile and link the binary for concolic execution:

```bash
cat compile.sh 
g++ -DBIGARRAY_MULTIPLIER=1 -DUSING_XED -Wall -Werror -Wno-unknown-pragmas -fno-stack-protector -DTARGET_IA32E -DHOST_IA32E -fPIC -DTARGET_LINUX  -I../../../source/include/pin -I../../../source/include/pin/gen -I../../../extras/components/include -I./z3/src/api/c++ -I../../../extras/xed2-intel64/include -I../../../source/tools/InstLib -O3 -fomit-frame-pointer -fno-strict-aliasing    -c -o obj-intel64/ConcolicExecution.o ConcolicExecution.cpp

g++ -shared -Wl,--hash-style=sysv -Wl,-Bsymbolic -Wl,--version-script=../../../source/include/pin/pintool.ver    -o obj-intel64/ConcolicExecution.so obj-intel64/ConcolicExecution.o  -L../../../intel64/lib -L../../../intel64/lib-ext -L../../../intel64/runtime/glibc -L../../../extras/xed2-intel64/lib -lpin -lxed -ldwarf -lelf -ldl -lz3
```


