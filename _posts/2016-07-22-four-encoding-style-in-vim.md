---
layout: post
title: Vim中的四种编码设置
date: 2016-07-22
categories: Linux
tags: Linux Vim
---


## 0. 各种编码简介


各种语言编码格式 | 说明
---|---
GB2312 | GB2312是中国1981年公布的，包含7,445个字符，其中6,763个汉字和682个非汉字字符
euc-cn | GB2312的别名，全名为Extended Unix Code，相应的后面是国家的代码，比如说euc-jp表示是日文的编码
GBK | GB代表国家标准(Guojia biaozhun), K的意思是扩展（Kuozhan），扩展的是GB2312， 主要扩展是加入了繁体字，一些用GB2312不能表示的字，在GBK中可以表示了。GBK定义了23,940个符号，21,886个字符
cp936 | GBK的别名，Windows使用代码页(code page)来适应各个国家和地区。code page可以被理解为字符的编码。GBK对应的code page是cp936
GB18030 | GB18030还向后兼容了GB2312与GBK编码。GB18030是一种对字符集的多字节编码格式，相当于utf-8（对Unicode码点(code point)的编码传输格式），而且都是向后兼容ASCII，并且能表示所有的Unicode码点
BIG5 | BIG5主要是用来表示繁体字的。
Unicode | Unicode是一个很大的编码集合，可以编码世界上所有的符号，现在的规模可以容纳100多万个符号。
utf-8 | utf-8是在互联网上使用最广的一种Unicode的实现方式。
ucs-bom | UCS可以看作是"Unicode Character Set"的缩写，BOM表明编码方式。Unicode的学名是"Universal Multiple-Octet Coded Character Set"，简称为UCS。Unicode规范中推荐的标记字节顺序的方法是BOM(Byte Order Mark)；
latin1 | 单字节编码，向下兼容ASCII，其编码范围是0x00-0xFF，0x00-0x7F之间完全和ASCII一致；latin1是一种非常宽松的编码方式，任何一种编码方式得到的文本，用latin1进行解码，都不会发生解码失败——当然，解码得到的结果自然也就是理所当然的“乱码”。


## 1. 对于vim来说

### 1.0 内容概述

Vim中有四种设置编码的变量需要注意，介绍如下：

变量名称 | 说明
---|---
encoding | 指定Vim内部使用的编码，如果没有特别的理由，请始终将encoding设置为utf-8
fileencodings | Vim按顺序使用fileencodings中的编码对打开的文件尝试解码，如果成功的话，就使用该编码方式进行解码；因此需要注意顺序，推荐设置为set fileencodings=utf-8,gb2312,gb18030,gbk,ucs-bom,latin1
fileencoding | 打开文件的编码，不设置好像也没有影响，或者设置为个人常用的字符编码，如utf-8或者gbk
termencoding | Vim用于屏幕显示的编码，如果在windows下使用工具连接远程服务器，需要设置工具的编码与此一致，确保不乱码

### 1.1 encoding

> Vim内部使用的字符编码方式，包括Vim的缓冲区、菜单文件、消息文本等；

举例说明：

假设在.vimrc为如下设置，而使用Vim编辑的文件的编码格式为cp936：

```
set encoding = utf-8
```
vim将做如下操作：

- 将读入的文件转换成utf-8编码（Vim能读懂的方式）；
- 但用户写入文件时，Vim将utf-8编码转换成cp936的格式；

> 由于encoding选项涉及到Vim中所有字符的内部表示，因此只能在Vim启动的时候设置一次。在Vim工作过程中修改encoding会造成非常多的问题。如果没有特别的理由，请始终将encoding设置为utf-8。为了避免在非UTF-8的系统如Windows下，菜单和系统提示出现乱码，可同时做这几项设置：

```
set encoding=utf-8
set langmenu=zh_CN.UTF-8
language message zh_CN.UTF-8
```


### 1.2 fileencoding

> 当Vim从磁盘上读取文件的时候，会对文件的编码进行探测。如果文件的编码方式和Vim的内部编码方式（encoding指定）不同，Vim就会对编码进行转换。转换完毕后，Vim会将fileencoding选项设置为文件的编码。当Vim存盘的时候，如果encoding和 fileencoding不一样，Vim就会进行编码转换。fileencoding是在打开文件的时候，由Vim进行（根据fileencodings）探测后自动设置的。
 
 注：
 
 - 此处有疑，设置与否好像并没有什么影响，不论是新建一个文件还是打开旧的文件，主要还是fileencodings的编码顺序来决定的；


