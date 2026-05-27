# shell数组

Shell 的数组和其他语言不同，Shell 数组只支持一维数组，并且不用限定数组大小。Shell 数组和众多语言一样，数组的下标都是从 0 开始的。

## 定义数组

a) 把一个数组常量赋值给变量时，变量就会自动被声明为数组：

```bash
arr=(element0 element1 element2 )
```

b) 在数组常量内指定下标，可以实现对特定位置的元素进行赋值：

```bash
# arr[0],arr[3] 为空
arr=([1]=element1 element2 [4]=element3)
```

c) 直接对数组元素赋值，也会创建一个新的数组。

```text
arr[0]=element0
```

> 这里的下标数你可以任意指定，范围没有限制，而且下标并不是必须连续。

以上 3 种直接对数组或数组元素赋值，都会自动创建对应的数组变量。

d) 如果想声明一个数组变量，但不初始化，可以使用

```bash
# 不支持直接初始化
declare -a arr
```

## 读取数组

读取数组中的某一个元素，常用方式如下：

```bash
${array_name[index]}
```

例如：

```bash
value=${arrs[2]}
```

如果要将数组中的元素全部输出，我们这时候可以使用特殊符号`*`和`@`

```bash
echo $arrs[*]
echo $arrs[@]
```

## 获取数组长度

获取数组长度：

```text
len=${#arrs}
len=${#arrs[*]}
len=${#arrs[@]}
```

我们注意到，在 Shell 中获取数组长度上，需要使用`#`符号放在数组名称之前。

获取具体元素的长度：

```bash
len=${#arrs[n]}
```

这里的`n`表示具体下标的数字，如果你要获取数组中下标为 2 的数组长度，那么 Shell 语句就如下所示：

```bash
len=${#arrs[2]}
```

看个问题：

```bash
arr=(0 1 [4]=4)
echo ${#arr[@]}  # 3
```

这说明，`arr` 的结构是 `([0]=0 [1]=1 [4]=4)`，而不是 `(0 1 null null 4)`，所以，数组的内部结构更像是 map。Bash 并没有要求下标是连续的。

## 数组遍历

`${arr[@]}` 和 `${arr[*]}` 展开为数组的全部元素。二者的区别与 `$@` 和 `$*` 类似，在执行单词分割的上下文中，前者可被分割为单个元素，后者作为整体不可分割。

```bash
pets=("a cat" "a dog")

# 这是理想的遍历效果
for pet in "${pets[@]}"; do echo $pet; done
# a cat
# a dog

# "${arr[*]}" 把所有元素拼在一起
for pet in "${pets[*]}"; do echo $pet; done
# a cat a dog

# 不加引号，会以空格分割
for pet in ${pets[@]}; do echo $pet; done
# a
# cat
# a
# do
```

> 💡 用 `arr[@]` 的地方都可以使用 `arr[*]`，它们的区别与上例类似，下面忽略 `arr[*]` 写法。

如果要通过下标遍历，需要使用 `"${!name[@]}"` 。

```bash
for i in "${!arr[@]}"; do echo "arr[${i}]=${arr[i]}"; done
# arr[0]=0
# arr[1]=1
# arr[4]=4
```

因为数组的下标并不一定是连续递增，通过 `0...len` 的下标遍历长度为 len 的数组并不靠谱。

## 拼接数组

不同的数组如果要组合成一个数组，首先使用`@`和`*`来获取数组的全部元素，然后通过 Shell 数组定义的方式将两个数组拼接到一起，示例如下：

```bash
#!/bin/bash

arr1=(1 2 3 4 5 qq)
arr2=(6 7 8 9 10 wechat)

arr3=(${arr1[@]} ${arr2[@]})
# 这是理想的遍历效果
for arr in "${arr3[@]}"; do echo $arr; done
```

输出结果：

```text
1
2
3
4
5
qq
6
7
8
9
10
wechat
```

## 数组切片

数组切片与字符串子串语法类似。

```bash
${arr[@]:offset:len}
```

表示从 `arr[offset]` 开始，长度为 `len` 的数组。

其中，`offset` 可以为负数。`len` 非负，省略时表示直到数组末尾。

```bash
arr=(0 1 2 3 4 a b c d)
echo ${arr[4]}
# 4 a b c d

echo ${arr[@]:4:2}
# 4 a

# offset 为负时，注意前面需要加空格
echo ${arr[@]: -4: 2}
# a b
```

## 数组元素的增删

往数组最后增加其它元素（push），可以使用

```bash
arr+=(value1 value2 ...)
```

灵活使用切片和赋值，可以实现多种数组操作。

```bash
# push
arr=("${arr[@]}" value1 value2 )

# shift
arr=(value1 value2 "${arr[@]}")

# insert（在下标从0开始且连续的前提下）
arr=("${arr[@]:0:2}" value1 value2 "${arr[@]:2}")

# remove（在下标从0开始且连续的前提下）
arr=("${arr[@]:0:2}" "${arr[@]:3}")
```

## 删除数组中的元素

在 Shell 中删除元素可以通过 unset 进行删除，具体格式如下：

```bash
unset arrs[n]
```

其中的`n`就是数组元素的下标，可以根据要删除的数组元素填写下标。

如果我们要删除整个数组也可以通过 unset 来进行删除，具体方式如下：

```bash
unset arrs
```

以上的语句执行后，arrs 整个数组的元素将会被删除。

## 特殊的数组：关联数组

Bash 还支持一种更通用的数组语法 —— 关联数组（associative array），可以在数组中使用字符串作为下标。普通数组是数字到字符串的映射，关联数组建立了字符串到字符串的映射，是更通用的 map 结构。

```bash
ass_arr=(["white"]="#fff" ["green"]="#0f0") # 或者使用 declare -A 声明
echo ${ass_arr["white"]} # #fff
```
