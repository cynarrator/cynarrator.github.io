---
Title: The Shortcomings of using namespace std in C++
date: 2022-6-22 08:26 AM
categories: [Technology]
tags: [programming,C++]
---
## Introduction
You have to learn C++ in many Bachelors in Information Technology program as a part of your college syllabus, usually in second semester. Students are supposed to write all the C++ code in a paper with a pen, the less they write and cross, the more tidy their work becomes, resulting in better marks. So teachers teach students to ``using namespace std`` so that they do not forget to use ``std::`` when needed and its less things to write.

If your only purpose of practicing C++ is to pass exam than it is okay to do so. But in real world, this practice is discouraged. Try to compile the following seemingly innocent program:

```C++
#include <iostream>
using namespace std;
class data 
{
    int id;
    char* name;
public:
    data(int passed_id, char* passed_name)
    {
        this -> id = passed_id;
        this -> name = passed_name;
    }
};

int main()
{
    data my_record = data(20, "Sujan");
    return 0;
}
```

It will not compile and you will notice there is some mention of ambiguity. Its because ``data`` class is already defined in the ``std`` namespace and that causes conflict.
## Solution
Always use ``std::`` in front of statements where you use classes from the std namespace. I see alot of professional C++ developers doing it

## But wait, there is a better way
You are discouraged from using the entire std namespace in your program, however, you can do this

```C++
using std::cout;
using std::cin;
using std::string;
using std::endl;
```