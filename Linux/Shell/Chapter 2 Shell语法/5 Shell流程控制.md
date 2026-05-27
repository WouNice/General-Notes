# 流程控制

Shell 命令默认以从上到下顺序执行，除非一些特殊的运算符和复合命令作用。

即使某条命令执行异常，也会继续执行后面的命令。这点与大部分编程语言不同。

## 条件结构

### if 语句

if 语句语法格式：

```bash
if condition
then
    command1
    command2
    ...
    commandN
fi
```

写成一行（适用于终端命令提示符）：

```bash
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

每个 `if/elif` 分支，先执行 `condition`，如果最后退出码为 0，即条件为 true，执行对应的分支命令，否则进入下一分支。

Shell 没有表达式语法，使用命令执行后的退出码进行条件判断，退出码为 0 表示 true，非 0 表示 false。

`test` 命令是一个专门用于条件判断的内置命令，它支持一系列条件表达式，当条件成立时退出码为 0，不成立时为 1。

```bash
test expr

# 或者使用简写

[ expr ]

# 还有一种拓展的test，在原有的基础上支持了正则匹配

[[ expr ]]
```

### if else 语句

if else 语法格式：

```bash
if condition
then
    command1
    command2
    ...
    commandN
else
    command
fi
```

### if elif else 语句

if else-if else 语法格式：

```bash
if condition1
then
    command1
elif condition2
then
    command2
else
    commandN
fi
```

以下实例判断两个变量是否相等：

```bash
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

输出结果：

```bash
a 小于 b
```

if else 语句经常与 test 命令结合使用，如下所示：

```bash
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi
```

输出结果：

```bash
两个数字相等!
```

### case

case 语句为多选择语句。可以用 case 语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。

```text
case word in
    [ [(] glob-pattern  ) commands ;;]…
esac
```

- 使用 glob 模式匹配，不是正则。
- 模式用括号包裹，括号左边经常省略，右括号不能省略。
- 子句必须用 `;;`，`;&` 或 `;;&` 结尾（不可省略），它们会影响执行该子句之后的行为。
  - `;;` 同 break，退出 case 结构，不再往下执行。
  - `;&` 表示直接执行下一个子句的命令（不管是否匹配），与其它语言漏掉 `break` 类似。
  - `;;&` 表示接着进入下一子句的匹配，好像没有子句命中过一样。
- 最后的 `...` 表示可以出现多个 case 子句。
- 可以在最后一个子句中使用模式 `*` 作为 `default` 分支。

case 语句格式如下：

```text
case 变量值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```

case 工作方式如上所示。取值后面必须为单词 in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至`;;`。

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

下面的脚本提示输入 1 到 4，与每一种模式进行匹配：

```bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

### `&&` 和 `||` 运算符

```bash
test-command && command1 || command2
```

是一种简单实用的条件执行写法。如果 `test-command` 为真（状态码为 0）就执行 `command1`，否则，执行 `command2`。

## 循环结构

### for 循环

`for` 循环有两种语法格式：传统的 `for...in` 格式和 C 风格的 `for(( expr1; expr2; expr3 ))` 格式。

#### for...in

for...in 循环格式为：

```bash
for variable [in lists];
do
  commands;
done
```

当变量值在 lists 列表里，for 循环即执行一次所有命令，使用变量名获取列表中的当前取值。命令可为任何有效的 shell 命令和语句。in 列表可以包含替换、字符串和文件名。

`in words` 部分内容可以省略，如果省略，默认为 `in "$@"`，即对所有的命令参数做遍历。

举个例子：

```bash
for option in A B C D; do echo -n $option; done

# ABCD

```

`words` 部分会执行展开，可以利用这种特性快速生成遍历内容，看两个简单例子：

1、利用大括号展开重写上例：

```bash
for option in {A..D}; do echo -n $option; done
```

2、利用文件名展开，遍历当前目录下所有的 `.txt` 文件：

```bash
for txt_file in *.txt; do echo $txt_file; done
```

#### for(( expr1; expr2; expr3 ))

for(( expr1; expr2; expr3 ))循环格式为：

```bash
for(( expr1; expr2; expr3 ));
do
  commands
done
```

`expr1`，`expr2`，`expr3` 都是算术表达式。注意这里使用双括号，与算术表达式语法相同。

执行结构与 C 语言 for 语句一样。等效于：

```bash
(( expr1 ))
while (( expr2 )); do
  commands
  (( expr3 ))
done
```

看个例子：

```bash
for(( option=1; option<1+4; option++ ));do echo -n $i; done

# 1234

```

### while 语句

while 循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。其格式为：

```bash
while condition
do
    command
done
```

以下是一个基本的 while 循环，测试条件是：如果 int 小于等于 5，那么条件返回真。int 从 0 开始，每次循环处理时，int 加 1。运行上述脚本，返回数字 1 到 5，然后终止。

```bash
#!/bin/bash

int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

运行脚本，输出：

```bash
1
2
3
4
5
```

以上实例使用了 let 命令，它用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量。

while 循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量 FILM，按结束循环。

```bash
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read FILM
do
    echo "是的！$FILM 是一个好网站"
done
```

运行脚本，输出类似下面：

```bash
按下 <CTRL-D> 退出
输入你最喜欢的网站名:example教程
是的！example教程 是一个好网站
```

### 无限循环

无限循环语法格式：

```bash
while:
do
    command
done
```

或者

```bash
while true
do
    command
done
```

或者

```bash
for (( ; ; ))
```

### until 循环

until 循环执行一系列命令直至条件为 true 时停止。

until 循环与 while 循环在处理方式上刚好相反。

until 语法格式：

```bash
until condition
do
    command
done
```

condition 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。

以下实例我们使用 until 命令来输出 0 ~ 9 的数字：

```bash
#!/bin/bash

a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

运行结果：

输出结果为：

```text
0
1
2
3
4
5
6
7
8
9
```

## 跳出循环

在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell 使用两个命令来实现该功能：break 和 continue。

### break命令

break 命令允许跳出所有循环（终止执行后面的所有循环）。

下面的例子中，脚本进入死循环直至用户输入数字大于 5。要跳出这个循环，返回到 shell 提示符下，需要使用 break 命令。

```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```

执行以上代码，输出结果为：

```bash
输入 1 到 5 之间的数字:3
你输入的数字为 3!
输入 1 到 5 之间的数字:7
你输入的数字不是 1 到 5 之间的! 游戏结束
```

### continue

continue 命令与 break 命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。

对上面的例子进行修改：

```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

运行代码发现，当输入大于 5 的数字时，该例中的循环不会结束，语句 `echo "游戏结束`" 永远不会被执行。

### case ... esac

`case ... esac` 与其他语言中的 `switch ... case` 语句类似，是一种多分枝选择结构，每个 case 分支用右圆括号开始，用两个分号 `;;` 表示 break，即执行结束，跳出整个 `case ... esac` 语句，`esac`（就是 `case `反过来）作为结束标记。

`case ... esac` 语法格式如下：

```text
case 值 in
模式1)
    command1
    command2
    command3
    ;;
模式2）
    command1
    command2
    command3
    ;;
*)
    command1
    command2
    command3
    ;;
esac
```

case 后为取值，值可以为变量或常数。

值后为关键字 in，接下来是匹配的各种模式，每一模式最后必须以右括号结束，模式支持正则表达式。
