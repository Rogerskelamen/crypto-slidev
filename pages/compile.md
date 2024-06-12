---
transition: slide-left
---

# 编译执行的命令行操作

—— 浮在表面的工作

<br>

以编译`Programs/Source/tutorial.mpc`并使用Rep3协议执行为例

如果不需要额外的参数：

```bash
./compile.py tutorial
Scripts/ring.sh tutorial
```

<br>

- 第一行会分别在`Programs/Schedules`和`Programs/Bytecode`下生成<u>schedule文件</u>和<u>二进制的字节码</u>

- 第二行调用自动脚本通过编译好的对应协议的二进制文件来执行字节码

  这里对应的协议二进制文件就是`replicated-ring-party.x`

<!-- 还有一种直接编译运行在一条命令的方式：`Scripts/compile-run.py -E ring tutorial` -->



---
transition: slide-left
class: text-base
---

# 程序底层的操作
—— 编译阶段的剖析

<div grid="~ cols-2 gap-12">
<div>

`compile.py`代码如下：

```python {3-19}{lines:true,maxHeight:'340px'}
from Compiler.compilerLib import Compiler

def compilation(compiler):
    prog = compiler.compile_file()

    if prog.public_input_file is not None:
        print(
            "WARNING: %s is required to run the program" % prog.public_input_file.name
        )

def main(compiler):
    compiler.prep_compile()
    if compiler.options.profile:
        import cProfile
        p = cProfile.Profile().runctx("compilation(compiler)", globals(), locals())
        p.dump_stats(compiler.args[0] + ".prof")
        p.print_stats(2)
    else:
        compilation(compiler)

if __name__ == "__main__":
    compiler = Compiler()
    main(compiler)
```
</div>
<div>

1. `prep_compile`会调用`Compiler`类中的`build`函数，来进行必要的初始化和创建`Program`类的实例

2. `build`会调用`build_program()`将程序实体给到`self.prog`，再调用`build_vars()`将所有**需要的**指令加入到`self.VARS`中去

2. `compilation()`调用`compiler.compile_file()`，后者会使用python的`compile`函数编译源文件，并将结果信息给到`self.VARS`

3. 编译完成后还会调用`finalize_compile()`打印信息，其中调用函数`prog.finalize()`来生成schedule文件和二进制字节码文件（**真正的有效输出**）

</div>
</div>



---
transition: slide-left
class: text-base
---

# 程序底层的操作
—— 执行阶段的剖析

还是以Rep3协议执行程序为例，`Scripts/ring.sh`内容如下：

```bash
#!/usr/bin/env bash

HERE=$(cd `dirname $0`; pwd)
SPDZROOT=$HERE/..

export PLAYERS=3

. $HERE/run-common.sh

run_player replicated-ring-party.x $* || exit 1
```

这个脚本主要做了两件事：

1. 设置环境变量，其中<u>比较重要的是</u>`PLAYERS`，它决定了有多少个参与方（如果有多个参与方就需要使用`Scripts/setup-ssl.sh`来生成各方的SSL密钥）

2. 载入<u>`Scripts/run-common.sh`</u>脚本，使用其中定义的`run_player`方法来执行程序



---
transition: slide-left
class: text-base
---

# 程序底层的操作
—— 执行阶段的剖析

<div grid="~ cols-2 gap-10">
<div>

在`run-common.sh`中的`run-player`主要代码：

```bash {all}{lines:true,maxHeight:'320px'}
for i in $(seq 0 $[players-1]); do
  if test "$GDB_PLAYER" -a $i = "$GDB_PLAYER"; then
  my_prefix=gdb_front
  else
  my_prefix=$prefix
  fi
  front_player=${GDB_PLAYER:-0}
  >&2 echo Running $my_prefix $SPDZROOT/$bin $i $params
  log=logs/$log_prefix$i
  $my_prefix $SPDZROOT/$bin $i $params 2>&1 |
  {
    if test "$BENCH"; then
    if test $i = $front_player; then tee -a $log; else cat >> $log; fi;
    else
    if test $i = $front_player; then tee $log; else cat > $log; fi;
    fi
  } &
  codes[$i]=$! # 保存进程ID
done
```

</div>
<div>

- 这个代码块就是`run-player`函数的主要逻辑，其按照参与方个数来定义执行的进程数，并指定执行的协议

- 最核心的代码是第10行`$my_prefix $SPDZROOT/$bin $i $params`
  - 其中`$SPDZROOT/$bin`指的就是例子中的`replicated-ring-party.x`
  - `$i`代表参与者的编号
  - `$params`表示其他参数，包括程序名和各种执行参数

- 脚本除了完成代码块中的工作，还会将命令行输出重定向到`logs/`文件目录中，最后等待所有参与方进程完成

</div>
</div>