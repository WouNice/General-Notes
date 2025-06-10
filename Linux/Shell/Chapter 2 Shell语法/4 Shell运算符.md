# shell运算符

Shell中的运算符和其他语言类似，都支持多种运算符，主要包括以下几种：

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

需要注意，原生bash不支持简单的数学运算，但是可以通过相关命令来实现，例如 `awk`和 `expr`，其中 `expr` 最常用。

`expr` 是一款表达式计算工具，使用它能完成表达式的求值操作。

例如，两个数相加：

```shell
val=`expr 1 + 2`
echo "两数之和为 : $val"
```

>   注意：表达式和运算符之间要有空格，例如 `1+2` 是不对的，必须写成 `1 + 2`。
>

## 算术运算符

Shell 采用和 C 语言相同的运算符和运算优先级。

### 算术运算

支持的运算包括：

- 自增、自减（包括前置后置）：`i++`，`i--`，`++i`，`--i`。
- 正负：`+i`，`-i`。
- 基本算术运算：`+ - \* / %`
- 比较：`== != > < <= >=`
- 位运算：`~ & | ^ << >>`
- 逻辑运算：`! && ||`
- 三元运算：`expr1?expr2:expr3`
- 赋值：`= += -=`

在逻辑运算和比较运算时，用 0 表示真，1 表示假。

### 支持算术表达式的几个命令

不能像这样直接使用算术表达式：

```
# 错误用法
sum=1+1;
echo $sum    # 1+1
```

只有在使用特定的命令时，才会以算术表达式的形式解析。主要有：

1、算术展开 `$(( expr ))`

```
sum=$((expr)); echo $sum
```

2、`let` 命令

```
let sum=1+1; echo $sum
```

3、`let` 简写形式` (( expr ))`

```
(( sum=1+1 )); echo $sum
```

4、`declare -i`

```
declare -i sum=1+1; echo $sum
```

`(( expr ))` 和 `let expr` 命令会在表达式结果非 0 时，设置退出码为 0，反之，退出码为 1。因此，它们也经常作为 `if` 命令的判断。

### 算术运算示例

下表列出了常用的算术运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                          | 举例                           |
| :----- | :-------------------------------------------- | :----------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。     |
| -      | 减法                                          | `expr $a - $b` 结果为 -10。    |
| *      | 乘法                                          | `expr $a \* $b` 结果为 200。   |
| /      | 除法                                          | `expr $b / $a` 结果为 2。      |
| %      | 取余                                          | `expr $b % $a` 结果为 0。      |
| =      | 赋值                                          | `a=$b` 将把变量 b 的值赋给 a。 |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | `[ $a == $b ]` 返回 false。    |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | `[ $a != $b ] `返回 true。     |

**注意：**

