---
title: CRC校验
date: 2018-09-08 15:36:19
categories: tech
tags:
- computer network
---

因为前几天笔试题的时候，让我手算 CRC 校验。。然后今天随手写了一个 CRC 校验的脚本。

当然了，现实中的 CRC 校验是在数据链路层实现的，都是硬件完成的，我只是模拟一下 CRC 校验的过程。

## CRC简介

现实中通信线路都不会是理想的，比特在传输过程中会出现差错的，比如 0 变成 1，1 变成 0，这个就叫做 **比特差错**。传输错误的比特占比特总数的比率叫做 **误码率 BER（Bit Error Rate）**。

为了保证数据传输的可靠性，必须采用各种差错检测措施，目前在数据链路层广泛采用了 CRC（Cyclic Redundancy Check）的检测技术。

## CRC 过程

数据链路层的数据单位是帧（frame）。

双方要事先约定一个除数 P，P 可以用多项式表示，P(X) 叫做 **生成多项式**。

几种常见的生成多项式：


CRC P       | 	Checksum Width  |  Generator Polynomial
---         | ---               | ---
CRC-CCITT   |   16 bits         | 	10001000000100001
CRC-16      | 	16 bits         |   11000000000000101 
CRC-32      |   32 bits         |   100000100110000010001110110110111

-----

CRC 的过程大概是这样的：

- P 的长度为 n，frame 就左移 (n - 1) 的长度，相当于在后面加上 (n - 1) 个零，此时 frameM = $frame * 2 ^ {n}$
- 然后就是 frameM 作为被除数，P 作为除数，做二进制的模 2 运算（相当于做二进制加法，但是不进位，**其实跟异或一样**）。
- 求出余数，就是 FCS（Frame Check Sequence），加在 frame 后面
- 接收端收到 frame 后，同样跟 P 做模 2 运算，如果正确的话，**余数一定是 0**，表示没有差错，就接受，如果出错的话，无法判断哪个出错，就丢弃

这个图中的原始的 frame 是 10110011，P 是 11001，P 的长度是 n 为 5，所以给原始的 frame 填上 4 个零（填上 n - 1 个零），然后求出余数为 FCS 为 0100，加在原始的 frame 后面，变成 101100110100 传给接收端，接收端收到 101100110100 后，跟 P 做模 2 运算，余数为 0 则表示差错，然后接受这个帧。

![crc.png](https://i.loli.net/2018/09/08/5b938180e1c25.png)

最后可以实现的是，**在接收端数据链路层接受的帧均无差错**。

## CRC 代码实现

- check() 函数就是模 2 运算，算出余数（FCS）
- crc() 就给 frame 填上零然后做模 2 运算，返回运算的余数。
- 别的就是我随机生成一个二进制的 frame 和几个常见生成多项式做的测试代码

```python
def check(frame, p):
  n = len(p) - 1
  fcs = frame[0: n]
  for i in range(n, len(frame)):
    fcs += frame[i]
    fcs = bin(int(fcs, 2))[2:]
    if (len(fcs) < len(p)): 
      continue
    fcs = bin(int(fcs, 2) ^ int(p, 2))[2:]
  while len(fcs) < n:
    fcs = "0" + fcs
  return fcs

def crc(frame, p):
  n = len(p) - 1
  frameM = frame +  n * "0"
  return check(frameM, p)

import random

def test(p):
  for i in range(50):
    frame = random.choice("01") * random.randint(50, 100)
    assert check(frame + crc(frame, p), p) == (len(p) - 1) * "0", f"encountered error when tested {p}"


CRC_16 = "11000000000000101"                    # 16 bits
CRC_CCITT = "10001000000100001"                 # 16 bits
CRC_32 = "100000100110000010001110110110111"    # 32 bits

test(CRC_16)
test(CRC_CCITT)
test(CRC_32)
print("Accept")
```



## References

- [CRC.py - GIST](https://gist.github.com/pwxcoo/a664bf4b05f845e2891ed7fccd9dcffe)
- [Cyclic redundancy check](https://en.wikipedia.org/wiki/Cyclic_redundancy_check#Designing_polynomials)
- [CRC Mathematics and Theory](https://barrgroup.com/Embedded-Systems/How-To/CRC-Math-Theory)
- 谢希仁的《计算机网络》第六版