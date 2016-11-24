---
title: git命令行入门
date: 2016-08-31 19:43:02
tags: Git
categories: Git
---

本文结合项目开发中遇到的实际问题对git命令行的使用做一个简单的总结。

### 常用命令

1. 下载项目

       git clone [url]

2. 查看本地分支

       git branch

3. 查看远程分支

       git branch -r

4. 创建本地分支

       git checkout [branch]

5. 切换本地分支

       git checkout [branch]

6. 拉取远程分支代码

       git pull origin [remote-branch]

7. 添加当前目录的所有文件到暂存区

       git add .

8. 提交暂存区到仓库区

       git commit -m [message]

9. 合并分支

       git merge [branch]

10. 提交本地分支代码

      git push origin [remote-branch]

### tag命令

1. 列出所有tag

       git tag

2. 新建tag

       # 在当前commit新建一个tag
       git tag [tag]
       
       # 在指定commit新建一个tag
       git tag [tag] [commit]

3. 删除本地tag

       git tag -d [tag]

4. 删除远程tag

       git push origin :refs/tags/[tag]

5. 提交指定tag

       git push [remote] [tag]

6. 提交所有tag

       git push --tags

7. 基于某个tag新建一个分支

       git checkout -b [branch] [tag]

   ​    
### 撤销命令

1. 恢复暂存区的指定文件到工作区

       git checkout [file]

2. 恢复某个commit的指定文件到工作区

       git checkout [commit] [file]

3. 恢复暂存区的所有文件到工作区

       git checkout .

4. 重置暂存区的指定文件，与上一次commit保持一致，但是工作区不变

       git reset [file]

5. 重置暂存区和工作区，与上次commit保持一致

       git reset --hard       
   ​    
### 删除命令

1. 删除工作区的文件，并将这次删除放入暂存区

       git rm [file1] [file2]  

2. 停止追踪指定文件，但该文件会保留在工作区

       git rm --cached [file]

3. 删除本地分支

       git branch -d [branch]

4. 删除远程分支

       git push origin --delete [remote-branch]

       git branch -dr [remore-branch]

5. 删除本地tag

       git tag -d [tag]

6. 删除远程tag

       git push origin :refs/tags/[tag]


#### 冲突解决

1. 查看冲突

   ```
   git diff
   ```

2. 选择对方版本

   ```
   git checkout --theirs [conflict file]
   ```

3. 选择本地版本

   ```
   git checkout --ours [conflict flie]
   ```

### 参考链接

[常用Git命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)