- 条件表达式要放在方括号之间，并且要有空格，例如: `[$a==$b]`是错误的，必须写成 `[$a == $b]`。
- 乘号(`*`)前边必须加反斜杠(`\`)才能实现乘法运算
- 完整的表达式要被 \` ` 包含。

示例：

```shell
#!/bin/bash

a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a 等于 b"
fi
if [ $a != $b ]
then
   echo "a 不等于 b"
fi
```

脚本执行后，输出结果如下：

```
a + b : 30
a - b : -10
a * b : 200
b / a : 2
b % a : 0
a 不等于 b
```

## 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                                  | 举例                         |
| :----- | :---------------------------------------------------- | :--------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | `[ $a -eq $b ]` 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | `[ $a -ne $b ] `返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | `[ $a -gt $b ] `返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | `[ $a -lt $b ] `返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | `[ $a -ge $b ] `返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | `[ $a -le $b ]` 返回 true。  |

示例：

```shell
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
else
   echo "$a -eq $b: a 不等于 b"
fi
if [ $a -ne $b ]
then
   echo "$a -ne $b: a 不等于 b"
else
   echo "$a -ne $b : a 等于 b"
fi
if [ $a -gt $b ]
then
   echo "$a -gt $b: a 大于 b"
else
   echo "$a -gt $b: a 不大于 b"
fi
if [ $a -lt $b ]
then
   echo "$a -lt $b: a 小于 b"
else
   echo "$a -lt $b: a 不小于 b"
fi
if [ $a -ge $b ]
then
   echo "$a -ge $b: a 大于或等于 b"
else
   echo "$a -ge $b: a 小于 b"
fi
if [ $a -le $b ]
then
   echo "$a -le $b: a 小于或等于 b"
else
   echo "$a -le $b: a 大于 b"
fi
```

脚本执行后，输出结果如下：

```
10 -eq 20: a 不等于 b
10 -ne 20: a 不等于 b
10 -gt 20: a 不大于 b
10 -lt 20: a 小于 b
10 -ge 20: a 小于 b
10 -le 20: a 小于或等于 b
```

## 布尔运算符

假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                                | 举例                                       |
| :----- | :-------------------------------------------------- | :----------------------------------------- |
| `!`    | 非运算，表达式为 true 则返回 false，否则返回 true。 | `[ ! false ] `返回 true。                  |
| `-o`   | 或运算，有一个表达式为 true 则返回 true。           | `[ $a -lt 20 -o $b -gt 100 ] `返回 true。  |
| `-a`   | 与运算，两个表达式都为 true 才返回 true。           | `[ $a -lt 20 -a $b -gt 100 ]` 返回 false。 |

示例：

```shell
#!/bin/bash

a=10
b=20

if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a == $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a 小于 100 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 100 或 $b 大于 100 : 返回 false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a 小于 5 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 5 或 $b 大于 100 : 返回 false"
fi
```

脚本执行后，输出结果如下：

```
10 != 20 : a 不等于 b
10 小于 100 且 20 大于 15 : 返回 true
10 小于 100 或 20 大于 100 : 返回 true
10 小于 5 或 20 大于 100 : 返回 false
```

## 逻辑运算符

假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明       | 举例                                        |
| :----- | :--------- | :------------------------------------------ |
| `&&`   | 逻辑的 AND | `[[ $a -lt 100 && $b -gt 100 ]]` 返回 false |
| `||`   | 逻辑的 OR  | `[[ $a -lt 100 || $b -gt 100 ]]` 返回 true  |

示例：

```shell
#!/bin/bash

a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

脚本执行后，输出结果如下：

```
返回 false
返回 true
```

## 字符串运算符

假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符 | 说明                                         | 示例                       |
| :----- | :------------------------------------------- | :------------------------- |
| `=`    | 检测两个字符串是否相等，相等返回 true。      | `[ $a = $b ]` 返回 false。 |
| `!=`   | 检测两个字符串是否相等，不相等返回 true。    | `[ $a != $b ]` 返回 true。 |
| `-z`   | 检测字符串长度是否为0，为0返回 true。        | `[ -z $a ]` 返回 false。   |
| `-n`   | 检测字符串长度是否不为 0，不为 0 返回 true。 | `[ -n "$a" ]` 返回 true。  |
| `$`    | 检测字符串是否为空，不为空返回 true。        | `[ $a ]` 返回 true。       |

示例：

```shell
#!/bin/bash

a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a 等于 b"
else
   echo "$a = $b: a 不等于 b"
fi
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
if [ -n "$a" ]
then
   echo "-n $a : 字符串长度不为 0"
else
   echo "-n $a : 字符串长度为 0"
fi
if [ $a ]
then
   echo "$a : 字符串不为空"
else
   echo "$a : 字符串为空"
fi
```

脚本执行后，输出结果如下：

```
abc = efg: a 不等于 b
abc != efg : a 不等于 b
-z abc : 字符串长度不为 0
-n abc : 字符串长度不为 0
abc : 字符串不为空
```

## 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

文件测试运算符列表：

| 操作符    | 说明                                                         | 举例                        |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| `-b file` | 检测文件是否是块设备文件，如果是，则返回 true。              | `[ -b $file ]` 返回 false。 |
| `-c file` | 检测文件是否是字符设备文件，如果是，则返回 true。            | `[ -c $file ] `返回 false。 |
| `-d file` | 检测文件是否是目录，如果是，则返回 true。                    | `[ -d $file ] `返回 false。 |
| `-f file` | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | `[ -f $file ]` 返回 true。  |
| `-g file` | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | `[ -g $file ] `返回 false。 |
| `-k file` | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | `[ -k $file ] `返回 false。 |
| `-p file` | 检测文件是否是有名管道，如果是，则返回 true。                | `[ -p $file ]` 返回 false。 |
| `-u file` | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | `[ -u $file ] `返回 false。 |
| `-r file` | 检测文件是否可读，如果是，则返回 true。                      | `[ -r $file ]` 返回 true。  |
| `-w file` | 检测文件是否可写，如果是，则返回 true。                      | `[ -w $file ]` 返回 true。  |
| `-x file` | 检测文件是否可执行，如果是，则返回 true。                    | `[ -x $file ]` 返回 true。  |
| `-s file` | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | `[ -s $file ] `返回 true。  |
| `-e file` | 检测文件（包括目录）是否存在，如果是，则返回 true。          | `[ -e $file ]` 返回 true。  |

其他检查符：

- `-S`：判断某文件是否 socket。
- `-L`：检测文件是否存在并且是一个符号链接。

变量 file 表示文件 `/var/www/example/test.sh`，它的大小为 100 字节，具有 `rwx `权限。下面的代码，将检测该文件的各种属性：

```shell
#!/bin/bash

file="/var/www/example/test.sh"

if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

脚本执行，输出结果如下：

```
文件可读
文件可写
文件可执行
文件为普通文件
文件不是个目录
文件不为空
文件存在
```
