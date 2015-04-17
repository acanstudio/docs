Git对象模型
============

基本概念
---------

Git的对象模型是了解Git工作原理的基础。下面通过一个实例展示Git对象模型。

在Git系统中有四种类型的对象，所有的Git操作都是基于这四种类型的对象。
    * "blob"：这种对象用来保存文件的内容。
    * "tree"：可以理解成一个对象关系树，它管理一些"tree"和"blob"对象。
    * "commit"：只指向一个"tree"，它用来标记项目某一个特定时间点的状态。它包括一些关于时间点的元数据，如时间戳、最近一次提交的作者、指向上次提交（初始commit没有这一项）。
    * "tag"：给某个提交(commit) 增添一个标记。
 
commit对象记录了版本库的整个生命周期，包含的信息也更复杂，一个commit对象一般包含以下信息：
    * 代表commit的哈希值
    * 指向tree 对象的哈希值
    * 作者
    * 提交者
    * 注释

#### SHA1哈希值

在Git系统中，每个Git对象都有一个特殊的ID来代表这个对象，这个特殊的ID就是我们所说的SHA1哈希值。SHA1哈希值是通过SHA1算法（SHA算法家族的一种）计算出来的哈希值，对于内容不同的对象，会有不同的SHA1哈希值。如果你读过前面一篇文章，就肯定还记得我们是怎么根据commit id撤销更新的，这里的commit id就是一个SHA1哈希值。

Git对象模型实例
----------------

跟对象相关的命令
<pre>
$ git log --pretty=raw
# 查看每个commit的SHA1哈希值，也可以得到这个commit对应的tree的哈希值。

# git cat-file是研究Git对象模型的利器，可以通过这个命令查询特定对象的信息
# git cat-file -t key：通过一个对象的哈希值可以通过这条命令查看对象的类型（blob、tree、commit或tag）
# git cat-file -p key：通过对象的哈希值可以查看这个对象的内容
$ git cat-file -t *******
$ git cat-file -p *******
</pre>

下面我们通过一个例子来认识一下Git的四种对象，为了更加清楚，这里将一步步展示经过一系列操作后对象的关系变化。

第一步：新建一个仓库，添加一个"calc.py"的文件，并提交到版本库，当前版本的对象分布如下图所示

![step_sha1_1](#UPLOAD_URL#/images/step_sha1_1.png)

第二步：更新"calc.py"文件，添加sub函数，提交更新至版本库，对象分布图如下图所示：

![step_sha1_2](#UPLOAD_URL#/images/step_sha1_2.png)

这里需要注意的一点，Perforce、SVN和CVS属于"增量文件系统" （Delta Storage systems），它们每次只存储提交(commit)之间的差异。而对于Git，它会把你的每次提交的文件的全部内容（snapshot）都会记录下来。

第三步：增加一个"app.py"；增加"advance"文件夹，包括"__init__.py"和"calc.py"，对象分布图如下图所示

![step_sha1_3](#UPLOAD_URL#/images/step_sha1_3.png)

总结
-----

Git对象模型就像是Git系统特有的文件系统，以特定的方式存储更新的内容、元数据以及版本历史信息。

通过Git对象模型进一步熟悉了Git的工作原理，相信有了这些知识，我们就可以分析git命令背后到底发生了什么。
