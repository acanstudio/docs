本地Repo
========

Git基本概念
------------

在Git中，每个版本库都叫做一个仓库（repository，通常缩写为repo），每个仓库可以简单理解成一个目录，这个目录里面的所有文件都通过Git来实现版本管理，Git都能跟踪并记录在该目录中发生的所有更新。假如我们现在建立一个仓库（repo），那么在建立仓库的这个目录中会有一个".git"的文件夹。这个文件夹非常重要，所有的版本信息、更新记录，以及Git进行仓库管理的相关信息全都保存在这个文件夹里面。所以，不要修改/删除其中的文件，以免造成数据的丢失。

进一步的讲解请参考下面一张图，大概展示出了我们需要了解的基本知识（注意，".git"目录中还有很多别的东西，图中并没有涉及，这里也不做解释了）。

![step_local_1](#UPLOAD_URL#/images/step_local_1.png)

根据上面的图片，下面给出了每个部分的简要说明：
    * Directory：使用Git管理的一个目录，也就是一个仓库；包含我们的工作空间和Git的管理空间。
    * WorkSpace：从仓库中checkout出来的，需要通过Git进行版本控制的目录和文件；这些目录和文件组成了工作空间。
    * .git：存放Git管理信息的目录，初始化仓库的时候自动创建。
    * Index/Stage：暂存区，或者叫做待提交更新区；在提交进入repo之前，我们可以把所有的更新放在暂存区。
    * Local Repo：本地仓库，一个存放在本地的版本库；HEAD会指示当前的开发分支（branch）。
    * Stash：是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

本地repo上的基本操作
-----------------------
 
#### 创建仓库

通过"Git Bash"命令行窗口进入到想要建立版本仓库的目录，通过"git init"就可以轻松的建立一个仓库。这时，我们的仓库目录中会自动的产生一个".git"文件夹，这个就是我们前面提到的Git管理信息的目录。

相关命令
<pre>
$ git init

# 查看版本库当前状态，该命令使用频率很高，随时掌握版本库的当前状态
$ git status 
</pre>

#### 添加

现在我们在仓库中新建一个"calc.py"的文件，文件内容如下。

<pre>
def add(a, b):
    print a + b
    
if __name__ == "__main__":
    add(2, 3)
</pre>

相关命令和状态
<pre>
$ git status 
# 显示"calc.py"没有被Git跟踪，并且提示我们可以使用"git add <file>..."把该文件添加到待提交区（暂存区）。注意，这时的更新只是在WorkSpace中。

$ git add calc.py
$ git add .
$ git status
# 发现文件已经被放到暂存区。这时的更新已经从WorkSpace保存到了Stage中。

$ git commit -m
# 提交更新了。-m后面跟的是对commit的描述（message）。这时的更新已经又从Stage保存到了Local Repo中。
</pre>

通过上面的操作，文件"calc.py"就成功的被添加到了仓库中。
 
#### 更新

假设现在需要对"calc.py"进行更新，修改文件后，查看WorkSpace的状态，会发现提示文件有更新，但是更新只是在WorkSpace中，没有存到暂存区中。

<pre>
def add(a, b):
    print a + b
    
def sub(a, b):
    print  a - b
    
if __name__ == "__main__":
add(2, 3)
</pre>

同样，通过add、commit的操作，我们可以把文件的更新先存放到暂存区，然后从暂存区提交到repo中。

注意，只有被add到暂存区的更新才会被提交进入repo。比如下面的一系列操作，操作结束后只有"multi"函数的更新会被提交到repo中，"div"函数的更新还在WorkSpace中。这点应该也是比较容易理解的。

<pre>
def multi(a, b):
    print a * b
    
def div(a, b):
    if b != 0: 
        print a / b    
</pre>

#### git diff

"git diff"是一个很有用，而且会经常用到的命令。基于上面的例子，我们通过"git diff"来查看WorkSpace和Stage的diff情况，当我们把更新add到Stage中，diff就不会有任何输出了。

当然，我们也可以把WorkSpace中的状态跟repo中的状态进行diff，命令如下，关于HEAD，将在后面解释。

<pre>
$ git diff
$ git diff HEAD~n
</pre>

#### 撤销更新

更新可能存在三个地方，WorkSpace中、Stage中和repo中。下面就分别介绍一下怎么撤销这些更新。

Git使用reset命令来实现更新的撤销操作，reset有两个常用的选项--hard和--soft
    * --hard：撤销并删除相应的更新
    * --soft：撤销相应的更新，把这些更新的内容放的Stage中

相关命令
<pre>
# 撤销WorkSpace中的更新
$ git checkout -- <file>
# 撤销WorkSpace中指定文件<file>的更新，注意一定不要漏掉"--"
# 注意，在使用这种方法撤销更新的时候一定要慎重，因为通过这种方式撤销后，更新将没有办法再被找回。

# 撤销Stage中的更新,WorkSpace中的更新通过add更新到了Stage
$ git reset HEAD <file>
# 该命令把暂存区的更新移出到WorkSpace中。
# 如果想继续撤销WorkSpace中的更新，请参考上面一步。

# 撤销repo中的更新
$ git log
# 通过这个命令我们可以查看commit的历史记录。可以看到我们进行的三次提交。
# 通过HEAD撤销更新,HEAD对应当前分支中最新的提交
$ git reset --hard HEAD^
$ git reset --hard HEAD^^
$ git reset --hard HEAD~n
# 通过commit-id撤销
$ git rest --hard 1a72f49ae49c1716e52c12f2b93fdcef6aac0886
$ git rest --hard 1a72f49

# 恢复reop中被撤销的提交
$ git reflog
# git log只是包括了当前分支中的commit记录，而git reflog中会记录这个仓库中所有分支的所有更新记录，包括已经撤销的更新。
$ git reset --hard HEAD@{n}
$ git reset --hard 1a72f49
</pre>

#### 删除文件

在Git中，如果我们要删除一个文件，可以使用下面的命令,"git rm"相比"rm"只是多了一步，把这次删除的更新发到Stage中。

<pre>
$ rm <file>
$ git rm <file>
</pre>

总结
-----

通过这篇文章，了解了Git的一些基本概念。然后介绍了Git中常用的命令来操作本地repo，内容比较多，但是都是基本的，只要自己动手操作一遍就清楚了。

到这里，下面命令流图中的命令都已经被介绍到了。相信通过这些命令，我们已经可以对自己的本地repo进行Git的版本管理了。

![step_local_2](#UPLOAD_URL#/images/step_local_2.png)

