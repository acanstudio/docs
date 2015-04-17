git入门之分支（branch）
========================

在代码版本控制工具中，都会有branch的概念。刚开始建立版本仓库的时候，我们只有一个主分支（master branch），我们不可能把日常的新功能开发、代码优化以及bug修复等概念工作全都放在主分支上，这样会使主分支很难维护。这就是为什么会有branch。
在使用Git进行代码控制中，分支是很容易创建的，所以也建议使用分支来进行开发新功能、bug修复。
 
分支的创建
------------

<pre>
# 基于当前HEAD创建分支release-1.0
git branch release-1.0
# 查看当前本地分支
git branch

# 切换至release-1.0分支
git checkout release-1.0
# 创建并切换至release-1.1
git checkout release-1.1 -b
</pre>

注意：大家一定还记得第二篇文章中我们通过"checkout"命令来还原WorkSpace中的更新，在还原的命令中我们使用的是"checkout --"，如果没有"--"就代表切换branch。

根据前面两篇文章的知识，我们进入".git/refs/heads"目录，发现有"HEAD"和"release-1.0"两个文件，并且两个文件包含的哈希值相同，根据"git log"可以知道这个哈希值代表master上最新的提交。所以，创建分支后我们会得到下面的关系图，从值张图中可以看到，branch的切换对应HEAD引用值的改变。

![step_branch_1](#UPLOAD_URL#/images/step_branch_1.png)

有了新的branch之后，我们就可以分别在不同的branch上工作了。假设我们现在更新"app.py"，并且在release-1.0 branch上面提交，重新查看对象关系图。

![step_branch_2](#UPLOAD_URL#/images/step_branch_2.png)

在新的release-1.0分支上提交更新后，"ref/heads/release-1.0"文件内的哈希值将更新为release-1.0 branch上最新的commit-id，release-1.0 branch上面的更新不会体现在master branch。

分支的删除
-----------

<pre>
# 不能删除当前分支，需要切换至其他分区后再删除指定分区
$ git branch -d release-1.1
</pre>
 
分支的合并
-----------

branch的创建是为了方便开发、修复bug以及保持master的稳定。但是最终branch上的内容还是要合并到master的，接下来就看看分支的合并。

<pre>
$ git checkout master
$ git merge release-1.0
# 这样release-1.0上的更新就合并到master branch上了
</pre>

合并之后，master的HEAD就被更新了，跟release-1.0内容一致了，这些就是merge命令做的事情。如下图

![step_branch_3](#UPLOAD_URL#/images/step_branch_3.png)
 
合并冲突
---------

在branch的合并中，很多时候我们会遇到冲突，那么我们就需要手动解决冲突，然后再提交了。

<pre>
$ git checkout master
# master回滚到与release-1.0合并之前的版本
$ git reset --hard HEAD~1
# 在master产生与release-1.0相冲突的更新
$ git add .
$ git commit -m 'master conflict update to release-1.0'
$ git merge release-1.0
# 合并失败，提示有冲突
$ git status
# 找到冲突的文件，文件中会用"<<<<<<<"，"======="，">>>>>>>"标记出冲突区域，
# 手动编辑并修复文件中的冲突
$ git add .
$ git commit -m 'fix conflict'
</pre>

冲突解决之后，可以通过"git log"来产看一些结果，但是这次我们要给"git log"加一些参数。

<pre>
# git log的选项 
# --graph:显示commit的历史,包括不同分支的提交
# --pretty-online: 只显示一行提交信息
# --pretty=raw: 显示提交的详细信息
# --abbrev-commit: 显示commit-id的精简值
$ git log --graph --pretty-oneline --abbrev-commit
</pre>

同时，这里给出最新的对象关系图

![step_branch_4](#UPLOAD_URL#/images/step_branch_4.png)
 
stash
-------

在Git的分支管理中，stash是个很有用的命令，可以保存我们做到一半的工作，可以理解成一个未完成工作的保存区。

假如我们在release-1.0 branch做了一些更新，但是想做的事情还没有全部完成，不能提交，这是我们又要切换到master branch，这是Git会禁止branch切换。

<pre>
$ git checkout release-1.0
# release-1.0的Workspace有了新的更新，在提交新的更新之前要切换到master分支
$ git checkout master
# git会提示release-1.0有更新不能切换至master

# 将当前未提交的更新压入工作缓存区
$ git stash 
# 查单当前工作缓存区列表
$ git stash list
# 顺利切换至master分区
$ git checkout master

$ git checkout release-1.0
# 从工作缓存区中获取release-1.0之前未提交的更新
$ git stash apply
</pre>

stash空间就像是一个栈空间，每次通过stash保存等内容都会被压入stash栈。命令不仅仅是支持简单的list、apply操作，接下来我们看看更多的stash命令。

<pre>
# 可以通过自定义的信息来描述一个stash
$ git stash save 'description for stash'

# 可以执行多次stash，生成多个缓存区
$ git stash list
stash@{0}: On release-1.0: description
stash@{1}: On release-1.0: other description
# 获取指定的缓存区更新
$ git stash apply stash@{0}
# 用最近的stash返回更新，并删除该缓存区
$ git stash pop
</pre>

### stash工作原理

相信大家都看到了stash的强大，下面我们来看看stash的工作原理。在使用过stash保存之后，我们会发现.git目录中出现了两个新文件".git/refs/stash"和".git/logs/refs/stash"。两个文件内容分别如下：

<pre>
.git/refs/stash
dcac98e565864edc6f636b08660baebe2c97e7d2

.git/logs/refs/stash
0000000000000000000000000000000000000000 dcac98e565864edc6f636b08660baebe2c97e7d2 WilberTian <Wilber***com> 1420201804 +0800    WIP on release-1.0: ed17809 update app.py on release branch
</pre>

根据Git的对象模型原理，可以看到，stash存放内容也是可以根据对象关系模型一点点找出来。

branch之间的diff
-----------------

在前面的文章中我们通过diff比较过同一个分支上的内容在WrokSpace、stage和repo中的差别。

同样diff可以支持分支之间的比较。

<pre>
# 把当前branch跟branchName进行比较
$ git diff branchName
# 比较branchName1和branchName2两个分区
$ git diff branchName1 branchName1

# 比较两个branch的fileName文件差异
$ git diff branchName -- fileName
</pre>

