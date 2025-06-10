# ls-files

使用`git ls-files`指令可以查看指定状态的文件列表，格式如下：

```
# 查看指定状态的文件
git ls-files [<options>] [<file>...]

git ls-files 				# 默认查看所有缓存的文件
git ls-files -o				# 查看未被跟踪的文件
git ls-files --modified 	# 查看被修改的文件
git ls-files -s 			# 查看暂存区中文件详细
```
