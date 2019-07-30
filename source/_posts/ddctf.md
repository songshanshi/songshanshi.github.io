---
title: ddctf解题记录
password: zero1234
---
### reverse 1
使用ollydbg调试
这个题就是字符串替换也可以写一个脚本在ascii码的十六进制直接做运算，我这里为了速度快直接输入存在的ascii字符的  一一对应
这道题需要配置的是DDCTF{reverseME}   找到对应的一一匹配就是flag
=<;:9876543210/.-,+*)('&%$]\[ZYXWVUTSRQPONMLKJIHGFEDCBA"
abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ[\]"

abcdefghijklmdcbpqrstyz#!B
=<;:987654321:;<.-,+*%${}\
### DDCTF逆向二

脱壳   首先文件有一层asp壳  由于只是业余逆向就不手脱了，用
ASPack Stripper这个工具

#### 总体分析
脱完壳拉进ollydbg进行调试
首先总结初步调试的
- 输入字符数是偶数
输入的字符57 - 65     >48     <70     
可以输入ABCDEF1234567890
- 最后将输入的值进行转换后与reverse+进行比较
- python模块itertools的排列组合函数模拟所有输入的所有情况

#### 输入测试
以下是几组测试输入的值转换后的变化情况

1234567890
EjRWeJA=

1234567890ABCDEF
EjrWeJCrze8=

FEDCBA0987654321
/ty6CYdlQyE=

#### 逆向分析
使用0x401240这个函数对输入过滤
![Alt text](./QQ截图20190416190048.png)

接着单步往下执行
发现输入的字符主要在
![Alt text](./QQ截图20190416191121.png)
这个函数进行变换
步入进去这个函数
![Alt text](./QQ图片20190416193040.png)

将计算数来的值在0x403020里面对应找到的值再与0x76异或
![Alt text](./QQ图片20190416201046.png)
解析出来是

![Alt text](./QQ图片20190416202957.png)

最后将转换完的字符串与reverse+比较
![Alt text](./QQ截图20190416202656.png)
##### 多次测试最后总结
- 这个循环的输入是6个字符串为一组。输出是4个字符串
所以我们输入是十二个字符

题解奉上
```python
import itertools

cover = [0x37,0x34,0x35,0x32,0x33,0x30,0x31,0x3E,0x3F,0x3C,0x3D,0x3A,0x3B,0x38,0x39,0x26,0x27,0x24,0x25,0x22,0x23,0x20,0x21,0x2E,0x2F,0x2C,0x17,0x14,0x15,0x12,0x13,0x10
,0x11,0x1E,0x1F,0x1C,0x1D,0x1A,0x1B,0x18,0x19,0x06,0x07,0x04,0x05,0x02,0x03,0x00
,0x01,0x0E,0x0F,0x0C,0x46,0x47,0x44,0x45,0x42,0x43,0x40,0x41,0x4E,0x4F,0x5D,0x59
,0x01,0x00,0x00,0x00,0xF8,0x1B,0x2B,0x00,0xB8,0x2E,0x2B,0x00]
cc = ""

input_ = list(itertools.product([1,2,3,4,5,6,7,8,9,"A","B","C","D","E","F"],repeat = 6))


for i in range(len(input_)):
    
    a = int(str(input_[i][0])+str(input_[i][1]),16)
    one = a>>0x2
    
    b = int(str(input_[i][2])+str(input_[i][3]),16)
    tmp = (a&0x3)<<0x4
    tmp_1 = b>>0x4
    two = tmp_1+tmp

    c = int(str(input_[i][4])+str(input_[i][5]),16)
    tmp_2 = ((b&0xF)*2)*2
    tmp_3 = c>>0x6
    three = tmp_2+tmp_3
    
    four = c&0x3F

    fuzz = chr(cover[one]^0x76) + chr(cover[two]^0x76) + chr(cover[three]^0x76) + chr(cover[four]^0x76)
    if(fuzz =="reve" or fuzz =="rse+"):
        print str(hex(a))+str(hex(b))+str(hex(c))

```










