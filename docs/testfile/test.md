# Linux中的管道与连接符号


目录
--- 

[TOC]

## 1. 背景

最近在学习Linux中的管道与连接符号，写一篇笔记进行总结。

## 2. 输入输出重定向

### 2.1. 管道符号 `|`

```shell
Command A | Command B
```

将前`A`命令的输出作为`B`命令的输入。

常与筛选命令`grep`搭配使用，如

```shell
ps aux | grep mysql
```

在所有进程中查询含有`mysql`关键字的信息。

```shell
ps aux | grep mysql | less
```

还可以将输入导入`less`命令，`less` 是一个分页工具，它允许你一页一页地查看信息。

### 2.2. 输出重定向符号 `>`

```shell
Command A > File B
```

将前`A`命令的输出写入`B`文件，覆盖原文件内容。

如在`hello.txt`中写入`Hello World!`。

```shell
echo Hello World! > hello.txt
```

### 2.3. 输出重定向符号 `>>`

```shell
Command A >> File B
```

将前`A`命令的输出追加写入`B`文件，不覆盖原文件的内容。

```shell
echo Hello World! >> hello.txt
```

将 `Hello World!` 追加到 `hello.txt` 后面。

注：`cat hello.txt >> hello.txt`会导致无限循环。

### 2.4. 输入重定向符号 `<`

```shell
Command A < File B
```

将 `B`文件的内容作为命令 `A` 的输入。

先创建 `Python` 文件 `test.py`，功能为输出用户输出的内容。

```shell
echo 'print(input())' > test.py
```

将 `hello.txt` 作为 `test.py` 的输入。

```shell
python test.py < hello.txt
```

结果会输出 `Hello World!`。

## 3. 连接符号

### 3.1. 连接符号 `;`

```shell
Command A ; Command B
```

在一行写入多个命令时，使用`;`进行分割，不论 `A` 命令是否执行成功，`B`命令都会继续执行。

比如下面的命令，A命令执行失败，B命令执行成功。

```shell
ee ; echo Hello World!
```

输出为

```shell
zsh: command not found: ee
Hello World!
```

### 3.2. 后台执行符号 `&`

```shell
Command A &
```

命令`A`会进入后台执行，当用于挂载后台服务，或执行耗时任务时可能会用到。

如下面的命令，会创建一个后台进程，在60秒后结束。

```shell
sleep 60 &
```

命令行会输出`[1] 51277`，表示进程的`PID`为 `51277`，60秒后会输出`[1]    51277 done sleep 60`，表示进程结束。

### 3.3. 逻辑与符号 `&&`

```shell
Command A && Command B
```

若`A`命令执行成功，继续执行`B`命令，否则不执行`B`命令。

如下面的命令会在60秒后输出`Hello World!`。

```shell
sleep 60 && echo Hello World!
```

而接下里这条命令只会输出命令`A`的错误信息。

```shell
ee && echo Hello World!
```

输出为`zsh: command not found: ee`。

### 3.4. 逻辑或符号 `||`

```shell
Command A || Command B
```

如果命令 `A` 执行失败，则执行 `B`。

```shell
ee || echo Hello World!
```

输出为

```shell
zsh: command not found: ee
Hello World!
```

## 4. 子命令符号

### 4.1. 子命令符号 `()`

```shell
(Command A)
```

创建一个子 `shell` 去执行命令 `A`，在`A`中切换文件夹不会影响当前`shell`工作路径。

```shell
(cd .. ; ls)
```

这条命令会返回上层目录的内容，但工作路径仍然在当前目录。

```shell
Command A $(Command B)
```

表示返回命令`B`的结果，并将结果作为命令`A`的参数继续执行。

```shell
$(echo echo) B
```

输出为 `B`。

```shell
echo A $(echo B)
```

输出为 `A B`。

### 4.2. 表达式计算符号 `(())`

```shell
((Calculation A))
```

计算表达式`A`的值。

如果要作为变量的话，需要加上`$`。如`echo $((3+2))`，输出为`5`。

常与 `if` 和 `for` 命令配合使用。

```shell
if ((3>2)) echo true
```

```shell
for ((i=1;i<=16;i++));do echo $i; done
```

---
