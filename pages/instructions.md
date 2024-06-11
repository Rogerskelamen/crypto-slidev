---
transition: slide-left
class: text-base
---

# 指令系统
—— 程序的基本单元

与指令集相关的代码：

- `Compiler/instructions.py`

    以类的形式定义了所有指令的名称和参数形式，也被用来当作生成指令集的文档的基础

- `Compiler/instuctions_base.py`

    定义了所有指令的二进制形式，也就是字节码，这和`Processor/Instruction.h`中的定义完全相同

- `Processor/instructions.h`

    用宏的方式定义了所有指令的行为，即需要执行的操作

- `Processor/Instruction.hpp`

    定义并实现了很多和指令相关的方法，并且针对不同的指令定义了不同的`X`宏


---
layout: two-cols
transition: slide-left
---

# 指令类型

- 存取指令：Load and store
- 硬件指令：Machine
- 基本算术：Basic arithmetic
- 位运算：Bitwise operations
- 带立即数的运算：Arithmetic with immediate
- 移位指令：Shift instructions
- 数据访问：Data access instructions
- 输入输出：I/O
- 整型操作：Integer operations
- 比较清除：Clear comparison
- 跳转指令：Jumps
- 类型转换：Conversion
- 其他：Other instructions

::right::

```python
opcodes = dict(
    # Emulation
    CISC = 0,
    # Load/store
    LDI = 0x1,
    LDSI = 0x2,
    LDMC = 0x3,
    LDMS = 0x4,
    STMC = 0x5,
    STMS = 0x6,
    LDMCI = 0x7,
    LDMSI = 0x8,
    STMCI = 0x9,
    STMSI = 0xA,
    MOVC = 0xB,
    MOVS = 0xC,
    PROTECTMEMS = 0xD,
    PROTECTMEMC = 0xE,
    PROTECTMEMINT = 0xF,
    LDMINT = 0xCA,
    STMINT = 0xCB,
    LDMINTI = 0xCC,
    STMINTI = 0xCD,
    PUSHINT = 0xCE,
    POPINT = 0xCF,
    ...
```

<!-- 这里的指令类型是按instructions.py中的分类来的，毕竟是作为文档的代码块 -->

---
transition: slide-left
class: text-base
---

# 指令行为
—— 案例驱动的指令分析

这里以一个样例程序的执行流为例，来完成指令的分析

```python
a = sint(1)
b = sint(2)

print_ln('%s', (a + b).reveal())
```

通过编译之后的字节码反编译结果如下：

```text
ldsi s1, 1 # 0
ldsi s2, 2 # 1
adds s0, s1, s2 # 2
asm_open 3, 1, c0, s0 # 3
print_reg_plain c0 # 4
print_char 10 # 5
...
```

下面还有`use 0, 7, 1`等指令，那些是用来指示虚拟机进行资源分配等预处理操作的信息

对程序的主体逻辑没有太大影响，暂不列出

<!--
后面省略掉的指令:
use 0, 7, 1 # 6
ldmc c0, 8191 # 7
gldmc cg0, 8191 # 8
ldmint ci0, 8191 # 9
ldms s0, 8191 # 10
gldms sg0, 8191 # 11
active 1 # 12
-->



---
transition: slide-left
class: text-base
---

# 程序的基本语法
—— mpc的编程语法

程序中的`sint()`类型和`reveal()`函数都不是python中的常规语法，在这里需要补充一下这方面的语法知识

1. 数据类型

    在`Compiler/types.py`中定义了程序的<u>基本数据类型</u>和<u>容器数据类型</u>

    其中基本类型有：sint, cint, regint, sfix, cfix, sfloat 等

    容器类型有：MemValue, Array, Matrix, MultiArray

2. `reveal()`

    对于`s`类型（secret text）的数据，不能直接进行打印，需要使用`reveal`函数解除加密，对应于指令就是`asm_open`；与之对应的，`c`类型（clear text）数据可以直接打印

3. 寄存器

    在`Compiler/instructions_base.py`中定义了5种寄存器类型，其中对于`ClearModp`是`c`寄存器，而`SecretModp`是`s`寄存器，本例中的`sint`就使用`s`寄存器



---
transition: slide-left
class: text-base
---

# 指令行为
—— 案例驱动的指令分析

在`Processor/instructions.h`