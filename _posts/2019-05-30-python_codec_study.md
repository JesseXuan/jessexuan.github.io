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

   一个字符用一个字节表示，共256个字符，不包含中文字符
   
   （一个字符是指，如“abc"共三个字符即一个字母一个字符，“中文”表示两个字符）

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
   如：test.py是utf8编码格式保存的，文件开头声明却用了#coding:gbk，则加载运行时会报错，因为以
   GBK无法decode出utf8编码格式的字符，此时文件开头应该同样声明#coding:utf8则不会报错。

Python2.7字符类型                              {#Py27}
------------------------
1. str(实则是字节码，字节串bytes)

   type 'str';可能是ascii,utf8,gbk等编码格式的字符串；此时只能decode
   
2. unicode(非str类型)

   type 'unicode';必须以u开头声明指定是unicode编码格式字符串，如u'xxxxx'；此时只能encode

Python3字符类型                              {#Py3}
------------------------
1. unicode(即是str)

   class 'str'即是unicode。python3里变量赋值等于字符串时，无需加u开头指定；此时只能encode成字节串bytes

2. bytes(是bytes)

   class 'bytes'只能由str.encode('编码格式')得到;直接打印时，内存数据是什么就打印什么

编码encode&解码decode                              {#Encode}
------------------------
1. python2.7时，对unicode类型指定编码格式encode得到str类型的字节串；
   python3时，对unicode类型指定编码格式encode得到bytes类型的字节串；

2. python2.7时，对str类型指定编码格式decode得到unicode类型的字符串；
   python3时，对bytes类型指定编码格式decode得到unicode类型的字符串；

3. encode的非法使用，就是对已经是str类型字节串进行encode，而由于encode的前提需要原类型是unicode类型，
   所以此时解释器会使用系统默认编码（如‘ascii’）先decode成unicode类型，接着再根据给定编码格式encode。

4. decode的非法使用，就是对已经是unicode类型字符串进行decode，解释器会使用系统默认编码先encode成str
   类型，再根据给定编码格式进行decode。

字符类型工厂转换方法                              {#Fact}
------------------------
python2.7工厂转换方法都是以ASCII编码为基础的：

1. str(x),转换返回str字符串对象

   str(x)其实就是x.encode('ascii')，x是unicode类实例；

   如x=u'溪',此时str(x)就相当于x.encode('ascii')，那就会报错，因为ascii编码格式不包含中文字符；
   所以这种情况就不能用工厂转换方法了，只能用x.encode('utf8')或者x.encode('gbk')转换编码类型。

2. unicode(x),转换返回unicode字符串对象

   unicode(x)相当于x.decode('ascii')，x是str类型字节码字符串；

   如x='溪'，此时unicode(x)就相当于x.decode('ascii')，那就会报错，原因同上。

Python乱码问题                          {#Error}
------------------------
所有出现乱码的原因都可以归结为字符经过不同编码解码在编码的过程中使用的编码格式不一致造成：
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
test.py

```bash
#!/usr/bin/env python2
#_*_ coding:utf-8 _*_

print "------"
x = 'jesse'
y = u'溪‘

```

1. 启动python解释器
2. python解释器读取py脚本到内存中(从硬盘中读取py脚本的内容)
   
   这时候python解释器读取py脚本的前两行内容，查看是否有编码格式声明来决定以什么编码格式读入内存；

   如果没有在脚本文件开头声明指定信息，则使用解释器默认的py2 ascii或者py3 utf-8编码格式读入内存。

3. 执行内存中的代码(加载到内存的代码是unicode编码的二进制)，执行过程中可能会开辟新的内存空间来存放变量的内容

   内存的编码使用unicode，不代表内存中全都是unicode编码的二进制；

   在程序执行之前，内存中确实都是以unicode编码的二进制，如上面脚本读取到x = 'jesse'，其中x,=,
   引号等地位都一样，都是普通字符，都是以unicode编码的二进制形式存放在内存中的；
   
   但是程序在执行过程中，会申请新的内存空间来存放任意编码格式的数据（与程序代码字符实在的是两个内存空间），
   如x = 'jesse'会被解释器识别为字符串，会申请临时内存空间utf8编码的二进制，y=u'溪‘会临时申请内存空间unicode编码的二进制。

4. print打印到终端

   当程序执行时，如y=u'溪';print(y);执行这一步时是将y指向那块新的内存空间（非代码块实在的内存空间）中的内存，
   打印到终端，而终端是仍然运行于内存中的，这就相当于内存打印到内存，unicode->unicode；

   对于unicode格式的数据，无论在什么编码格式的终端内打印都不会出现乱码；
   
   而print(x)则会先按照终端的编码（这里假定是pycharm）执行x.decode('utf8')变成unicode，之后再打印出来；
   若此时是win7_CN CMD终端执行的则是先x.decode('gbk')，与文件开头指定的声明编码格式不一致，则会打印乱码了。

其它                              {#Other}
------------------------
1. 获取/修改系统默认编码

```bash
import sys
sys.getdefaultencoding()   #获取默认编码

reload(sys)
sys.setdefaultencoding('utf8')  #设置默认编码
```

2. 如unicode形式的字符串(实则是str类型的字节串)进行转换

```bash
string1 = 'id\pythonu003d215903184\u0026index\u003d0\u0026st\u003d52\u0026sid'
转成成真正的unicode类型字符串
string1.decode('unicode-escape')
```

3. 中文字符串处理

```bash
#coding:utf-8

str1 = '小萱萱'
str1[-2:] == '萱萱'
#返回False，因为str1是utf8编码格式的字节串而不是真正的字符串，utf8中一个汉字占3个字节，即‘萱萱’占了6个字节，
 而str1[-2:]是以unicode编码格式为单位的（即两个“双字节”）截取

str2 = u'小萱萱'
str2[-2:] == u'萱萱'
#返回True
```

4. 查看文件编码

```bash
import chardet

with open(filename, 'r') as f:
    data = f.read()
    return chardet.detect(data)
```

参考                              {#Ref}
------------------------
Python字符编码[https://www.cnblogs.com/zihe/p/6993891.html](https://www.cnblogs.com/zihe/p/6993891.html)

Python编码为什么那么蛋疼？[https://www.zhihu.com/question/31833164](https://www.zhihu.com/question/31833164)

python2.7中的字符编码问题[https://www.cnblogs.com/liaohuiqiang/p/7247393.html](https://www.cnblogs.com/liaohuiqiang/p/7247393.html)
