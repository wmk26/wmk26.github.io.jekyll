---
layout: post
title: c++结构体string赋值问题
date: 2017-09-18
categories: c++
tags: c++
---

## 0. 结论

STL中的string在使用之前需要调用默认的构造函数进行初始化，之后才能执行赋值、打印等操作，如果使用malloc或者memset给结构体分配内存，就不会调用string默认的构造函数来初始化结构体中的string，因此如果直接赋值就会发生错误；

注：string是变长的，因此也不适合用malloc或者memset进行清空；同时在结构体里面也不建议定义变长的数据类型；

## 1. 问题描述

源代码

```
#include <string>
#include <iostream>
#include <vector>
#include <algorithm>
#include <sstream>

struct KeywordStruct
{
    std::string keyword;
    uint32_t weight;
};

int main()
{
    std::vector<KeywordStruct> ks_vec;
    KeywordStruct ks;
    for (int i = 0; i < 5; ++i)
    {
        memset(&ks, 0, sizeof(KeywordStruct));
        std::stringstream ss;
        ss << i;
        ks.keyword = "keyword" + ss.str();
        ks.weight = i + 1.0;
        ks_vec.push_back(ks);
    }

    return 0;
}

```

运行上述程序，会core dump，使用gdb调试，错误信息如下：

```
Program terminated with signal 11, Segmentation fault.
#0  0x00000036c56b76d2 in __gnu_cxx::__exchange_and_add(int volatile*, int) () from /usr/lib64/libstdc++.so.6
(gdb) where
#0  0x00000036c56b76d2 in __gnu_cxx::__exchange_and_add(int volatile*, int) () from /usr/lib64/libstdc++.so.6
#1  0x00000036c569caab in std::basic_string<char, std::char_traits<char>, std::allocator<char> >::assign(std::basic_string<char, std::char_traits<char>, std::allocator<char> > const&) ()
from /usr/lib64/libstdc++.so.6
#2  0x0000000000400fe8 in main () at src/main.cpp:22
```

解决：

```
#include <string>
#include <iostream>
#include <vector>
#include <algorithm>
#include <sstream>

struct KeywordStruct
{
    std::string keyword;
    uint32_t weight;
};

int main()
{
    std::vector<KeywordStruct> ks_vec;
    // KeywordStruct ks;
    for (int i = 0; i < 5; ++i)
    {
        // memset(&ks, 0, sizeof(KeywordStruct));
        KeywordStuct *ks_ptr = new KeywordStruct;
        std::stringstream ss;
        ss << i;
        ks_ptr->keyword = "keyword" + ss.str();
        ks_ptr->weight = i + 1.0;
        ks_vec.push_back(*ks_ptr);
        delete ks_ptr;
    }

    return 0;
}
```

## 2. 相关知识点

### 2.1 malloc && new

malloc只是分配内存。new除了分配内存还会调用构造函数。

具体可以参考这篇文章，写的很好：[细说new与malloc的10点区别](http://www.cnblogs.com/QG-whz/p/5140930.html)

### 2.2 memset

```
void *memset(void *s, int ch, size_t n);
```

函数解释：将s中前n个字节用ch(函数定义是int类型，实际上会转换成char类型进行替换)替换并返回s 。

memset：作用是在一段内存块中填充某个给定的值，它是对较大的结构体或数组进行清零操作的一种最快方法


## 参考文献：

- [C++ string中的几个小陷阱，你掉进过吗？](http://www.cnblogs.com/lanxuezaipiao/p/3704578.html)(主要参考文献)
- [C++中memset的用法](http://liaolushen.github.io/2015/10/21/C-%E4%B8%ADmemset%E7%9A%84%E7%94%A8%E6%B3%95/)
- [How to use a C++ string in a structure when malloc()-ing the same structure?](https://stackoverflow.com/questions/3411815/how-to-use-a-c-string-in-a-structure-when-malloc-ing-the-same-structure)
- [c++新手容易犯的几个错误](http://www.cnweblog.com/fly2700/archive/2011/10/11/318081.html)
- [memset](https://baike.baidu.com/item/memset)
- [结构体中定义string成员变量](http://jimmyleeee.blog.163.com/blog/static/930961820087255453299/)(GDB分析)
- [In what cases do I use malloc vs new?](https://stackoverflow.com/questions/184537/in-what-cases-do-i-use-malloc-vs-new)
- [细说new与malloc的10点区别](http://www.cnblogs.com/QG-whz/p/5140930.html)
