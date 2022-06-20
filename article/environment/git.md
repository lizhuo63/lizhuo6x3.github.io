# 基本概念

+ 集中式：只能向远端仓库提交，不够方便（SVN），且分支昂贵
+ 分布式：支持向本地仓库提交 （Git），且分支廉价
+ 快照 ：即被追踪的最顶层的文件树，Git只记录存储在本地库中的快照(parent快照)。
+ 工作区：实际写代码的地方，没保存到数据库中
+ 暂存区：临时存放变更的代码，下次提交会带上它。
+ 本地库：存放本地历史版本，装入了parent快照信息

[Pro Git书籍](https://git-scm.com/book/zh/v2)

[Git练手小游戏](https://learngitbranching.js.org)

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-16_23-05-57.png" style="zoom:67%;" />

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-16_23-09-56.png" style="zoom:67%;" />

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-16_23-11-00.png" style="zoom:67%;" />

#  配置

## windows

配置文件C:\Users\operator\.gitconfig

```
[alias]
	gs=status
	gc=commit -m
	gca=git commit --amend -m
	gl=log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit  
	gb=branch
	ga=add .
	go=checkout
	gpm=push -u origin master
	gprm=pull -rebase origin master
	gp=push -u origin
	gpr=pull -rebase origin
	gra=remote add origin
```

## linux

修改~/.bash_profile；或者~/.zshrc (zsh)

```
alias gs="git status"
alias gc="git commit -m "
alias gca="git commit --amend -m "
alias gl="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit  "
alias gb="git branch"
alias ga="git add ."
alias go="git checkout"
alias gpm="git push -u origin master"
alias gprm="git pull -rebase origin master"
alias gp="git push -u origin"
alias gpr="git pull -rebase origin"
alias gra="git remote add origin"
```

# 操作

## 一波流

```
ga 添加本地
gc + 提示
gra + 远程地址
gpr + 分支名 拉取最新分支
gp + 分支名 推送
```

## 文件

+ **状态查看**

    ```
    gs -s #简版状态  ??:未添加 M:已修改,未暂存 A:以暂存,未提交
    git diff #未暂存文件的差异更新
    git diff --staged #已暂存文件对比最近提交的更新
    ```

+  **提交**

    ```
    git commit --amend -m ('提交描述2') #将此次提交融到上次提交中【在提交1之上进行更该，不会增加新的提交记录】
    ```

+ **删除&撤销**

    ```
    git checkout -- a.txt    工作区的修改全部撤销
      #1已暂存   回到上次提交版本库后的状态
      #2未暂存	  回到上次添加到暂存区后的状态
      
    git rm a.txt # 从仓库中删除，默认会连带删除本地和暂存区文件【连渣都不剩】
    git rm --cached b.txt # 从仓库、暂存区中删除，但不会动本地文件【再次添加后会自动提交】
    git mv b.txt newBee.txt # 对已提交文件进行改名
    
    git clean -f ../a.txt # 撤销新建文件
    git clean -df ./a #  撤销新建文件夹
    git reset HEAD f.txt # 删除f.txt的暂存
    
    # 撤销提交
    #第一步 撤销本地提交
     + git reset --hard [版本编号]   重置HEAD和branch，并重置stage区和工作区里的内容。
    + git reset --soft [版本编号]   重置HEAD和branch，但保留工作区和暂存区中的内容，并把重置HEAD所带来的新的差异放进暂存区。
    + git reset (--mixed) [版本编号]   保留工作区，并且清空暂存区
    #第二步 推送到服务器
    git push origin HEAD:master --force
    
    #回退到上一次的提交版本
    git reset HEAD f.txt #1. 删除f.txt在暂存区的存根，与上一次 commit 保持一致
    git checkout -- f.txt #2.回退到上次的提交版本【必须要求暂存区无此存根，否则回退失效】
    git revert [commit] # 新建一个 commit，用于撤销指定 commit
    ```

## 分支

+ **基本操作**

    ```
    创建并切换到此分支：git checkout -b 分支名
    
    将ask合并到master：1.git checkout master \ 2.git merge ask
    
    查看合并的分支：git branch --merged
    查看没有合并的分支：git branch --no-merged
    
    删除分支：git branch -d 分支名
    强制删除分支：git branch -D 分支名(未提交的分支-d无法删除)
    ```

+ **分支冲突**

    手动对冲突代码进行取舍保留，然后添加提交即可。

+ 分支合并前，所在主干出现了新提交(**目的：便于冲突处理，简化日志记录**)：

    ```
    #1 切到要合并的分支
    git checkout ask；
    #2 重新定位到所在主干的最近一次提交节点上
    git rebase master;
    #3 合并
    git merge master;
    ```

## 远程

```
git remote add origin 远程仓库地址；# 关联远程仓库
git push --set-upstream origin ask ；# 将本地的ask分支与远程进行关联
git remote -v；# 查看远程库
git branch -a；# 查看当前库的所有分支(包括远程)
git remote show [remote] # 显示某个远程仓库的信息

git pull；#更新当前分支
git fetch [remote] # 下载远程仓库的所有变动
git pull --rebase origin ask #将远程ask分支拉取到本地的ask分支，并变基合并

git push -u origin master；# 推送数据到远程仓库，远程分支合并

git push origin --delete ask； # 删除远程分支
```

##  标签

```
git tag #显示标签列表
git tag v1.0 -m "信息" #打标签
git push origin v1.0 #推送标签到远程
git tag -d v1.0 #删除本地标签
git push origin :refs/tags/v1.0 #删除远程标签
git checkout -b version2 v1.0 #更改旧标签，需要新建
```

## 发布

- 压缩包： git archive master --prefix='A/' --forma=zip > B.zip【将master分支放到A目录下，并将其压缩为B.zip】

##  日志

```
git log # 显示当前仓库所有的提交记录
附带参数
 -p #显示所有的变动信息
 -n #显示最近n次提交
 --oneline #只显示提交信息(简短描述) 
 --name-only # 显示有哪些文件发生了变动
 --name-status #显示变动文件及变动类型
```

**Q：退出log**

##  栈

Git的栈中保存当前未提交的工作进度

```
git stash save ‘message’ #将未提交的所有代码存到栈中
git stash save -a “messeag” #将未提交的所有代码存到栈中，新加入的代码同时放入暂存区
git stash list #查看
git stash drop #将最近stash记录删除。
git stash apply #从栈中读取最近内容并恢复到工作区，但该stash记录不删除
git stash pop #从栈中读取最近内容恢复到工作区。同时删除该stash记录
git stash clear 清空Git栈，
```

