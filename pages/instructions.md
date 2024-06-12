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

    定义并实现了很多和指令相关的方法（包括程序指令执行的逻辑），并且针对不同的指令定义了不同的`X`宏


---
layout: two-cols
transition: slide-left
---

# 指令类型

- 存取指令：Load and store
- 机器指令：Machine
- 基本算术：Basic arithmetic
- 位运算：Bitwise operations
- 带立即数的运算：Arithmetic with immediate
- 移位指令：Shift instructions
- 数据访问：Data access instructions
- 输入输出：I/O
- 整型操作：Integer operations
- 明文比较：Clear comparison
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

在`Processor/instructions.h`中可以查看`ldsi`和`adds`的语义（主要分析这两个指令）：

```cpp
X(LDSI, auto dest = &Procp.get_S()[r[0]]; \
        auto tmp = sint::constant(int(n), Proc.P.my_num(), Procp.MC.get_alphai()), \
        *dest++ = tmp) \
X(ADDS, auto dest = &Procp.get_S()[r[0]]; auto op1 = &Procp.get_S()[r[1]]; \
        auto op2 = &Procp.get_S()[r[2]], \
        *dest++ = *op1++ + *op2++) \
```

`X`的宏定义位于`Processor/Instruction.hpp`的`Program::execute`方法中：

<!-- 实际上Instruction::execute方法中也有 -->

<div grid="~ cols-2 gap-12">
<div>

```cpp
#define X(NAME, PRE, CODE) \
        case NAME: { PRE; for (int i = 0; i < size; i++) { CODE; } } break;
```

例如将`LDSI`指令带入到宏，会得到右边的宏展开：

</div>
<div>

```cpp
case LDSI: { // NAME
    auto dest = &Procp.get_S()[r[0]]; // PRE
    auto tmp = sint::constant(int(n), Proc.P.my_num(), Procp.MC.get_alphai());
    for (int i = 0; i < size; i++) {
        *dest++ = tmp; // CODE
    }
} break;
```

</div>
</div>



---
transition: slide-left
class: text-base
---

# 程序执行的基本框架
—— 程序的底层行为

实际上`Program::execute`方法中包含所有指令的`X`宏，整体结构就是通过switch根据指令名选择要执行的操作

重新整理一下逻辑，程序的执行`execute`核心就是以下的步骤：

```cpp {all}{lines:true}
// 1. 准备好全局变量
unsigned int size = p.size();
Proc.PC=0;
...
while (Proc.PC<size) // 2. 主循环，执行完所有指令后退出
{   // 3. 更新每次执行的局部变量
    auto& instruction = p[Proc.PC];
    auto& r = instruction.r; // 寄存器
    auto& n = instruction.n; // 立即数的值
    ...
    Proc.PC++; // 4. PC指针加一
    // 5. 根据指令名选择的switch
    switch(instruction.get_opcode()) {
        case LDSI: {
            // preprocess and execution behavior
        } break;
        ... // 更多的指令。。。
```

<!-- 这就是程序执行时的思路 -->



---
transition: slide-left
class: text-base
---

# 指令行为
—— 案例驱动的指令分析

回过头来再看看`LDSI`和`ADDS`到底在做什么？

1. 预处理

    ```cpp
    auto dest = &Procp.get_S()[r[0]];
    auto tmp = sint::constant(int(n), Proc.P.my_num(), Procp.MC.get_alphai());
    ```

    从子处理器(`SubProcessor`)中获取s寄存器的首地址，并将该地址赋值给了dest

    根据立即数的值`n`创建了一个`sint`的密文数据，将其赋值给了tmp

2. 执行

    ```cpp
    for (int i = 0; i < size; i++) {
        *dest++ = tmp; // CODE
    }
    ```

    从dest地址开始的`size`长度的连续地址空间，均加载同样的密文数据值

<!-- 个人猜测是s寄存器的大小随程序的长度而增长 -->



---
transition: slide-left
---

# 指令行为
—— 案例驱动的指令分析

针对不同的指令类型，会有不同的`X`宏定义（`Processor/Instruction.hpp`）：

```cpp
#define X(NAME, PRE, CODE) \
        case NAME: { PRE; for (int i = 0; i < size; i++) { CODE; } } break;
        ARITHMETIC_INSTRUCTIONS
#undef X
#define X(NAME, PRE, CODE) case NAME:
        CLEAR_GF2N_INSTRUCTIONS
        instruction.execute_clear_gf2n(Proc2.get_C(), Proc.machine.M2.MC, Proc); break;
#undef X
#define X(NAME, PRE, CODE) case NAME:
        REGINT_INSTRUCTIONS
        instruction.execute_regint(Proc, Proc.machine.Mi.MC); break;
#undef X
#define X(NAME, CODE) case NAME: CODE; break;
        COMBI_INSTRUCTIONS
#undef X
```

每种类型的指令根据语义的不同，进行出不同的宏展开，最终实现程序执行

<!-- 这就是整个指令系统在程序执行中的作用 -->