### 1.3 fileencodings

Vim自动探测fileencoding的顺序列表，当我们打开文件的时候，Vim按顺序使用fileencodings中的编码进行尝试解码，如果成功的话，就使用该编码方式进行解码，并将fileencoding设置为这个值，如果失败的话，就继续试验下一个编码。因此最好将Unicode 编码方式放到这个列表的最前面，将拉丁语系编码方式latin1放到最后面。 


### 1.4 termencoding

> termencoding是Vim用于屏幕显示的编码，在显示的时候，Vim会把内部编码转换为屏幕编码，再用于输出。内部编码中含有无法转换为屏幕编码的字符时，该字符会变成问号，但不会影响对它的编辑操作。如果termencoding没有设置，则直接使用encoding不进行转换。

> 举个例子，当你在Windows下通过telnet登录Linux工作站时，由于Windows的telnet是GBK编码的，而Linux下使用UTF-8编码，你在telnet下的Vim中就会乱码。此时有两种消除乱码的方式：一是把Vim的encoding改为GBK，另一种方法是保持encoding为UTF-8，把termencoding改为GBK，让Vim在显示的时候转码。显然，使用前一种方法时，如果遇到编辑的文件中含有GBK无法表示的字符时，这些字符就会丢失。但如果使用后一种方法，虽然由于终端所限，这些字符无法显示，但在编辑过程中这些字符是不会丢失的。


### 1.5 举例说明

由于Unicode能够包含几乎所有的语言的字符，而且Unicode的UTF-8编码方式又是非常具有性价比的编码方式(空间消耗比UCS-2小)，因此建议encoding的值设置为utf-8。这么做的另一个理由是encoding设置为utf-8时，Vim自动探测文件的编码方式会更准确(或许这个理由才是主要的。我们在中文Windows里编辑的文件，为了兼顾与其他软件的兼容性，文件编码还是设置为GB2312/GBK比较合适，因此fileencoding建议设置为chinese(chinese是个别名，在Unix 里表示gb2312，在Windows里表示cp936，也就是GBK的代码页)。 

最后以自己的配置举例：

```
// 注：在windows下使用xshell连接远程终端，设置的终端编码为utf-8
set fileencodings=gbk,gb2312,gb18030,utf-8,ucs-bom,cp936,latin1
// 这里是否设置好像没有影响；
set fileencoding=gbk
set encoding=utf-8
set termencoding=utf-8
```

1. vim启动，根据.vimrc中设置的encoding的值来设置buffer、菜单文本、消息文的字符编码方式。
    - vim内部编码为utf-8；
2. 读取需要编辑的文件，根据fileencodings中列出的字符编码方式逐一探测该文件编码方式。并设置fileencoding为探测到的，看起来是正确的字符编码方式。
    - 如果touch一个新文件则是gbk编码；
    - 旧文件依旧保持原来的编码，会在fileencodings中探测到旧文件的编码，并进行第3步的工作；
3. 对比fileencoding和encoding的值，若不同则调用iconv将文件内容转换为encoding所描述的字符编码方式，并且把转换后的内容放到为此文件开辟的buffer里，此时我们就可以开始编辑这个文件了。
4. 编辑完成后保存文件时，再次对比fileencoding和encoding的值。若不同，再次调用iconv将即将保存的buffer中的文本转换为fileencoding所描述的字符编码方式，并保存到指定的文件中。
5. 本地的屏幕上显示终端内容，根据termencoding=utf-8进行解码，本机设置的终端编码为utf-8，两者一致，因此不会再本地看到乱码显示；



## 2. 参考文献


- [VIM文件编码识别与乱码处理](http://edyfox.codecarver.org/html/vim_fileencodings_detection.html)
- [linux下vim中文乱码的解决方法](http://www.cnblogs.com/joeyupdo/archive/2013/03/03/2941737.html)
- [vim字符编码设置](http://www.cnblogs.com/freewater/archive/2011/08/26/2154602.html)
- [GB2312/GBK/GB18030 和汉字的Unicode编码](http://www.bagualu.net/wordpress/archives/1805)
- [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
- [latin1](http://baike.baidu.com/view/1488384.htm)
- [谈谈Unicode编码，简要解释UCS、UTF、BMP、BOM等名词](https://linux.cn/thread-9833-1-1.html)


