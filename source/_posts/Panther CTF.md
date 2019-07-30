---
title: Panther ctf
date: 2018-10-08 22:35:16
---


# 前言

第一次写writeup开始记录在我的博客
这次没有做一道题就写writeup所以有些题忘了，
下次比赛一定做一道写一道writeup



### misc
#### 你会解压吗

[flag99.zip](https://pan.baidu.com/s/14LHEPaYgXaq0XKf7k0aO8g)
```shell
#!/bin/bash

for i in {1..100}
do
x=$((100-$i))
unzip ./flag$x.zip
done
```
#### zzzzip?
[flag.zip](https://pan.baidu.com/s/14LHEPaYgXaq0XKf7k0aO8g)


用ziperello爆破出5位的密码
然后用十六进制编辑器发现flag

#### So many flag!
[lag .txt](https://pan.baidu.com/s/14LHEPaYgXaq0XKf7k0aO8g)

```python
import re

file=open("E:\\flag.txt",mode="r")

flag=file.read()
flag1=re.split(r"}pctf{",flag)
for i in flag1:
    # print(i)
    if len(i)==20:
        a=re.match(r"^[a-z][A-Z][0-9].+[a-z][A-Z][a-z]$",i)
        print(a)
        print(i)

# print(flag1)
file.close()
```

#### 出题人的心思
![Alt text](./nick.png)
题目是一张jpg图片用binwalk打开发现还有一张png图片
然后用kali自带的分离工具将图片分开，再用十六进制改变png图片的高度，看到flag

### crypto


#### just so so!

    题目描述:
        密文：706374667b686868685f546831735f4833785f636f64657d
    答案格式: pctf{xxxxxxxxxxxxx}


```python
import re
ab="70,63,74,66,7b,68,68,68,68,5f,54,68,31,73,5f,48,33,78,5f,63,6f,64,65,7d"
ac=re.split(",","70,63,74,66,7b,68,68,68,68,5f,54,68,31,73,5f,48,33,78,5f,63,6f,64,65,7d")

print(ac)
for i in ac:

    print(chr(eval(i)))


```
pctf{hhhh_Th1S_H3x_code}

#### 被加密了的flag


    题目描述:
        cpgs{Gu1f_1F_Ebg_1r}
    答案格式: pctf{xxxxxxxxxxxxx}


忘记细节了，思路是26个字母表中间对折然后替换相应的字母
就转换出pctf了

#### My math is very poor


题目描述:
```python
#!/usr/bin/env python
# -*- coding:utf8 -*-
data = 'abcdefghijklmnopqrstuvwxyz'
flag = [] #[x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x]
a = 9
b = 13
assert len(flag) == 17
def encrypt():
    print 'Encrypt : ',
    enc = ''
    enc_array = []
    for i in range(0,len(flag)):
        tmp=(a*flag[i]+b)%26
        enc_array.append(tmp)
        enc += data[tmp]
    print enc # yyynvjlpjccyxginp
    print enc_array 
|# [24,24,24,13,21,9,11,15,9,2,2,24,23,6,8,13,15]

if __name__ == '__main__':
    encrypt()
```
- 答案格式: pctf{xxxxxxxxxxxxx}

```python
 -*- coding: utf-8 -*-

from string import ascii_lowercase as lowercase
from string import ascii_uppercase as uppercase

frequencyTable = [4, 19, 14, 0, 13, 8, 17, 18, 7, 3, \
                  11, 2, 20, 12, 15, 24, 22, 6, 1, 21, \
                  10, 23, 9, 16, 25]


# 删除预留的标点

# 文本过滤
def text_filter(text):
    text = text.lower()
    result = ""
    for i in range(len(text)):
        if lowercase.find(text[i]) != -1:
            result += text[i]
    return result


# 加密部分
def encryption(plaintext, k1, k2):
    plaintext = text_filter(plaintext)
    result = ""
    for i in range(len(plaintext)):
        index = lowercase.find(plaintext[i])
        c_index = (k1 * index + k2) % 26
        result += uppercase[c_index]
    return result


# 解密部分
def get_inverse(a, b):
    """
    #求a关于模b的逆元
    """
    if (a == 1 and b == 0):
        x = 1
        y = 0
        return x, y
    else:
        xx, yy = get_inverse(b, a % b)
        x = yy
        y = xx - a // b * yy
        return x, y


def Decryption(k1, k2, ciphertext):
    k3 = get_inverse(k1, 26)[0]
    result = ""
    for i in range(len(ciphertext)):
        index = lowercase.find(ciphertext[i])
        p_index = k3 * (index - k2) % 26
        result += lowercase[p_index]
    return result




if __name__ == '__main__':
    ciphertext = "yyynvjlpjccyxginp"
    print(Decryption(9,13,ciphertext))


```
[这个博客]:https://blog.csdn.net/sinat_34927324/article/details/79700703
仿射密码看[这个博客]的

其他题我也忘了。。。


### reverse
#### 软件破解第一步
[Reverse11.exe](https://pan.baidu.com/s/14LHEPaYgXaq0XKf7k0aO8g)

用ida打开就有flag了嘿嘿

#### come on! 
[flag_file.pyo](https://pan.baidu.com/s/14LHEPaYgXaq0XKf7k0aO8g)

这是一个Python的编译后加了防反编译模块
用 easy Python Decompiler这个工具解开就可以看到源码

```python
# Embedded file name: /home/sliver/Documents/CTF/Python/python.py
import base64
import py_compile
py_compile.compile('/home/sliver/Documents/CTF/Python/python.py')

def encode(message):
    s = ''
    for i in message:
        x = ord(i)
        x = x - 16
        s += chr(x)

    return base64.b64encode(s)


correct = 'RFhZJU8hY09AaU9DZlJaVVNk'
flag = ''
print 'Input flag:'
flag = raw_input()
if encode(flag) == correct:
    print 'correct,this is my flag'
else:
    print 'wrong'
```
然后用

```python
s="p"
for i in "RFhZJU8hY09AaU9DZlJaVVNk":
    x=ord(i)
    x=x+16
    s+=chr(x)
print(s)
```

就可见flag


#### input flag
[Reverse3.exe](https://pan.baidu.com/s/14LHEPaYgXaq0XKf7k0aO8g)
用ida打开f5大法 

发现DXY%O!cO@iOCfRZUSd 对flag进行处理后与这个字符串进行比较
```python
s="p"
for i in "DXY%O!cO@iOCfRZUSd":
    x=ord(i)
    x=x+16
    s+=chr(x)
print(s)
```
可解出flag

### web
#### 为什么这么简单

    答案格式: pctf{xxxxxxxxxxxxx}
    题目链接: 题目链接
[题目链接]http://47.94.4.84/web/web1abf20c91a442da48.php


用hackbar  post一个比66666大的数 （最快的方法了）

#### i'm so sad
http://101.200.58.30/test1.php

base64对字符串解码可看到代码
发送get请求a[]=1$b[]=2 得到flag

#### php is best language!

http://47.94.4.84/web/web25d47c5d8a6299792.php

用hackbar post   
magic_keys[]=1可得flag

pctf结束









