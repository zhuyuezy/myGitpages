---
title: Python 3 读取文件错误：UnicodeDecodeError
date: 2019-11-14 16:11:11
categories: 
- python
tags: 
- error
- python
---

今天在Python 3中读取kaggle 2015提供的某.asm文件时，报错
`UnicodeDecodeError:'utf-8' codec can't decode byte 0xa0 in position 6605: invalid start byte.`

导致这个报错的原因是：该文件的编码方式为`ASCII text, with CRLF line terminators`，而不是utf-8。

解决方法多种：

1、改用Python 2，因为Python 2默认以字节流的方式读取文件;

2、基于与1同样的原因，可以在Python 3中以`'rb'`方式读取；

3、也可使用sublime等文本编辑软件，以`utf-8`编码重新存储该文件，再读取。

以上就是这个error的解决方法啦 ٩(ˊᗜˋ*)و

