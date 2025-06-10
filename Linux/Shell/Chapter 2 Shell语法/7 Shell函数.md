# 函数

## 函数的定义

shell中函数的语法可以表示成：

```shell
fname(){
  commands
  return
}
```

还有一种不推荐的写法 `function fname(){}`，目前已经废弃。

```shell
[ function ] funname [()]
{
    action;
    [return int;]
}
```

说明：

- 可以带function fun() 定义，也可以直接fun() 定义，不带任何参数。
- 调用函数只需要给出函数名，不需要加括号。
- 函数返回值，可以显式增加return语句；如果不加，会将最后一条命令运行结果作为返回值。
- 和其他语言类似，内置命令 `return` 用于退出函数，函数体最后一行的 `return` 可省略。
- 函数的定义本身也是一个命令（关键字），它在执行环境中创建一个函数名到函数体的引用。除非发生语法错误，函数定义的退出码总是为 0。
- 根据脚本的顺序执行特点，函数的定义必须位于其使用之前。

## 函数的使用

Shell 函数是可以看成命令，执行函数和执行其它命令是一样的。

```shell
fname [arguments...]
```

下面定义一个带有return语句的函数：

```shell
#!/bin/bash

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```

输出类似下面：

```
这个函数会对输入的两个数字进行相加运算...
输入第一个数字：
1
输入第二个数字：
2
两个数字分别为 1 和 2 !
输入的两个数字之和为 3 !
```

函数返回值在调用该函数后通过 `$?` 来获得。

## 函数内的位置变量

执行函数时，位置变量 `$N`(N>0) 和 特殊变量 `$#`, `$@`，`$*` 会被赋值成调用函数时的参数对应值，执行完成后再恢复。

```shell
#!/bin/bash

func(){
    # $0 仍然指向脚本文件名称
    echo "\$0 = $0"
    # 其它位置参数被更新成函数调用时的参数
    echo  "参数个数 $#, 分别为 $@"
    # 函数的名称存储在环境变量 FUNCNAME 中
    echo $FUNCNAME
}

# 给这个函数传参执行
func 1 2 3
```

执行该文件后输出：

```txt
$0 = function-position-parameters.sh
参数个数 3，分别为 1 2 3
func
```

注意，`$10` 不能获取第十个参数，获取第十个参数需要`${10}`。当`n>=10`时，需要使用`${n}`来获取参数。

## 函数的返回值

Shell 函数返回值只能是整数，一般用来表示函数执行成功与否，0表示成功，其他值表示失败。return后跟数值`n(0-255)`。如果 return 其他数据，比如一个字符串，往往会得到错误提示：`numeric argument required`。

如果一定要让函数返回字符串，那么可以先定义一个变量，用来接收函数的计算结果，脚本在需要的时候访问这个变量来获得函数返回值。

```shell
#!/bin/bash

function hello(){
	echo "hello world";
}

str=$(hello)

echo $str
```

运行结果：

```
hello world
hello world
```
