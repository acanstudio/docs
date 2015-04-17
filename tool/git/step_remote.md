Git远程仓库
============

前面文章中出现的所有Git操作都是基于本地仓库的，但是日常工作中需要多人合作，不可能一直都在自己的代码仓库工作。所以，这里我们就开始介绍Git远程仓库。

在Git系统中，用户可以通过push/pull命令来推送/获取别的开发人员的更新，对于一个工作组来说，这种方式会效率比较低。所以，在一个Git系统中，都会有一个中心服务器，大家都通过中心服务器来推送/获取更新。

为了方便本篇例子的进行，我就使用多个目录来模拟多个用户以及中心服务器，这样就不用搭建Git服务器了。
    * 中心服务器：C:\VM\CentralRepo
    * 用户wilber：C:\VM\wilberWorkSpace
    * 用户will：C:\VM\willWorkSpace

 
Git远程仓库基本操作
---------------------

<pre>
# 创建一个空的中心服务器就,带--bare选项创建的是一个裸仓库，
# 这个仓库只保存Git历史提交的版本信息，而不允许用户在上面进行其他的git操作
$ cd /path/to/repo
$ git init --bare
</pre>

用户user1从中心服务器获取版本库，更新并提交更新至中心服务器

<pre>
# git clone命令从中心服务器克隆一个版本库至本地
# 在clone命令中需要给出目标仓库的地址，
# Git支持http、ssh以及git原生协议来进行clone，也可以使用本地目录。
$ cd /path/to/local
$ git clone /path/to/repo ./user1
$ cd user1
# do something
$ git add .
$ git commit -m 'user1 commit first'
$ git push origin master
</pre>

用户user1从中心服务器获取版本库，更新并提交更新至中心服务器

<pre>
# 从中心服务器克隆一个版本库至本地
$ cd /path/to/local
$ git clone /path/to/repo ./user2
$ cd user2
# 此时user2的工作目录已经包含了user1提交的一个更新

# do something
$ git add .
$ git commit -m 'user2 commit first'
$ git push origin master

# 切换至用户user1工作目录
$ cd /path/to/local/user1
$ git pull 
# 用户user1的工作目录包含了user2提交的更新
</pre>

上游仓库(upstream repository)
------------------------------

在Git系统中，通常用"origin" 来表示上游仓库。我们可以通过 "git branch -r"命令查看上游仓库里所有的分支，再用 origin/name-of-upstream-branch 的名字来抓取(fetch)远程追踪分支里的内容。

<pre>
# 在中心服务器上创建两个分支
$ cd /path/to/repo
$ git branch release-1.0
$ git branch release-1.1

$ cd /path/to/local/user1
$ git pull
$ git branch -r
</pre>
从上面可以看出，对于wilber的仓库，master分支的上游分支是中心服务器的master分支。

通过上一篇文章的内容，我们可以在中心服务器上建立两个新的"release-1.0"和"release-1.1"分支，通过pull命令，用户wilber就看到了这些上游分支。

--set-upstream-to

当我们相当然的按照前一篇文章在本地仓库建立一个branch的时候，我们的pull操作会遇到以下问题，提示我们这个branch没有任何的跟踪信息。仔细想想也是，我们应该把本地仓库中的分支与上游分支关联起来。这时，我们就可以根据Git的提示，使用"--set-upstream-to"命令进行关联了。

<pre>
$ cd /path/to/local/user1
$ git branch release-1.0
$ git checkout -b release-1.1
$ git pull 
# 提示没有任何的跟踪信息
$ git branch --set-upstream-to=origin/remoteBranchName release-1.1
$ git checkout -b localBranchName origin/remoteBranchName
# 建议把"localBranchName"名称跟"remoteBranchName"名称设置成一样。
</pre>

patch
-------

在Git中patch是一个非常实用的工具。假设在一个没有网络的环境中，user1和user2还要继续工作，这时user1有一个更新，user2需要基于这个更新进行下一步的工作。如果是集中式的代码版本工具，这种情况就没有办法工作了，但是在Git中，我们就可以通过patch的方式，把user1的更新拷贝给user2。

在Git中有两种patch的方式

#### 通过"git diff"生成一个标准的patch

<pre>
$ mkdir /path/to/local/patch
$ cd /path/to/local/user1
# user1生成新的更新，同时针对这些更新生成一个patch
$ git diff > /path/to/local/patch/user1Patch1

$ cd /path/to/local/user2
# 用户user2通过git apply将user1的patch提取到自己的工作目录
$ git apply /path/to/local/patch/user1Patch1
</pre>

#### 通过"git format-patch"生成一个Git专用的patch。

<pre>
$ cd /path/to/local/user1
# user1生成新的更新
$ git add .
$ git commit -m 'commit for patch test'
# 注意参数"origin/master"，这个参数的含义是，找出当前master跟origin/master之间的差别，然后生成一个patch
$ git format-patch origin/master -o /path/to/local/patch/

$ cd /path/to/local/user2
$ git am /path/to/local/patch/*.patch
</pre>

#### 两种patch方式的比较

下面简单看看两种patch方式的比较。
    * patch兼容性："git diff"生成的patch兼容性强。也就是说，如果别人的代码版本库不是Git，那么必须使用git diff生成的patch才能被别的版本控制系统接受。
    * patch合并提示：对于"git diff"生成的patch，你可以用git apply --check 查看patch是否能够干净顺利地应用到当前分支中；如果"git format-patch" 生成的patch不能打到当前分支，git am会给出提示，帮你解决冲突，两者都有较好的提示功能。
    * patch信息管理：通过"git format-patch"生成的patch中有很多信息，比如开发者的名字，因此在应用patch时，这个名字会被记录进版本库，这样做是比较合理的。

