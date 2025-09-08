- 初始化仓库

```markdown
git init
```

<br/>

- 查看本地分支

```text
git branch
```

<br/>

- 切换分支

```markdown
# 传统方式
git checkout 分支名

# 新版
git switch 分支名
```

<br/>

- 创建新分支并切换

```text
# 传统方式
git checkout -b 新分支名

# 新版
git witch -c 新分支名
```

<br/>

- 本地提交分支

```text
# 本地修改提交
git commit -m "提交说明"

# 暂存修改，可恢复
git stash
# 恢复
git stash pop
```

<br/>

- 丢弃修改

```text
# 强制切换
git checkout -f 分支名
```

<br/>

- 拉取所有远程分支信息

```text
git fetch --all
git branch -a
```