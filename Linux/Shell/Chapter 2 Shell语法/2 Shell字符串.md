# Shell字符串

字符串是shell编程中常用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。

## 单双引号的区别

单双引号的区别：

* 双引号里可以有变量，单引号则原样输出；
* 双引号里可以出现转义字符，单引号则原样输出；
* 单引号字串中不能出现单引号；
* 单引号是不能识别变量，只会原样输出，单引号是不能转义的

即：

- 以单引号`‘`包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出。这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景。
- 以双引号`"`包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量名和命令原样输出。这种方式比较适合字符串中附带有变量和命令并且想将其解析后再输出的变量定义。

示例：

```shell
name="tom"
message='welcome ${name} !'
echo $message
message="welcome ${name} !"
echo $message
```

输出如下：

```
welcome ${name} !
welcom tom !
```

## 转义字符

| 转义字符 | 含义                             |
| -------- | -------------------------------- |
| \\       | 反斜杠                           |
| \a       | 警报，响铃                       |
| \b       | 退格（删除键）                   |
| \f       | 换页(FF)，将当前位置移到下页开头 |
| \n       | 换行                             |
| \r       | 回车                             |
| \t       | 水平制表符（tab键）              |
| \v       | 垂直制表符                       |

shell默认是不转义上面的字符的。需要加`-e`选项。

举个例子：

```bash
#!/bin/bash
a=11
echo -e "a is $a \n"
```

运行结果：

```bash
a is 11
```

这里 `-e` 表示对转义字符进行替换。如果不使用 `-e` 选项，将会原样输出：

```bash
a is 11\n
```

可以使用 echo 命令的 `-E` 选项禁止转义，默认也是不转义的；使用 `-n` 选项可以禁止插入换行符。

## 字符串

### 拼接字符串

```bash
#!/bin/bash

str1='i'
str2='love'
str3='you'

echo $str1 $str2 $str3
echo $str1$str2$str3
echo $str1,$str2,$str3
```

输出：

```
i love you
iloveyou
i,love,you
```

### 获取字符串长度

```bash
#!/bin/bash/

str='i love you'

echo ${#str}

# 输出：10
```

### 截取字符串

```bash
#!/bin/bash/

str='i love you'

echo ${str:1} # 从第1个截取到末尾。注意从0开始。
echo ${str:2:2} # 从第2个截取2个。
echo ${str:0} # 全部截取。
echo ${str:-3} # 负数无效，视为0。
```

输出：

```
love you
lo
i love you
i love you
```

### 查找字符串

```bash
#!/bin/bash/

str="i love you"

echo `expr index "$str" l`
echo `expr index "$str" you` #最后一个参数是字符，会对后面字符串每一个单独查找，返回最靠前的index
echo `expr index "$str" o`
echo `expr length "$str"` #字符串长度
echo `expr substr "$str" 1 6` #从字符串中位置1开始截取6个字符。索引是从0开始的。
```

输出：

```
3
4
4
10
i love
```

注意字符串变量需要加双引号。第2个例子里`you`虽然`y`的index是8，但是`o`在前面已经出现过，index是4，最终取所有字符里最靠前的index。

### 判断变量是否包含某个字符串

方法一：利用grep查找

```shell
strA="long string"
strB="string"
result=$(echo $strA | grep "${strB}")
if [[ "$result" != "" ]]
then
    echo "包含"
else
    echo "不包含"
fi
```

先打印长字符串，然后在长字符串中 grep 查找要搜索的字符串，用变量result记录结果，如果结果不为空，说明strA包含strB。如果结果为空，说明不包含。

方法二：利用字符串运算符

```shell
strA="hello world"
strB="hello"
if [[ $strA =~ $strB ]]
then
    echo "包含"
else
    echo "不包含"
fi
```

利用字符串运算符 =~ 直接判断strA是否包含strB。

方法三：利用通配符

```shell
A="helloworld"
B="hello"
if [[ $A == *$B* ]]
then
    echo "包含"
else
    echo "不包含"
fi
```

用通配符*号代理strA中非strB的部分，如果结果相等说明包含，反之不包含。

方法四：利用case in 语句

```shell
thisString="1 2 3 4 5"	# 源字符串
searchString="1 2"		# 搜索字符串
case $thisString in
    *"$searchString"*) echo  "包含";;
    *) echo  "不包含" ;;
esac
```

方法五：利用替换

```shell
STRING_A="helloworld"
STRING_B="hello"
if [[ ${STRING_A/${STRING_B}//} == $STRING_A ]]
    then
        echo  "不包含"
    else
       echo  "包含"
    fi
```

