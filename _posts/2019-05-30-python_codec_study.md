---
layout: post
title:  "Python编码问题"
date:   2019-05-30 11:31:01 +0800
categories: Documents
tag: 教程
---

* content
{:toc}


Python字符编码类型				{#Codec}
------------------------
1. ASCII

   一个字符用一个字节表示（一个字符是指，如“abc"共三个字符即一个字母一个字符，“中文”表示两个字符）

2. Unicode

   2至4个字节，一般一个字符由两个字节表示

3. Utf-8

   1至4个字节，一个英文字母由一个字节表示，一般一个中文由三个字节表示

4. GBK(GB2312)

   2至4个字节，一般一个中文由两个字节表示

Python2.7字符编码类型                              {#Py2.7}
------------------------
1. str
2. unicode

Python3字符类型                              {#Py3}
------------------------
1. unicode
2. bytes

Python乱码问题                          {#Error}
------------------------
常见编码错误的原因：
1. python解释器的默认编码
   
   python2.7默认编码是ASCII，python3默认编码是utf8

2. python源文件编码格式

   需要自行在文件开头两行声明指定编码格式

3. 终端terminal使用的编码

   + pycharm默认utf8
   + win7_CN cmd默认GBK
   + linux terminal默认utf8

4. 操作系统的语言设置

参考                              {#Ref}
------------------------
Python字符编码[https://www.cnblogs.com/zihe/p/6993891.html](https://www.cnblogs.com/zihe/p/6993891.html)

Python编码为什么那么蛋疼？[https://www.zhihu.com/question/31833164](https://www.zhihu.com/question/31833164)

python2.7中的字符编码问题[https://www.cnblogs.com/liaohuiqiang/p/7247393.html](https://www.cnblogs.com/liaohuiqiang/p/7247393.html)
