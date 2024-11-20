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
   0x000000000040110b <+23>:    mov    %rsp,%r14
   0x000000000040110e <+26>:    mov    $0x0,%r12d
   0x0000000000401114 <+32>:    mov    %r13,%rbp	# &num[i]
   0x0000000000401117 <+35>:    mov    0x0(%r13),%eax	# num[i]
   0x000000000040111b <+39>:    sub    $0x1,%eax	# num[i]--
   0x000000000040111e <+42>:    cmp    $0x5,%eax	# num[i]>6爆炸
   0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:    callq  0x40143a <explode_bomb>
   0x0000000000401128 <+52>:    add    $0x1,%r12d	# r12++
   0x000000000040112c <+56>:    cmp    $0x6,%r12d	# r12==6跳出循环
   0x0000000000401130 <+60>:    je     0x401153 <phase_6+95>
   0x0000000000401132 <+62>:    mov    %r12d,%ebx
   0x0000000000401135 <+65>:    movslq %ebx,%rax
   0x0000000000401138 <+68>:    mov    (%rsp,%rax,4),%eax	
   0x000000000040113b <+71>:    cmp    %eax,0x0(%rbp)	# num[i]互不相等
   0x000000000040113e <+74>:    jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:    callq  0x40143a <explode_bomb>
   0x0000000000401145 <+81>:    add    $0x1,%ebx	# ebx++
   0x0000000000401148 <+84>:    cmp    $0x5,%ebx	# ebx>5跳出循环
   0x000000000040114b <+87>:    jle    0x401135 <phase_6+65>
   0x000000000040114d <+89>:    add    $0x4,%r13
   0x0000000000401151 <+93>:    jmp    0x401114 <phase_6+32>
```

```assembly
   0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi
   0x0000000000401158 <+100>:   mov    %r14,%rax
   0x000000000040115b <+103>:   mov    $0x7,%ecx
   0x0000000000401160 <+108>:   mov    %ecx,%edx
   0x0000000000401162 <+110>:   sub    (%rax),%edx
   0x0000000000401164 <+112>:   mov    %edx,(%rax)
   0x0000000000401166 <+114>:   add    $0x4,%rax
   0x000000000040116a <+118>:   cmp    %rsi,%rax
   0x000000000040116d <+121>:   jne    0x401160 <phase_6+108>
   0x000000000040116f <+123>:   mov    $0x0,%esi
   0x0000000000401174 <+128>:   jmp    0x401197 <phase_6+163>
   0x0000000000401176 <+130>:   mov    0x8(%rdx),%rdx
   0x000000000040117a <+134>:   add    $0x1,%eax
   0x000000000040117d <+137>:   cmp    %ecx,%eax
   0x000000000040117f <+139>:   jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:   jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:   mov    $0x6032d0,%edx
   0x0000000000401188 <+148>:   mov    %rdx,0x20(%rsp,%rsi,2)
   0x000000000040118d <+153>:   add    $0x4,%rsi
   0x0000000000401191 <+157>:   cmp    $0x18,%rsi
   0x0000000000401195 <+161>:   je     0x4011ab <phase_6+183>
   0x0000000000401197 <+163>:   mov    (%rsp,%rsi,1),%ecx
   0x000000000040119a <+166>:   cmp    $0x1,%ecx
   0x000000000040119d <+169>:   jle    0x401183 <phase_6+143>
   0x000000000040119f <+171>:   mov    $0x1,%eax
   0x00000000004011a4 <+176>:   mov    $0x6032d0,%edx
   0x00000000004011a9 <+181>:   jmp    0x401176 <phase_6+130>
```

```assembly
   0x00000000004011ab <+183>:   mov    0x20(%rsp),%rbx
   0x00000000004011b0 <+188>:   lea    0x28(%rsp),%rax
   0x00000000004011b5 <+193>:   lea    0x50(%rsp),%rsi
   0x00000000004011ba <+198>:   mov    %rbx,%rcx
   0x00000000004011bd <+201>:   mov    (%rax),%rdx
   0x00000000004011c0 <+204>:   mov    %rdx,0x8(%rcx)
   0x00000000004011c4 <+208>:   add    $0x8,%rax
   0x00000000004011c8 <+212>:   cmp    %rsi,%rax
   0x00000000004011cb <+215>:   je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:   mov    %rdx,%rcx
   0x00000000004011d0 <+220>:   jmp    0x4011bd <phase_6+201>
   0x00000000004011d2 <+222>:   movq   $0x0,0x8(%rdx)
   0x00000000004011da <+230>:   mov    $0x5,%ebp
   0x00000000004011df <+235>:   mov    0x8(%rbx),%rax
   0x00000000004011e3 <+239>:   mov    (%rax),%eax
   0x00000000004011e5 <+241>:   cmp    %eax,(%rbx)
   0x00000000004011e7 <+243>:   jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:   callq  0x40143a <explode_bomb>
   0x00000000004011ee <+250>:   mov    0x8(%rbx),%rbx
   0x00000000004011f2 <+254>:   sub    $0x1,%ebp
   0x00000000004011f5 <+257>:   jne    0x4011df <phase_6+235>
   0x00000000004011f7 <+259>:   add    $0x50,%rsp
   0x00000000004011fb <+263>:   pop    %rbx
   0x00000000004011fc <+264>:   pop    %rbp
   0x00000000004011fd <+265>:   pop    %r12
   0x00000000004011ff <+267>:   pop    %r13
   0x0000000000401201 <+269>:   pop    %r14
   0x0000000000401203 <+271>:   retq
```



# Secret Phase