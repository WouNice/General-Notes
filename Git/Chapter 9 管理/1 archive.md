# git archive

`git archive` 用于生成一个可供发布的压缩包。

```bash
# 查看支持的归档格式
git archive --list

# 导出最新的版本库
git archive -o ../latest.zip HEAD

# 导出指定提交记录
git archive -o ../git-xxx.tar 8996b47

# 导出一个目录
git archive -o ../git-xxx-docs.zip HEAD:Documentation/

# 导出为 tar.gz 格式
git archive 8996b47 | gzip > ../git-xxx.tar.gz
```
