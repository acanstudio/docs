Git远程仓库（续）
------------------

.gitignore
------------

前一篇中，我们介绍了Git的patch功能，当我们生成patch之后，"git status"就会显示patch文件是"Untracked files"。当然，我们也没有必要去跟踪这个patch文件。

同样，项目中可能会经常生成一些Git系统不需要追踪(track)的文件，在编译生成过程中产生的文件或是编程器生成的临时备份文件。我们可以在使用"git add"是避免添加这些文件到暂存区中，但是每次都这么做会比较麻烦。

所以，为了满足上面的需求，Git系统中有一个忽略特定文件的功能。我们可以在工作目录中添加一个叫".gitignore"的文件，来告诉Git系统要忽略哪些文件。当我们使用添加过".gitignore"文件后，文件中的过滤规则就生效了。

注意：
    * 在windows环境中不支持文件名为".gitignore"，所以可以把文件命名为".gitignore."
    * ".gitignore"文件只会对当前目录以及所有当前目录的子目录生效；也就是说如果我们把".gitignore"文件移到"advance"文件夹中，那么过滤规则就是会对"advance"及其子目录生效了
    * 建议把".gitignore"文件提交到仓库里，这样其他的开发人员也可以共享这套过滤规则

常用的过滤语法：
    * 斜杠"/"开头表示目录
    * 星号"*"通配多个字符
    * 问号"?"通配单个字符
    * 方括号"[]"包含单个字符的匹配列表
    * 叹号"!"表示不忽略匹配到的文件或目录

下面举一些简单的例子：
    * foo/*：忽略目录 foo下的全部内容
    * *.[oa]：忽略所有.o和.a文件
    * !calc.o：不能忽略calc.o文件

exclude文件
------------

在Git仓库中有一个".git/info/exclude"文件，当我们指向对特定的仓库使用特定的过滤规则时，我们可以把过滤语句写在exclude文件中。
 
远程仓库命令
-------------

前面一篇文章简单的介绍了push、pull命令的使用，这里将进一步展开介绍。

#### git branch和git remote

<pre>
# 列出本地分支
$ git branch
# 列出远程分支
$ git branch -r
# 列出全部分支
$ git brach -a

# 列出所有远程主机
$ git remote
# 列出所有远程主机，并显示远程主机地址
$ git remote -v
</pre>

#### git push

push命令用来将本地分支的更新推送的远程仓库，该命令的格式如下： git push <远程主机名> <本地分支名>:<远程分支名>

<pre>
# 通过"git push"更新、创建远程分支
# 通过<本地分支名>:<远程分支名>方式推送更新
$ git push origin release-1.0:release-1.0
# 当本地分支已经与远程分支关联的，可以省略远程分支名
$ git push origin release-1.0
# 如果远程分支不存在，那么push将创建一个远程分支，并与本地分支关联?
$ git push origin dev

# 通过"git push"删除远程分支
$ git push origin :dev
$ git push origin --delete dev

# 省略分支信息的"git push origin"
# 通过这种方式push的时候，报出了一个警告，提示"push.default"没有设置。
# 在Git中push有两种设置：
# simple方式：只是推送当前分支的更新到对应的远程分支；在Git 2.0以后就默认使用这种方式
# matching方式：会推送所有有对应的远程分支的本地分支
# 根据Git的提示，我们可以通过"git config --global push.default"来设置push方式。
$ git config --global push.default simple
$ git push origin
</pre>

#### git pull

pull命令的作用是取回远程某个分支的更新，然后合并到指定的本地分支，pull命令格式如下：git pull <远程主机名> <远程分支名>:<本地分支名>

<pre>
$ git pull origin release-1.0:release1.0
# 取回origin主机release-1.0分支的更新，与本地的release-1.0分支合并。
# 一般来说，pull命令都是在关联的本地分支和远程分支之间进行；
# 也可以使用不关联的本地分支和远程分支进行pull操作，但是不建议这么做。
# 如果真的需要别的远程分支上的更新，建议使用"cherry-pick"把这个更新拿到关联的远程分支上，
# 然后在关联分支上进行pull操作。

# 省略本地分支名：
$ git pull origin release-1.0
# 表示取回origin/release-1.0远程分支的更新，然后合并到当前分支

# 如果当前分支存在上游（关联）分支，可以直接使用
$ git pull origin
# 表示本地的当前分支自动与关联的origin主机分支进行合并

# git pull操作实际上等价于，先执行git fetch获得远程更新，
# 然后git merge与本地分支进行合并。

# pull命令也支持使用rebase模式进行合并：
$ git pull --rebase <远程主机名> <远程分支名>:<本地分支名>
# 在这种情况下，"git pull"就等价于"git fetch"加上"git rebase"。
# 建议使用"git fetch"加上"git rebase"的方式来取代"git pull"获取远程更新，具体原因后面再介绍。
</pre>

#### git fetch

fetch命令比较简单，作用就是将远程的更新取回到本地。

<pre>
$ git fetch origin
# 该命令表示将远程origin主机的所有分支上的更新取回本地
$ git fetch origin master
# 该命令只取回远程origin主机上master分支上的更新
</pre>
