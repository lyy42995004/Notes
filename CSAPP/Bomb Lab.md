# 前言

lab下载链接，[csapp.cs.cmu.edu](https://csapp.cs.cmu.edu/3e/labs.html)

推荐B站视频讲解，[CSAPP bomblab讲解](https://www.bilibili.com/video/BV1vu411o7QP/?spm_id_from=333.999.0.0&vd_source=c7a2228562ec56ca67689b8d47bb87ba)，就是彩蛋没讲有点可惜

可以自定义一个`ans.txt`文件方便运行程序`./bomb ans.txt`

`objdump -d bomb > bomb.s`通过反汇编将bomb的汇编代码重定向到`bomb.s`文件中

在整个拆炸弹的工程中，都是通过`gdb ./bomb`命令进行调试

**所有答案都是通过断点跟踪输入数据是如何存储的与变化的，来分析汇编**

# GDB命令

```gdb
b(break)	[num]/[*addr] 				 	# 打断点
d(delete)	[breakpionts]	[num]       	# 删除断点
info(mation)[b]/[r]/[v]						# 打印断点，寄存器，变量等信息		
r(run)		[ans.txt] 						# 运行程序
c(continue)           				     	# 继续执行程序
stepi										# 单步执行汇编
nexti										# 单语句执行汇编
disas(disassemble)	[func]					# 打印函数的汇编代码
x(examine)/[n][f][u]	[address]			# 打印内存地址处的数据内容
p(print)	[expression]					# 打印变量的值
layout regs									# 显示寄存器信息和执行的汇编代码位置
```

# Phase 1

可以通过`disas`命令（用起来更舒服会显示当前汇编代码是第几行）或者在`bomb.s`中查找来查看函数的汇编代码

 ![image-20241120173054371](https://s2.loli.net/2024/11/20/ryYKBMJTH7aCoFG.png)

发现将一个地址移进了esi寄存器中了，然后又调用了比较string的函数，通过判断eax是否为零，来进行引爆炸弹

所以打印一下该地址的值，就是答案了

 ![image-20241120173546547](https://s2.loli.net/2024/11/20/47BAJXhSLYMRmOt.png)

# Phase 2

```assembly
   0x0000000000400efc <+0>:     push   %rbp
   0x0000000000400efd <+1>:     push   %rbx
   0x0000000000400efe <+2>:     sub    $0x28,%rsp
   0x0000000000400f02 <+6>:     mov    %rsp,%rsi
   0x0000000000400f05 <+9>:     callq  0x40145c <read_six_numbers>	# 从字符中读六个数字
   0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp)	# 比较num[0]!=1爆炸,等于1进入循环
   # 通过单步调试并观察寄存器，判断(rsp)存储第一个数，(rsp+4)存储第二个数，以此类推
   0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:    callq  0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:    jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax	# num[i]
   0x0000000000400f1a <+30>:    add    %eax,%eax		# eax *= 2
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)		# num[i + 1] == 2*num[i]
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>	# 不相等爆炸
   0x0000000000400f20 <+36>:    callq  0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:    add    $0x4,%rbx		# num[i+1]
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx		# rbx == rbp + 24跳出循环
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx	# num[i+1]
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp	# rbp + 24
   0x0000000000400f3a <+62>:    jmp    0x400f17 <phase_2+27>
   0x0000000000400f3c <+64>:    add    $0x28,%rsp
   0x0000000000400f40 <+68>:    pop    %rbx
   0x0000000000400f41 <+69>:    pop    %rbp
   0x0000000000400f42 <+70>:    retq  
```

通过分析可以得到上述伪代码为

```C
if (num[0] != 1)
    explode_bomb();
for (int i = 0; i < 6; i++)
{
    if (num[i] != num[i - 1])
        explode_bomb();
}
```

所以第二个答案是`1 2 4 8 16 32`

# Phase3

```assembly
   0x0000000000400f43 <+0>:     sub    $0x18,%rsp
   0x0000000000400f47 <+4>:     lea    0xc(%rsp),%rcx	# num2
   0x0000000000400f4c <+9>:     lea    0x8(%rsp),%rdx	# num1
   0x0000000000400f51 <+14>:    mov    $0x4025cf,%esi	# 打印地址数据"%d %d"
   0x0000000000400f56 <+19>:    mov    $0x0,%eax
   0x0000000000400f5b <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>	# 调用sscanf读数
   0x0000000000400f60 <+29>:    cmp    $0x1,%eax	# eax不大于1爆炸
   0x0000000000400f63 <+32>:    jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:    callq  0x40143a <explode_bomb>
   # 通过对寄存器的跟踪，发现(rsp+0x8)和(rsp + 0xc)分别存储第一个和第二个数字
   0x0000000000400f6a <+39>:    cmpl   $0x7,0x8(%rsp)	# num1>7爆炸
   0x0000000000400f6f <+44>:    ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:    mov    0x8(%rsp),%eax	# eax=num1
   0x0000000000400f75 <+50>:    jmpq   *0x402470(,%rax,8)	# 跳转到(0x402470 + 8*rax)
   0x0000000000400f7c <+57>:    mov    $0xcf,%eax		# 207
   0x0000000000400f81 <+62>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f83 <+64>:    mov    $0x2c3,%eax		# 707	
   0x0000000000400f88 <+69>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f8a <+71>:    mov    $0x100,%eax		# 256
   0x0000000000400f8f <+76>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f91 <+78>:    mov    $0x185,%eax		# 389
   0x0000000000400f96 <+83>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f98 <+85>:    mov    $0xce,%eax		# 206
   0x0000000000400f9d <+90>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f9f <+92>:    mov    $0x2aa,%eax		# 682
   0x0000000000400fa4 <+97>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400fa6 <+99>:    mov    $0x147,%eax		# 327
   0x0000000000400fab <+104>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fad <+106>:   callq  0x40143a <explode_bomb>
   0x0000000000400fb2 <+111>:   mov    $0x0,%eax		# 0
   0x0000000000400fb7 <+116>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fb9 <+118>:   mov    $0x137,%eax		# 311
   0x0000000000400fbe <+123>:   cmp    0xc(%rsp),%eax	# 新赋的值等于num2
   0x0000000000400fc2 <+127>:   je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:   callq  0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:   add    $0x18,%rsp
   0x0000000000400fcd <+138>:   retq
```

伪代码

```C
switch(num1)
{
// 具体x跳转到哪里我是通过分别带入得到
    case 0:
        ret = 207;
        break;
    case 1:
        ret = 311;
        break;
    case 2:
        ret = 707;
        break;
    case 3:
        ret = 256;
        break;
    case 4:
        ret = 389;
        break;
    case 5:
        ret = 206;
        break;
    case 6:
        ret = 682;
        break;
    case 7:
        ret = 327;
        break;
    default:
        explode_bomb();
}
if (num2 != ret)
    explode_bomb();
```

答案为`0 207` `1 311` `2 707` `3 256` `4 389` `5 206` `6 682` `7 327`其中之一

# Phase 4

```assembly
   0x000000000040100c <+0>:     sub    $0x18,%rsp
   0x0000000000401010 <+4>:     lea    0xc(%rsp),%rcx	# num2
   0x0000000000401015 <+9>:     lea    0x8(%rsp),%rdx	# num1
   0x000000000040101a <+14>:    mov    $0x4025cf,%esi
   0x000000000040101f <+19>:    mov    $0x0,%eax
   0x0000000000401024 <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000401029 <+29>:    cmp    $0x2,%eax	# 不等于2爆炸
   0x000000000040102c <+32>:    jne    0x401035 <phase_4+41>
   0x000000000040102e <+34>:    cmpl   $0xe,0x8(%rsp)	# 14<num1爆炸
   0x0000000000401033 <+39>:    jbe    0x40103a <phase_4+46>
   0x0000000000401035 <+41>:    callq  0x40143a <explode_bomb>
   0x000000000040103a <+46>:    mov    $0xe,%edx		# 14
   0x000000000040103f <+51>:    mov    $0x0,%esi		# 0
   0x0000000000401044 <+56>:    mov    0x8(%rsp),%edi	# num1
   0x0000000000401048 <+60>:    callq  0x400fce <func4>
   0x000000000040104d <+65>:    test   %eax,%eax	# ret不为0爆炸
   0x000000000040104f <+67>:    jne    0x401058 <phase_4+76>
   0x0000000000401051 <+69>:    cmpl   $0x0,0xc(%rsp)	#  num2==0函数返回
   0x0000000000401056 <+74>:    je     0x40105d <phase_4+81>
   0x0000000000401058 <+76>:    callq  0x40143a <explode_bomb>
   0x000000000040105d <+81>:    add    $0x18,%rsp
   0x0000000000401061 <+85>:    retq
```

```assembly
   0x0000000000400fce <+0>:     sub    $0x8,%rsp
   0x0000000000400fd2 <+4>:     mov    %edx,%eax	# eax = edx
   0x0000000000400fd4 <+6>:     sub    %esi,%eax	# eax -= esi
   0x0000000000400fd6 <+8>:     mov    %eax,%ecx	# ecx = eax
   0x0000000000400fd8 <+10>:    shr    $0x1f,%ecx	# ecx >> 31，ecx 正 0 负 1
   0x0000000000400fdb <+13>:    add    %ecx,%eax	# eax += ecx
   0x0000000000400fdd <+15>:    sar    %eax			# eax /= 2
   0x0000000000400fdf <+17>:    lea    (%rax,%rsi,1),%ecx	# ecx = (rax + rsi)
   0x0000000000400fe2 <+20>:    cmp    %edi,%ecx	# num1 >= ecx
   0x0000000000400fe4 <+22>:    jle    0x400ff2 <func4+36>
   0x0000000000400fe6 <+24>:    lea    -0x1(%rcx),%edx	# edx = (rcx - 1)
   0x0000000000400fe9 <+27>:    callq  0x400fce <func4>
   0x0000000000400fee <+32>:    add    %eax,%eax	# eax *= 2
   0x0000000000400ff0 <+34>:    jmp    0x401007 <func4+57>
   0x0000000000400ff2 <+36>:    mov    $0x0,%eax	# eax = 0
   0x0000000000400ff7 <+41>:    cmp    %edi,%ecx	# num1 >= ecx
   0x0000000000400ff9 <+43>:    jge    0x401007 <func4+57>
   0x0000000000400ffb <+45>:    lea    0x1(%rcx),%esi	# esi = (rcx + 1)
   0x0000000000400ffe <+48>:    callq  0x400fce <func4>
   0x0000000000401003 <+53>:    lea    0x1(%rax,%rax,1),%eax	# eax = (rax * 2 + 1)
   0x0000000000401007 <+57>:    add    $0x8,%rsp
   0x000000000040100b <+61>:    retq   
```

func4()是我看网上别人分析的代码，看不懂为啥。。。

第四个语句我是把num1假设为0，带入汇编语句，通过一步一步分析得到num2也应该为0

```C
if (num1 > 14)
    explode_bomb();
// n是num1在edi，l在esi，r在edx
func4(n, l, r)
{
	mid = l + (r - l)/2;
    if (mid == num)
        return 0;
    if (mid > num)
        return 2*func4(num, l, mid - 1);
    else
        return 2*func4(num, mid + 1, r) + 1;
}
if (func() != 0)
    explode_bomb();
if (num2 != 0)
    explode_bomb();
```

答案`0 0` `7 0`，应该还有其他答案

# Phase 5

```assembly
   0x0000000000401062 <+0>:     push   %rbx
   0x0000000000401063 <+1>:     sub    $0x20,%rsp
   0x0000000000401067 <+5>:     mov    %rdi,%rbx
   0x000000000040106a <+8>:     mov    %fs:0x28,%rax
   0x0000000000401073 <+17>:    mov    %rax,0x18(%rsp)
   0x0000000000401078 <+22>:    xor    %eax,%eax	# eax = 0
   0x000000000040107a <+24>:    callq  0x40131b <string_length>
   0x000000000040107f <+29>:    cmp    $0x6,%eax	# str长不为6爆炸
   0x0000000000401082 <+32>:    je     0x4010d2 <phase_5+112>
   0x0000000000401084 <+34>:    callq  0x40143a <explode_bomb>
   0x0000000000401089 <+39>:    jmp    0x4010d2 <phase_5+112>
   0x000000000040108b <+41>:    movzbl (%rbx,%rax,1),%ecx	# ecx = str[i]
   0x000000000040108f <+45>:    mov    %cl,(%rsp)
   0x0000000000401092 <+48>:    mov    (%rsp),%rdx	# rdx = ecx的最低的一字节
   0x0000000000401096 <+52>:    and    $0xf,%edx	# 与0xf取后四位
   0x0000000000401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
   # "maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you"
   # 在这里x/s不断打印就会发现彩蛋
   # "Curses, you've found the secret phase!"
   0x00000000004010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
   0x00000000004010a4 <+66>:    add    $0x1,%rax	# rax++
   0x00000000004010a8 <+70>:    cmp    $0x6,%rax	# rax == 6跳出循环
   0x00000000004010ac <+74>:    jne    0x40108b <phase_5+41>
   0x00000000004010ae <+76>:    movb   $0x0,0x16(%rsp)
   0x00000000004010b3 <+81>:    mov    $0x40245e,%esi	# "flyers"
   0x00000000004010b8 <+86>:    lea    0x10(%rsp),%rdi
   0x00000000004010bd <+91>:    callq  0x401338 <strings_not_equal>
   0x00000000004010c2 <+96>:    test   %eax,%eax	# 不相等爆炸
   0x00000000004010c4 <+98>:    je     0x4010d9 <phase_5+119>	
   0x00000000004010c6 <+100>:   callq  0x40143a <explode_bomb>
   0x00000000004010cb <+105>:   nopl   0x0(%rax,%rax,1)
   0x00000000004010d0 <+110>:   jmp    0x4010d9 <phase_5+119>
   0x00000000004010d2 <+112>:   mov    $0x0,%eax	# eax = 0
   0x00000000004010d7 <+117>:   jmp    0x40108b <phase_5+41>
   0x00000000004010d9 <+119>:   mov    0x18(%rsp),%rax
   0x00000000004010de <+124>:   xor    %fs:0x28,%rax
   0x00000000004010e7 <+133>:   je     0x4010ee <phase_5+140>
   0x00000000004010e9 <+135>:   callq  0x400b30 <__stack_chk_fail@plt>
   0x00000000004010ee <+140>:   add    $0x20,%rsp
   0x00000000004010f2 <+144>:   pop    %rbx
   0x00000000004010f3 <+145>:   retq  
```

```C
s = "maduiers...", cmps = "flyers"
for (int i = 0; i < 6; i++)
{
    index = str[i] & 0xF;
    str[i] = s[index];
}
if (strings_not_equal(s, cmps))
    explode_bomb();
```



通过分析得到字符后四位字节为9 15 14 5 6 7时，也就是的偏移量为这些时对应产生的字符串是"flyers"

答案`ionefg` `IONEFG`等等

# Phase 6

```assembly
   0x00000000004010f4 <+0>:     push   %r14
   0x00000000004010f6 <+2>:     push   %r13
   0x00000000004010f8 <+4>:     push   %r12
   0x00000000004010fa <+6>:     push   %rbp
   0x00000000004010fb <+7>:     push   %rbx
   0x00000000004010fc <+8>:     sub    $0x50,%rsp
   0x0000000000401100 <+12>:    mov    %rsp,%r13
   0x0000000000401103 <+15>:    mov    %rsp,%rsi
   0x0000000000401106 <+18>:    callq  0x40145c <read_six_numbers>
   0x000000000040110b <+23>:    mov    %rsp,%r14	# r14 = rsp
   0x000000000040110e <+26>:    mov    $0x0,%r12d	# r12d = 0
   0x0000000000401114 <+32>:    mov    %r13,%rbp	# &num[i]
   0x0000000000401117 <+35>:    mov    0x0(%r13),%eax	# num[i]
   0x000000000040111b <+39>:    sub    $0x1,%eax	# num[i]--
   0x000000000040111e <+42>:    cmp    $0x5,%eax	# num[i]>7爆炸
   0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:    callq  0x40143a <explode_bomb>
   0x0000000000401128 <+52>:    add    $0x1,%r12d	# r12++
   0x000000000040112c <+56>:    cmp    $0x6,%r12d	# r12==6跳出循环
   0x0000000000401130 <+60>:    je     0x401153 <phase_6+95>
   0x0000000000401132 <+62>:    mov    %r12d,%ebx	# ebx = r12d
   0x0000000000401135 <+65>:    movslq %ebx,%rax	# rax = ebx
   0x0000000000401138 <+68>:    mov    (%rsp,%rax,4),%eax	# num[i+1]
   0x000000000040113b <+71>:    cmp    %eax,0x0(%rbp)	# num[i]!=num[i+1]
   0x000000000040113e <+74>:    jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:    callq  0x40143a <explode_bomb>
   0x0000000000401145 <+81>:    add    $0x1,%ebx	# ebx++
   0x0000000000401148 <+84>:    cmp    $0x5,%ebx	# ebx>5跳出循环
   0x000000000040114b <+87>:    jle    0x401135 <phase_6+65>
   0x000000000040114d <+89>:    add    $0x4,%r13	# rsp + 4
   0x0000000000401151 <+93>:    jmp    0x401114 <phase_6+32>
```

```C
for (int i = 0; i != 6; i++)
{
    if (num[i] > 7)
        explode_bomb();
    for (int j = i + 1; j <= 5; j++)
    {
        if (num[j] == num[i])
            explode_bomb();
    }
}
```

这六个数都小于7，且互不相等


```assembly
   0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi	# rsp + 24
   0x0000000000401158 <+100>:   mov    %r14,%rax	# rsp
   0x000000000040115b <+103>:   mov    $0x7,%ecx	# ecx = 7
   0x0000000000401160 <+108>:   mov    %ecx,%edx	# edx = ecx
   0x0000000000401162 <+110>:   sub    (%rax),%edx	# edx -= (rax)
   0x0000000000401164 <+112>:   mov    %edx,(%rax)	# edx = (rax)
   0x0000000000401166 <+114>:   add    $0x4,%rax	# rax += 4
   0x000000000040116a <+118>:   cmp    %rsi,%rax	# rax != rsp + 24跳出循环
   0x000000000040116d <+121>:   jne    0x401160 <phase_6+108>
```

```C
for (int i = 0; i <= 6; i++)
    num[i] = 7 - num[i];
```

每个数都等于7-本身

```assembly
   0x000000000040116f <+123>:   mov    $0x0,%esi	# esi = 0
   0x0000000000401174 <+128>:   jmp    0x401197 <phase_6+163>
   0x0000000000401176 <+130>:   mov    0x8(%rdx),%rdx	# rdx = (rdx + 8) = rdx + 16
   # 通过调试发现，rdx每次增加16，进一步观察可以发现rdx是结构节点，+8指向next指针，所以每次增加16
   0x000000000040117a <+134>:   add    $0x1,%eax	# eax++
   0x000000000040117d <+137>:   cmp    %ecx,%eax
   0x000000000040117f <+139>:   jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:   jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:   mov    $0x6032d0,%edx
   0x0000000000401188 <+148>:   mov    %rdx,0x20(%rsp,%rsi,2)	# n[i] = rdx就是node的地址
   0x000000000040118d <+153>:   add    $0x4,%rsi	# rsi += 4
   0x0000000000401191 <+157>:   cmp    $0x18,%rsi	# rsi==24跳出循环
   0x0000000000401195 <+161>:   je     0x4011ab <phase_6+183>
   0x0000000000401197 <+163>:   mov    (%rsp,%rsi,1),%ecx	# ecx = num[i]
   0x000000000040119a <+166>:   cmp    $0x1,%ecx	
   0x000000000040119d <+169>:   jle    0x401183 <phase_6+143>
   0x000000000040119f <+171>:   mov    $0x1,%eax	# eax = 1
   0x00000000004011a4 <+176>:   mov    $0x6032d0,%edx	# edx = node
   0x00000000004011a9 <+181>:   jmp    0x401176 <phase_6+130>
```

 ![ 2024-11-23 104418.png](https://s2.loli.net/2024/11/23/2Cog8nEZU7eGJQL.png)

```C
struct node
{
    int val;
    int index;
    struct node* next;
}
```

```C
node* n[6];
node* p = 0x632d0;
for (int i = 0; i < 6; i++)
{
    node* cur = p;
    for (int j = 0; j < num[i]; j++)
    {
        n[i] = cur;
        cur = p->next;
    }
}
```

按照num[i]给n[i]，这个节点指针数组分别赋值

```assembly
   0x00000000004011ab <+183>:   mov    0x20(%rsp),%rbx	# &&n[0]
   0x00000000004011b0 <+188>:   lea    0x28(%rsp),%rax	# &&n[1]
   0x00000000004011b5 <+193>:   lea    0x50(%rsp),%rsi  # &&n[6]
   0x00000000004011ba <+198>:   mov    %rbx,%rcx	
   0x00000000004011bd <+201>:   mov    (%rax),%rdx	# n[i+1]
   0x00000000004011c0 <+204>:   mov    %rdx,0x8(%rcx)	# n[i]->next = n[i+1]
   0x00000000004011c4 <+208>:   add    $0x8,%rax	# n[i] = n[i]->next
   0x00000000004011c8 <+212>:   cmp    %rsi,%rax	# rax==rsi跳出循环
   0x00000000004011cb <+215>:   je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:   mov    %rdx,%rcx	
   0x00000000004011d0 <+220>:   jmp    0x4011bd <phase_6+201>
   0x00000000004011d2 <+222>:   movq   $0x0,0x8(%rdx)
```

```C
for (int i = 0; i < 5; i++)
    n[i]->next = n[i+1];
n[5]->next = NULL;
```

将n[i]按数组的顺序连接起来

```assembly
   0x00000000004011da <+230>:   mov    $0x5,%ebp
   0x00000000004011df <+235>:   mov    0x8(%rbx),%rax	# n[i+1]
   0x00000000004011e3 <+239>:   mov    (%rax),%eax	# n[i+1]->val
   0x00000000004011e5 <+241>:   cmp    %eax,(%rbx)	# n[i+1]->val>n[i]->val爆炸
   0x00000000004011e7 <+243>:   jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:   callq  0x40143a <explode_bomb>
   0x00000000004011ee <+250>:   mov    0x8(%rbx),%rbx	# rbp+8
   0x00000000004011f2 <+254>:   sub    $0x1,%ebp	# ebp--
   0x00000000004011f5 <+257>:   jne    0x4011df <phase_6+235>
   0x00000000004011f7 <+259>:   add    $0x50,%rsp
   0x00000000004011fb <+263>:   pop    %rbx
   0x00000000004011fc <+264>:   pop    %rbp
   0x00000000004011fd <+265>:   pop    %r12
   0x00000000004011ff <+267>:   pop    %r13
   0x0000000000401201 <+269>:   pop    %r14
   0x0000000000401203 <+271>:   retq
```

```C
for (int k = 5; k != 0; k--)
{
    i = 0;
    if (n[i]->val < n[i+1]->val)
        explode_bomb();
    i++;
}
```

**根据输入的六个数，每个数再通过7-本身得到新的数，通过新的num[i]数组，得到对应的节点指针数组n[i]，再将节点指针数组的next指针更新连接起来，最后进行判断数组是否是呈现递减趋势的**

# Secret Phase

 ![ 2024-11-23 145313.png](https://s2.loli.net/2024/11/23/Xh54palqMtnOvfe.png)

bomb.c中在拆完最后一个炸弹后，作者留下了这样的一段话，我们可以知道一定是错过了什么，在拆前六个炸弹时，我们也能发现一些端倪，就比如在`main()`发现每次拆弹成功后都会调用`phase_defused()`函数，在连续打印地址后续的字符串时会发现彩蛋，在重定向产生的`bomb.s`有一些从未用过的函数`fun7()`，`secret_phase`，下面我们就从`phase_defused()`函数来分析隐藏炸弹

```assembly
Dump of assembler code for function phase_defused:
   0x00000000004015c4 <+0>:     sub    $0x78,%rsp
   0x00000000004015c8 <+4>:     mov    %fs:0x28,%rax
   0x00000000004015d1 <+13>:    mov    %rax,0x68(%rsp)
   0x00000000004015d6 <+18>:    xor    %eax,%eax
   0x00000000004015d8 <+20>:    cmpl   $0x6,0x202181(%rip)        # 0x603760 <num_input_strings>
   # (0x202181 + $rip)存储的是所拆炸弹数量，必须把六个炸弹都拆了才能继续执行
   0x00000000004015df <+27>:    jne    0x40163f <phase_defused+123>
   0x00000000004015e1 <+29>:    lea    0x10(%rsp),%r8	# str
   0x00000000004015e6 <+34>:    lea    0xc(%rsp),%rcx	# y
   0x00000000004015eb <+39>:    lea    0x8(%rsp),%rdx	# x
   0x00000000004015f0 <+44>:    mov    $0x402619,%esi	# "%d %d %s"
   0x00000000004015f5 <+49>:    mov    $0x603870,%edi	# "7 0 DrEvil"
   0x00000000004015fa <+54>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x00000000004015ff <+59>:    cmp    $0x3,%eax	# 三个参数才能出发隐藏炸弹
   0x0000000000401602 <+62>:    jne    0x401635 <phase_defused+113>
   0x0000000000401604 <+64>:    mov    $0x402622,%esi	# "DrEvil"
   0x0000000000401609 <+69>:    lea    0x10(%rsp),%rdi	# 需要输入的字符串strs要与上面的相同
   0x000000000040160e <+74>:    callq  0x401338 <strings_not_equal>
   0x0000000000401613 <+79>:    test   %eax,%eax
   0x0000000000401615 <+81>:    jne    0x401635 <phase_defused+113>
   0x0000000000401617 <+83>:    mov    $0x4024f8,%edi
   # "Curses, you've found the secret phase!"
   0x000000000040161c <+88>:    callq  0x400b10 <puts@plt>
   0x0000000000401621 <+93>:    mov    $0x402520,%edi
   # "But finding it and solving it are quite different..."
   0x0000000000401626 <+98>:    callq  0x400b10 <puts@plt>
   0x000000000040162b <+103>:   mov    $0x0,%eax
   0x0000000000401630 <+108>:   callq  0x401242 <secret_phase>
   0x0000000000401635 <+113>:   mov    $0x402558,%edi
   # "Congratulations! You've defused the bomb!"
   0x000000000040163a <+118>:   callq  0x400b10 <puts@plt>
   0x000000000040163f <+123>:   mov    0x68(%rsp),%rax
   0x0000000000401644 <+128>:   xor    %fs:0x28,%rax
   0x000000000040164d <+137>:   je     0x401654 <phase_defused+144>
   0x000000000040164f <+139>:   callq  0x400b30 <__stack_chk_fail@plt>
   0x0000000000401654 <+144>:   add    $0x78,%rsp
   0x0000000000401658 <+148>:   retq   
```

```C
void phase_defused()
{
    if (phase_1,...,phase_6 未全部拆除)
        return;
    puts("Congratulations! You've defused the bomb!");
    if (sscanf("%d %d %s", x, y, str) != 3)
    	return;
    if (str = "DrEvil")
    {
        puts("Curses, you've found the secret phase!");
        puts("But finding it and solving it are quite different...");
        secret_phase();
        if (隐藏炸弹被拆)
            puts("Congratulations! You've defused the bomb!");
    }
}
```

```assembly
Dump of assembler code for function secret_phase:
   0x0000000000401242 <+0>:     push   %rbx
   0x0000000000401243 <+1>:     callq  0x40149e <read_line>
   0x0000000000401248 <+6>:     mov    $0xa,%edx	# 10
   0x000000000040124d <+11>:    mov    $0x0,%esi	# 0
   0x0000000000401252 <+16>:    mov    %rax,%rdx
   0x0000000000401255 <+19>:    callq  0x400bd0 <strtol@plt>	# strtol提取字符串为整数
   0x000000000040125a <+24>:    mov    %rax,%rbx
   0x000000000040125d <+27>:    lea    -0x1(%rax),%eax
   0x0000000000401260 <+30>:    cmp    $0x3e8,%eax	# eax>1000爆炸
   0x0000000000401265 <+35>:    jbe    0x40126c <secret_phase+42>
   0x0000000000401267 <+37>:    callq  0x40143a <explode_bomb>
   0x000000000040126c <+42>:    mov    %ebx,%esi	# 传ebx
   0x000000000040126e <+44>:    mov    $0x6030f0,%edi	# 传地址
   0x0000000000401273 <+49>:    callq  0x401204 <fun7>
   0x0000000000401278 <+54>:    cmp    $0x2,%eax	# fun7返回值不等于2爆炸
   0x000000000040127b <+57>:    je     0x401282 <secret_phase+64>	
   0x000000000040127d <+59>:    callq  0x40143a <explode_bomb>
   0x0000000000401282 <+64>:    mov    $0x402438,%edi
   # "Wow! You've defused the secret stage!"
   0x0000000000401287 <+69>:    callq  0x400b10 <puts@plt>
   0x000000000040128c <+74>:    callq  0x4015c4 <phase_defused>
   0x0000000000401291 <+79>:    pop    %rbx
   0x0000000000401292 <+80>:    retq  
```

```C
void secret_phase()
{
    num = strtol(s);
    if (num > 1000)
        explode_bomb();
    if (fun7(地址, num) != 2)
        explode_bomb();
}
```

```assembly
Dump of assembler code for function fun7:
   # rdi传入地址就是根节点，esi传入的num，eax函数的返回值
   0x0000000000401204 <+0>:     sub    $0x8,%rsp
   0x0000000000401208 <+4>:     test   %rdi,%rdi			# 空地址跳转
   0x000000000040120b <+7>:     je     0x401238 <fun7+52>
   0x000000000040120d <+9>:     mov    (%rdi),%edx			# edx = root->val
   0x000000000040120f <+11>:    cmp    %esi,%edx			# root->val<=num跳转
   0x0000000000401211 <+13>:    jle    0x401220 <fun7+28>
   0x0000000000401213 <+15>:    mov    0x8(%rdi),%rdi		# root = root->left
   0x0000000000401217 <+19>:    callq  0x401204 <fun7>
   0x000000000040121c <+24>:    add    %eax,%eax			# ret*=2
   0x000000000040121e <+26>:    jmp    0x40123d <fun7+57>
   0x0000000000401220 <+28>:    mov    $0x0,%eax			# eax = 0
   0x0000000000401225 <+33>:    cmp    %esi,%edx			# num == root->val返回0
   0x0000000000401227 <+35>:    je     0x40123d <fun7+57>
   0x0000000000401229 <+37>:    mov    0x10(%rdi),%rdi		# root = root->right
   0x000000000040122d <+41>:    callq  0x401204 <fun7>
   0x0000000000401232 <+46>:    lea    0x1(%rax,%rax,1),%eax# ret = ret*2 + 1
   0x0000000000401236 <+50>:    jmp    0x40123d <fun7+57>
   0x0000000000401238 <+52>:    mov    $0xffffffff,%eax		# ret = -1
   0x000000000040123d <+57>:    add    $0x8,%rsp
   0x0000000000401241 <+61>:    retq   
```

```C
struct Node {
    int value;
    Node* left;
    Node* right;
};
int fun7(Node* root, int num)
{
    if (root == NULL)
        return -1;
    if (root->val == num)
        return 0;
    if (root->val > num)
        return 2*fun7(root->left, num);
    if (root->val < num)
        return 2*fun7(root->right, num) + 1;
}
```

 <img src="https://s2.loli.net/2024/11/23/T4FQ3raftezOX6m.png" style="zoom:67%;" />

                   36
                 /     \
               8        50
             /  \      /   \
            6   22    45  107 
          / \   / \  / \   /  \
         1   7 20 35 40 47 99 1001
    当fun7(root, 22)时返回2

**从 `phase_defused` 开始：**检查输入条件是否满足。如果满足，进入 `secret_phase`。

**在 `secret_phase` 中**：解析用户输入的目标值，并检查范围。将目标值传递给 `fun7`，在二叉树中查找路径。

**在 `fun7` 中**：按二叉树递归查找目标值。返回路径编码，验证是否与预期值（`2`）匹配。

**返回到 `secret_phase`**：如果路径正确，打印成功信息。返回到 `phase_defused` 结束程序。

**最终答案**

 ![ 2024-11-23 165828.png](https://s2.loli.net/2024/11/23/2DmvQJHN7nMWGpV.png)

 ![ 2024-11-23 170023.png](https://s2.loli.net/2024/11/23/XgR9jy7ZbozkfIs.png)