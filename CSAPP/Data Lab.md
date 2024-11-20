# 前言

这个lab大部分都不是自己写出来的，太难了，看的是[CSAPP datalab讲解](https://www.bilibili.com/video/BV183411k7VM/?spm_id_from=333.999.0.0&vd_source=c7a2228562ec56ca67689b8d47bb87ba)，讲的很好

# Rating: 1

 <img src="https://s2.loli.net/2024/11/20/2kzybU6I1Mf8FvS.png#pic_left" style="zoom:;" />

 <img src="https://s2.loli.net/2024/11/20/de5iwsJQB3zx6jv.png#pic_left" style="zoom:50%;" />

```C
int bitXor(int x, int y) {
  return (~(x & y)) & (~((~x) & (~y)));
}
```

 <img src="https://s2.loli.net/2024/11/20/egcpUdqJik2GrD8.png#pic_left" alt="image-20241120103901469" />

```C
int tmin(void) {
  // 实际有32位，只用8位做演示
  // 10000000 -> -2147483648
  // 11111111 -> -1
  return 1 << 31;
} 
```


 <img src="https://s2.loli.net/2024/11/20/SqEQeAhJFz15rU7.png#pic_left" alt="image-20241120104305653" />

```C
int isTmax(int x) {
  // 实际有32位，只用8位做演示
  // 01111111 + 1 -> 10000000 取反 -> 01111111
  // 要排除下面情况
  // 11111111 + 1 -> 00000000 取反 -> 11111111
  // 0 !! -> 0  其他数 !! -> 1
  return !(~(x + 1) ^ x) & !!(x + 1);
}
```

# Rating:2

<img src="https://s2.loli.net/2024/11/20/cLozOv2KB3Atkf4.png#pic_left" alt="image-20241120105450272" />

```C
int allOddBits(int x) {
  int A = 0xA;
  int AA = A | (A << 4);
  int AAAA = AA | (AA << 8);
  int mask = AAAA | (AAAA << 16);
  return !((x & mask) ^ mask);
}
```

 <img src="https://s2.loli.net/2024/11/20/SR9cvByd18e436q.png#pic_left" alt="image-20241120110206329" />

```C
int negate(int x) {
  return ~x + 1;
}
```

# Rating:3

<img src="https://s2.loli.net/2024/11/20/4FLJy6pZ7s29CHT.png" alt="image-20241120110351873" />

```C
int isAsciiDigit(int x) {
  // 0x30 00110000
  // 0x39 00111001

  // 1. 11XXXX
  int a = ((x >> 4) ^ 0x3);
  int cond1 = !a;
  // 2. XXXX < 10 -> XXXX - 10 < 0
  int b = x & 0xF;
  int res = b + (~(0xA) + 1);
  int cond2 = !!(res >> 31);
  return cond1 & cond2;
}
```

 <img src="https://s2.loli.net/2024/11/20/QcG25wjUNRsakBW.png" alt="image-20241120110601926" />

```C
int conditional(int x, int y, int z) {
  // x = 1 mask = 11111111
  // x = 0 mask = 00000000  
  int mask = (!!x) << 31 >> 31;	
  return (mask & y) | (~mask & z);
}
```

 <img src="https://s2.loli.net/2024/11/20/YT6vrCPV4oKEzF7.png" alt="image-20241120110721713" />

```C
int isLessOrEqual(int x, int y) {
  // 1. x == y
  int cond1 = !(x ^ y);
  int signX = (x >> 31) & 0x1;
  int signY = (y >> 31) & 0x1;
  // 2. x - y + 
  int cond2 = (signX & !signY);
  // 3. x + y -
  int cond3 = ((!signX) & signY);
  // 4. x + y + or x - y - 
  // x - y 来判断大小，不分类直接减会有溢出
  int cond4 = (!!((x + ~y + 1) >> 31));
  return cond1 | cond2 | ((!cond3) & cond4);
}
```

# Rating:4

 <img src="https://s2.loli.net/2024/11/20/LgpMVSt3CT4w9EY.png" alt="image-20241120110932751" />

```C
int logicalNeg(int x) {
  // 0 取反 -> 0
  // 其他数取反一定有负的
  int negX = ~x + 1;
  int sign = (negX | x) >> 31;
  return sign + 1;
}
```

<img src="https://s2.loli.net/2024/11/20/lBhXcMzyURjeGIk.png" alt="image-20241120111215827" />

```C
int howManyBits(int x) {
  // flag 全1 x负 全0 x正
  int flag = x >> 31;
  int bit_16, bit_8, bit_4, bit_2, bit_1, bit_0;
  // 将负的x取反，只需要计算1所在的最高位大小，加上符号位的1
  x = ((~flag) & x) | (flag & (~x));
  // 10000000 00000000 00000000 00000000
  // 10000000 00000000 
  // 10000000
  // 1000
  // 10 
  // 1
  // 采用类似二分的思想，计算高十六位有没有值
  // 有继续计算高十六位，没有计算第十六位的值，其余类似
  bit_16 = (!((!!(x >> 16)) ^ 0x1)) << 4;
  x = x >> bit_16;
  bit_8 = (!((!!(x >> 8)) ^ 0x1)) << 3;
  x = x >> bit_8;
  bit_4 = (!((!!(x >> 4)) ^ 0x1)) << 2;
  x = x >> bit_4; 
  bit_2 = (!((!!(x >> 2)) ^ 0x1)) << 1;
  x = x >> bit_2;
  bit_1 = (!((!!(x >> 1)) ^ 0x1));
  x = x >> bit_1;
  bit_0 = x;
  return bit_16 + bit_8 + bit_4 + bit_2 + bit_1 + bit_0 + 1;
}
```

**float**

 <img src="https://s2.loli.net/2024/11/20/2n1XQArSbkjhOCG.png" alt="image-20241120112020338" />
$$
V = (-1)^s*M*2^E
$$
浮点数的**实际值**，等于**符号位**（sign bit）乘以**指数偏移值**（exponent bias）再乘以**分数值**（fraction)。

 <img src="https://s2.loli.net/2024/11/20/3rpYAxcLalmqKMH.png" alt="image-20241120111939697" />

 <img src="https://s2.loli.net/2024/11/20/6lWzEqTCDJGQfX4.png" alt="202411201116470.png" />

