# 三、程序的机器级表示

产生汇编代码：`gcc -Og -S xx.c`

编译并汇编：`gcc -Og -c xx.c`

展示程序的字节表示：`x/14xb xx` 显示从xx所处地址开始的14个16进制格式表示的字节

反汇编：`objdump -d xx.o`

## 访问信息

### 通用目的寄存器

63->31->15->7->0

返回值：%rax, %eax, %ax, %al

被调用者保存：%rbx, %rbp,( %ebx, %ebp, ), %12-15

第654321个参数：%r9, %r8, %rcx, %rdx, %rsi, %rdi

栈指针：%rsp, %esp, %sp, %spl

调用者保存：%r10,11

### 操作数指示符

1. $立即数
2. r寄存器。ra:寄存器a。R[ra]:它的值
3. Mb[Addr]内存引用：从Addr开始b个字节值的引用。
4. 寻址：Imm(rb, ri, s): Imm + R[rb] + R[ri]*s

### 数据传送指令

MOV S, D : D<--S

MOVZ S, R : R<--零扩展(S)

MOVS S, R : R<--符号扩展(S)

cltq: %rax<--符号扩展(%eax)

movb字节，w字，l双字，q四字

movzbw-movzbl-movzwl-movzbq-movzwq

movsbw...........................................................-movslq

movabsq绝对的四字: 以任意64位立即数值作为源操作数，并且只能以寄存器作为目的

### 压入和弹出栈数据

pushq S : R[%rsp]<--R[%rsp] - 8

​				  M[R[%rsp]]<--S

popq D : D<--M[R[%rsp]]

​				R[%rsp]<--R[%rsp] + 8

## 算数和逻辑操作

### 加载有效地址

leaq : 从内存读数据到寄存器，实际上没有引用内存。目的必须是寄存器。用来进行地址计算。

leaq 7(%rdx, %rdx, 4), %rax -->%rax = 5 * x + 7

imulq : 补码乘法，乘积放在%rdx(高64)，%rax(低64)

idivl : 有符号除法，商存在%rax, 余数存在%rdx

## 控制

### 条件码

检测这些寄存器来执行条件分支指令

CF：进位，检测无符号操作的溢出

ZF：零标志。最近的操作得出的结果为0

SF：符号标志。最近的操作得到的结果为负数

OF：溢出标志。补码溢出

cmp根据两个操作数之差来设置条件码。与sub指令一致

test与add一致，除了只设置条件码而不改变目的寄存器的值。

testq %rax, %rax 测试%rax是负数，0，正数、

### 访问条件码

set

不会直接读取

1. 根据条件码的某种组合，将一个字节设置为0或者1
2. 可以条件跳转到程序的某个其他的部分
3. 可以有条件的传送数据

例如：sete D : 相等时设置

sete D (=/0)

setne D (!=/!0)

sets D (<0)

setns D (>=0)

有符号：

setg D ( >0)

setge D (>=0)

setl D (<0)

setle D (<= 0)

无符号

seta D (>0)

setae D (>=0)

setb D (<0)

setbe D (<=0)

### 跳转指令

jmp Label直接跳转

jmp *Operand间接跳转

je Label (=/0)

jne Label (!= / !0)

js Label & jns Label 负数&非负数

jg jge (> & >=)

jl jle ( < & <=)

ja jae (> & >=)

jb jbe (< & <=)

编码：将目标指令的地址与紧跟在跳转指令后面哪条指令的值之间的差作为编码。这些地址偏移量可以编码为1,2,4个字节。或者给出"绝对"地址，用四个字节直接指定目标，汇编器和连接器会选择适当的跳转目的指令。

### 条件控制

数据的条件转移：计算一个条件操作的两种结果，根据条件是否满足从中选取一个。

条件传送指令：条件满足时，把S复制到R

cmove S, R (=/0)

cmovne S, R (!=/!0)

cmovs S. R (<0)

cmovns S, R (>=0)

有符号

cmovg S, R (>0)

cmovge S, R (>0)

cmovl S, R (<)

cmovle S, R (<=)

无符号

cmova S, R (>)

cmovae S, R (>=)

cmovb S, R (<)

cmovbe S, R (<=)

处理器无需预测的结果就可以执行条件传送，处理器知识读源值，检查条件码，要么更新目的寄存器，要么保持不变。

### 循环

用条件测试和跳转组合起来实现循环的效果。

### switch

跳转表

优点：执行开关语句额时间与开关情况的数量无关。当开关情况数量比较多，并且值的范围比较小时，就会使用跳转表。

## 过程

# gdb

```bash
gdb proj # 启动
# 开始和停止
quit
run
kill
# 断点
break func
break * 0x789798
delete 1
delete
# 执行
stepi
stepi 4
next i
continue
finish
# 检查代码:反汇编
disas
disas func
disas 0x78978
disas 0x787987,0x789789 # 范围内
print /x $rip # 16进制输出pc的值
# 检查数据
print $rax # 十进制
print /x $rax # 十六进制
print /t $rax # 二进制
print 0x100 # 输出十进制表示
print /x 555
print /x ($rsp+8)
print * (long *) 0x89789789789
print * (long *) ($rsp+8)
x/2g 0x898098908 # 检查从地址开始的8字节
x/20bfunc # 检查func的前20个字节
# 有用的信息
info frame # 当前栈帧信息
info registers
help
```

## 内存越界引用和缓冲区溢出

在栈中分配某个字符数组来保存一个字符串，但是字符串长度超出了为数组分配的空间。

**对抗缓冲区溢出攻击**

1. 栈随机化

   程序开始时，在栈上分配一段0~n字节之间的随机大小的空间。但是要足够小。

2. 栈破坏检测

   在栈帧中任何局部缓冲区与栈状态之间存储一个特殊的金丝雀(哨兵)值，在程序每次运行时随机产生，攻击者没有简单的办法能够知道它是什么。在回复寄存器状态和从函数返回之前，程序检查这个值是否被该函数的某个操作或者该函数调用的某个函数的某个操作改变了，如果是，那么程序异常中止。

3. 限制可执行代码区域

# 链接

`gcc -Og -o prog main.c sum.c`

