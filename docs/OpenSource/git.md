### Git

> 版本&配置

```bash
# 版本
git —version

# 配置
git config —global user.name 'Your Name'
git config —global user.email 'Your Email'

# 查看配置
Git config -l
```

> 初始化

```bash
git init folder
```

> 添加文件&commit
 
```bash
git add . && git commit -m '1'
```

> 查看日志
                             
```bash
git log —oneline
```

> remote 远程仓

```bash
git remote
git remote -v

# 推送到远程
git push -u GitHub mster

# 克隆远程仓
git clone https://... <your floder>

# 更改远程 remote
git remote set-url origin URL
```

> e.g.

```bash
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/CaoChensy/MyLearning.git
git push -u origin master
```

> log 日志

```bash
# 详细查看版本日志，查看提交历史
git log
# 简单罗列版本日志
git log --pretty=oneline
# 查看命令历史
git reflog
```

> remote 远程仓

```bash
# 查看远程库信息 
git remote -v；
# 从本地推送分支 
git push origin branch-name，如果推送失败，先用git pull抓取远程的新提交；
# 在本地创建和远程分支对应的分支 
git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；
# 建立本地分支和远程分支的关联 
git branch --set-upstream branch-name origin/branch-name；
# 从远程抓取分支 
git pull，如果有冲突，要先处理冲突。
# 工作区的状态 
git status命
# 查看修改内容 
git diff
# 版本回滚 
git reset --hard commit_id
# 添加远程 push
git remote set-url --add origin https://gitee.com/chensy_cao/Learning.git
# 删除远程
git remote rm gitee
# 添加远程
gie remote add <gitee> <url>
```

> branch 分支

```bash
# 查看分支
git branch
# 创建分支
git branch <name>
# 切换分支
git checkout <name>
# 创建+切换分支
git checkout -b <name>
# 合并某分支到当前分支
git merge <name>
# 删除分支
git branch -d <name>
# 列出所有分支(远程和本地)
git branch -a
# 新建并切换到分支(克隆当前分支)
git checkout -b new_branch
# 新建一个远程分支
git push origin new_branch
```

> tag 标签

```bash
# 新建一个标签 
git tag <name>
# 指定标签信息 
git tag -a <tagname> -m "blablabla..."
# 用PGP签名标签 
git tag -s <tagname> -m "blablabla..."
# 查看所有标签 
git tag
# 查看本地有所的tag
git tag -l
# 新建一个附注标签
git tag -a new_tag -m "your tag message"
# 提交标签到远程
git push origin your_tag_name
# 提交所有的tag到远程
git push origin --tags
# 删除标签
git -d your_tag_name
# 查看标签信息
git show your_tag_name
# 推送一个本地标签 
git push origin <tagname>
# 推送全部未推送过的本地标签 
git push origin --tags
# 删除一个本地标签 
git tag -d <tagname>
# 删除一个远程标签 
git push origin :refs/tags/<tagname>
```

> `.gitignore` 忽略 `Pycharm` `.idea` 文件夹

- 如果`.gitignore`文件是后续添加的
- 需要先清除`.idea`的`git`缓存

```git
git rm -r --cached .idea
```