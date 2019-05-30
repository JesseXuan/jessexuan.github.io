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

   2至4个字节，一般一个字符由两个字节表示，缺点占用空间大为了内存里速度快

3. Utf-8(可变字节长度的编码方式)

   1至4个字节，一个英文字母由一个字节表示，一般一个中文由三个字节表示，节省空间带宽

4. GBK(GB2312)

   2至4个字节，一般一个中文由两个字节表示

5. 其它编码与注意事项

   计算机内存中统一使用unicode编码，而保存到硬件为文件或者网络传输过程中一般使用Utf-8编码格式。
   python2.7解释器默认使用ascii编码去解释源文件，但如果源文件中出现了非ascii字符，而这时没有在
   文件开头声明指定编码格式就会报错。若声明utf-8告诉解释器用来读取文件代码内容，则源文件出现中
   文也不会出现报错。

   一般文件以什么编码保存的，就以什么编码方式打开。而文件编码保存时使用的编码方式一般是编辑器
   右下角提示的编码方式，而解码的时候是使用文档开头声明的编码方式，此两种编码方式不一致时容易
   出现乱码的情况。

Python2.7字符类型                              {#Py27}
------------------------
1. str(实则是字节码，字节串bytes)
2. unicode(非str类型)

Python3字符类型                              {#Py3}
------------------------
1. unicode(即是str)
2. bytes(是bytes)

Python乱码问题                          {#Error}
------------------------
常见编码错误的原因：
1. python解释器的默认编码
   
   + python2.7默认编码是ASCII
   + python3默认编码是utf8

2. python源文件编码格式

   + 需要自行在文件开头两行声明指定编码格式

3. 终端terminal使用的编码

   + pycharm默认utf8
   + win7_CN cmd默认GBK
   + linux terminal默认utf8

4. 操作系统的语言设置

Python程序的执行                              {#Prog}
------------------------
1. 启动python解释器
2. python解释器读取py脚本到内存中(从硬盘中读取py脚本的内容)
   
   这时候python解释器读取py脚本的前两行内容，查看是否有编码格式声明来决定以什么编码格式读入内存；

   如果没有在脚本文件开头声明指定信息，则使用解释器默认的py2 ascii或者py3 utf-8编码格式读入内存。


3. 执行内存中的代码(加载到内存的代码是unicode编码的二进制)，执行过程中可能会开辟新的内存空间来存放变量的内容

参考                              {#Ref}
------------------------
Python字符编码[https://www.cnblogs.com/zihe/p/6993891.html](https://www.cnblogs.com/zihe/p/6993891.html)

Python编码为什么那么蛋疼？[https://www.zhihu.com/question/31833164](https://www.zhihu.com/question/31833164)

python2.7中的字符编码问题[https://www.cnblogs.com/liaohuiqiang/p/7247393.html](https://www.cnblogs.com/liaohuiqiang/p/7247393.html)
