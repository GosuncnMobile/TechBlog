---
title: Git常用命令简介
date: 2017-12-18 09:39:09
categories: Git
tags: Git
---
*欢迎对文章内容进行补充和修正~*
### 声明
* 本文章转自网络[github常用命令汇总](https://www.cnblogs.com/TaylorApril/p/6815142.html)
* 如有侵权，请作者及时联系删除。



### 创立版本库
```
 mkdir Baidu
 cd Baidu
 git init
```

### SSH
```
ssh-keygen -t -rsa -C "TaylorApril947939@gmail"
（在github上new SSH，内容为id_rsa.pub）
```
### 添加远程库
```
（github上新建git仓库,选择REAMDE.md）
git remote add origin git@github.com:TaylorApril/Baidu.git
git push -u origin master
（报错）
git pull --rebase origin master
git push -u origin master
git push origin master
```
### 提交(文件名字read.txt)
```
git add read.txt
（所有文件 git add .）
git commit -m "说明文字"
git push
```
<!--more-->
### 版本回退
```
（查看最近到最远提交日志）
git log --prtty=oneline
（回退版本计数：HEAD^上一个版本、HEAD~100上与100个版本）
git reset --hard HEAD^
（回退版本号码：回退版本commit id为3628df）
git reset --hard 3628df
（想要恢复:使用命令记录来找回commit id回退）
git reflog
```

### 撤销修改
```
———>              git add files              git commit 
working directory--------------stage-------------history
<———         git checkout --files         git reset --files
```

### 删除文件（read.txt）
```
rm read.txt
（从版本库中删除）
git rm read.txt
git commit -m "delete"
（删错了）
git checkout -- read.txt
```
### 分支(分支名字dev)
```
创建分支：git branch dev
切换分支：git checkout dev
创建+切换分支：git checkout -b dev
查看当前分支：git branch
切换回master分支：git checkout master
合并指定分支到当前分支：git merge dev
(fast-forward 快进模式)
删除分支：git branch -d dev

```
### 解决冲突(分支名字fea)
* 冲突原因：master和Dev同时增长。
```
git checkout -b fea
（修改Creating a new branch is quick AND simple.）
git add read.txt
git commit -m "fea"
git checkout master
（修改Creating a new branch is quick & simple.）
git addread.txt
git commit -m "master"
（此时形成了master和fea各自指着一个分支）
git merge fea
（合并错误，git status , cat read.txt可以查看）
（修改read.txt的文本内容 Creating a new branch is quick and simple.）
git add read.txt
git commit -m “conf”
（现在master和fea指向同一个人点了，git log可查看合并情况）
git branch -d fea
```
### 分支管理策略（--no-ff）（分支名字dev 文件名字read.txt）
```
git checkout -b dev
git add read.txt
git commit -m "dev"
git checkout master
（注意下个参数--no-ff,表示禁用fast forward）
（fast forward合并看不出曾经做过合并，而--no-ff参数合并后的历史有分支，negative看出曾经做过合并）
git merge --no-ff -m "merge with --no-ff" dev
（查看分支历史 git log --graph --pretty=oneline --abbrev-commit）
```
### bug分支(bugg分支为要解决bug的分支)
* 思想：当手头还有工作时，先将工作现场git stash(避免bug修复好后将为完成的工作一起提交),然后修复bug、提交之后，在用git stash pop将原来的工作显示在工作区 。
```
（git status查看状态）
git stash
git checkout -b bugg
（修改bug后）
git add bugg.txt
git commit -m "fixed bug"
git checkout master
git merge --no-ff -m "merge bug" bugg
git branch -d bugg
（接下来回到dev上继续工作）
git checkout dev
（查看工作区git status）
（用git stash list查看）
git stash list
（恢复的第一种方法：恢复的同时把stash内容同时删除）
git stash pop
（恢复的第二种方法：恢复的同时不删除stash内容）
git stash apply
（若使用第二种方法想删除stash则用git stash drop）
（若是多个文件stash 可用git stash apply stash@{0}恢复指定的stash）
```
### feature分支(分支为dev)
```
（在没完全完成合并时强行删除）
git branch -D dev
```
### 多人协作
```
（查看远程库信息）
git remote
（查看远程库更详细信息）
git remote -v
```
### 推送dev分支
```
git push origin dev
```
### 抓取分支
```
（克隆）
git clone git@github.com:TaylorApril/test.git
（查看能见分支 git branch）
（在dev分支上开发，创建远程origin的dev分支到本地）
git checkout -b dev origin/dev
（修改后，进行提交）
git commit -m "add"
git push origin dev
（在他提交之后你再push的情况时）
（指定本地dev分支与远程origin分支链接）
git branch
git pull
git commit -m "fixed"
git push
```
### 报错
#### 报错1

```
$ git push -u origin master
To git@github.com:TaylorApril/test.git
! [rejected] master -> master (fetch first)
error: failed to push some refs to 'git@github.com:TaylorApril/test.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

* 解决：
```
git pull --rebase origin master
git push -u origin master
```

#### 报错2
```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
(use "git push" to publish your local commits)
```
* 解决：
```
git push
```
