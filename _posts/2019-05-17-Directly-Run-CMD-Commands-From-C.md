---
Title: Run CMD commands from C program
date: 2019-05-17 4:23 PM
categories: [Technology]
tags: [programming, C]
---

I am sure 90% of the books in C programming wont have it but, you can actually run commands into the command prompt directly from your C code using the system() function.

An example code;
```C
#include <stdio.h>
void main(){
system("echo Hello");
}
```

will run the command "echo Hello" directly in the command prompt. Same goes for GNU/Linux distributions as well. I am not sure though if its something related to codeblocks itself or something added to the C standard later but it works.

So now, if you want to clear out the console, you can just system("cls"); and console will be cleared.
Just something I found out while tinkering around and thought it may be handy to you as well.