```C
unsigned floatScale2(unsigned uf) {
  unsigned s = (uf >> 31) & 0x1;
  unsigned expr = (uf >> 23) & 0xFF;
  unsigned frac = uf & 0x7FFFFF;
  // 0
  if (expr == 0 && frac == 0) 
    return uf;
  // infity or not a number
  if (expr == 0xFF)
    return uf;
  // denormalize
  if (expr == 0)
  {
    frac <<= 1;
    return (s << 31) | frac;
  }
  // normalize
  expr++;
  return (s << 31) | expr << 23 | frac;
}
```

<img src="https://s2.loli.net/2024/11/20/FzBhabXmUg1qywA.png" alt="image-20241120112942886" />

```C
int floatFloat2Int(unsigned uf) {
  unsigned s = (uf >> 31) & 0x1;
  unsigned expr = (uf >> 23) & 0xFF;
  unsigned frac = uf & 0x7FFFFF;
  int e = expr - 127;
  // 0
  if (expr == 0 && frac == 0)
    return 0;
  // infity or not a number 
  if (expr == 0xFF)
    return 1 << 31;
  // denormalize
  if (expr == 0) 
    return 0;
  // normalize
  frac = (1 << 23) | frac; 
  if (e > 31)
    return 1 << 31;
  else if (e < 0)
    return 0;
  if (e > 23)
    frac <<= (e - 23);
  else 
    frac >>= (23 - e);
  if (s)
    frac = -frac;
  return frac;
}
```

<img src="https://s2.loli.net/2024/11/20/EucaLZfQSDvdRem.png" alt="image-20241120113047708" />

```C
unsigned floatPower2(int x) {
  if (x < -149)  
    return 0;
  else if (x < -126)
  {
    int shift = 23 + (x + 126);
    return 1 << shift;
  }
  else if (x <= 127)
  {
    int e = x + 127;
    return e << 23;
  }
  else
    return 0xFF << 23;
}
```
