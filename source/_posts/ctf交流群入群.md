---
title: ctf交流群的加群题
---
### misc
[题目下载](https://pan.baidu.com/s/14YsPdaY5J7iM05Yn11Q8rQ)
Tips：
https://processor.pub/2017/03/22/0CTF-2017-python%E9%80%86%E5%90%91/

题目中有一个pyc文件
先用Easy Python Decompiler将文件反编译

```python
# Embedded file name: test.py
import base64
part_1 = 'cmFnZVq'
part_2 = '95b3Vy'
part_3 = 'X2RyZWFt'
part_4 = 'ISEh'

def decode--- This code section failed: ---

0	 LOAD_GLOBAL              None
3	<153>             None
6	SLICE+2           None
7	<151>             None
10	BINARY_ADD        None
11	<151>             None
14	BINARY_ADD        None
15	<151>             None
18	<153>             None
21	BINARY_MULTIPLY   None
22	BINARY_ADD        None
23	STORE_FAST        'secret'

26	<151>             None
29	LOAD_ATTR         'b64decode'
32	LOAD_FAST         'secret'
35	CALL_FUNCTION_1   None
38	STORE_FAST        'text'

41	LOAD_FAST         'text'
44	PRINT_ITEM        None
45	PRINT_NEWLINE_CONT None
46	<153>             None
49	RETURN_VALUE      None
-1	RETURN_LAST       None

Syntax error at or near `<151>' token at offset 0
```
可见有两个指令是不能识别
用python中的dis，和marshal 进行调试
发现文件的前四位是magic number，用来识别python版本，接着的四位为时间戳
```python
>>> import dis,marshal
>>> f=open("/opt/crypt.pyc")
>>> f.read(4)
'\x03\xf3\r\n'
>>> f.read(4)
'\xf6\xa1$['
```
接下来是读取opcode，发现opcode逻辑不对，所以修改opcode来还原函数算法。
```python
>>> code = marshal.load(f)
>>> code.co_consts
(-1, None, 'cmFnZVq', '95b3Vy', 'X2RyZWFt', 'ISEh', <code object decode at 0x7fb03916ca30, file "test.py", line 14>)
>>> dec = code.co_consts[6]
>>> dis.dis(dec.co_code)
          0 <151>               0
          3 <153>               1
          6 SLICE+2
          7 <151>               1
         10 BINARY_ADD
         11 <151>               2
         14 BINARY_ADD
         15 <151>               3
         18 <153>               2
         21 BINARY_MULTIPLY
         22 BINARY_ADD
         23 STORE_FAST          0 (0)
         26 <151>               4
         29 LOAD_ATTR           5 (5)
         32 LOAD_FAST           0 (0)
         35 CALL_FUNCTION       1
         38 STORE_FAST          1 (1)
         41 LOAD_FAST           1 (1)
         44 PRINT_ITEM
         45 PRINT_NEWLINE
         46 <153>               0
         49 RETURN_VALUE
```
然后查看有关函数的信息
```python
>>> dec.co_names
('part_1', 'part_2', 'part_3', 'part_4', 'base64', 'b64decode')
>>> dec.co_varnames
('secret', 'text')
>>> dec.co_consts
(None, -1, 2)
```
发现<151>这个指令使用的传入值0，1，2，3，4。而part的几个值是在函数的外面，猜想是LOAD_GLOBAL这个指令，<151>这个指令使用传入的值0，1，2. 调用的是varnames的值。猜想是LOAD_CONST指令，
参考：https://github.com/python/cpython/blob/master/Include/opcode.h
https://docs.python.org/2/library/dis.html#opcode-BUILD_SET
读常量用的是“100”，而里面是“153”，读取全局变量是“116”
使用：
```python
>>> dis.dis(dec.co_code.replace("\x99","\x64").replace("\x97","\x74"))
          0 LOAD_GLOBAL         0 (0)
          3 LOAD_CONST          1 (1)
          6 SLICE+2
          7 LOAD_GLOBAL         1 (1)
         10 BINARY_ADD
         11 LOAD_GLOBAL         2 (2)
         14 BINARY_ADD
         15 LOAD_GLOBAL         3 (3)
         18 LOAD_CONST          2 (2)
         21 BINARY_MULTIPLY
         22 BINARY_ADD
         23 STORE_FAST          0 (0)
         26 LOAD_GLOBAL         4 (4)
         29 LOAD_ATTR           5 (5)
         32 LOAD_FAST           0 (0)
         35 CALL_FUNCTION       1
         38 STORE_FAST          1 (1)
         41 LOAD_FAST           1 (1)
         44 PRINT_ITEM
         45 PRINT_NEWLINE
         46 LOAD_CONST          0 (0)
         49 RETURN_VALUE
```
参考：http://anhkgg.com/python-bytecode/
还原程序的加密部分：
```python
decode()
	secret=part_1[:-1]+part_2+part_3+part_4
	text=b64decode(secret)
	return test
```
payload.py
```python
import base64
part_1 = 'cmFnZVq'
part_2 = '95b3Vy'
part_3 = 'X2RyZWFt'
part_4 = 'ISEh'

secret=part_1[:-1]+part_2+part_3+part_4

text=base64.b64decode(secret)
print(text)
```
解密得到
rage_your_dream!!!

其他题期待更新。。。