abbreviation
-------------

Vim有一句哲学是这样说的：“if you write a thing once,it it okay,However,if you're writing it twice or more times,then you should find a better way to do it"。这句话估计也是引用软件开发里面的DRP（Don't Repeat Yourself）原则。如果你老是需要重复的写一些相同的东西，此时你就应该使用"abbreviation（缩写）".

<pre>
:abbreviate 作用于所有模式   （ab） 
:iabbrev    仅作用于插入模式 （iab）
:cabbrev    仅作用于命令行模式（cab）
</pre>

abbreviation可以用在很多有意思的地方，比如：

    * 纠正错误的拼写::iabbr teh the
    * 程序中你能想到的模版语句：:iabbr forx for(x=0;x<100;x++){<cr><cr>}
    * 简化命令的输入：cabbr cse colorscheme evening

如果你是Java程序员，如下命令毫不逊色于Eclipse
<pre>
abbr psvm public static void main(String[] args){<CR>}<esc>O  
abbr sysout System.out.println("");<esc>2hi  
abbr sop System.out.println("");<esc>2hi  
abbr syserr System.err.println("");<esc>2hi  
abbr sep System.err.println("");<esc>2hi  
 
abbr forl for (int i = 0; i < ; i++) {<esc>7hi  
abbr tryb try {<CR>} catch (Exception ex) {<CR> ex.printStackTrace();<CR>}<esc>hx3ko  
abbr const public static final int  
 
abbr ctm System.currentTimeMillis()  
abbr slept try {<CR> Thread.sleep();<CR>}<esc>hxA catch(Exception ex) {<CR> ex.printStackTrace();<CR>}<esc>hx3k$hi  
</pre>

写程序追求的高内聚，低耦合，同样，毫无疑问，Vim也遵循同样的原则，如果我们有上十条百条这样的缩写命令，如果都挤在vimrc配置文件中，这样过显得很难管理，因此我们可以把专门用于缩写的命令放置在单独的文件中，然后在vimrc文件中引用就ok，:source $VIM/abbreviation.vim

你有没有想过一个问题，如果把forx设置成了缩写格式，那么有时候我本意是输入'forx'呢？

方法一：就是把它的映射取消掉una forx，这样有个缺点是下次我又需要这个缩写了，这时又不得不重新捡回来。

方法二：写一个函数，在每次输入'forx'的时候询问是作为普通字符串还是作为缩写呢？函数如下：

<pre>
function! s:forxAsk(abbr,expansion)
  let answer = confirm("使用缩写'" . a:abbr ."'?",
                         "&Yes\n&No",1)
  return answer ==1 ? a:expansion : a:abbr
endfunction

iabbrev <expr> forx <SID>forxAsk('forx','for(x=0;x<100;x++)')
</pre>

函数中abbr是缩写，expansion就是全写，这样一来，每次输入forx时，就会弹出一对话框询问你是使用缩写还写不使用。当然这种方式显得比较笨拙，另外一个目的也是告诉大家如果写函数。

方法三：使出杀手锏，输入forx完成后，按Ctrl-v（windows系统按Ctrl-q）就能避免尴尬了。

Tagelist初体验
---------------

使用taglist需要在系统平台上安装ctags，同时为vim安装taglist插件。

宏---Record、Play
------------------

写这篇文章的时候想到了读高中那会儿买的第一个电子产品，某某高复读机，话说是为了学英语，呵呵，你懂的，其实是为了好玩。当时差不多花了300担，父母在子女的教育方面可是毫不手软，想想如果那时开始接触计算机互联网相关的东西了，买的就是一台电脑，我离那“一万小时定律”就要早几年完成了...言归正传。
tu

今天要说的其实就和这个复读机相关，复读机在按下复读的按钮后，就开机录制需要复读的内容，再按一下录制完成，接下来就可以播放了。Vim中也有与之惊人相似的操作，如果想重复某个操作，就可以用**宏**来完成，还记得以前讲过的一个命令吗：.就是这个**点**可以重复执行最后一次操作，但是这个.的功能比较弱，没法组合使用，如下代码，想在每行末加上分号";"：

int a = 1
int b = 2
int c = a+b
print a
print b
print c

如果是用.来实现的话，首先在第一行执行$a;，然后重复5次执行j$.，这样算下来你要敲击的键总数在15次之多，但是我们用Record/Play的话，即使是100行代码，按键也不会超过10次。命令闪亮登场：q，就是这个q，它的威力很猛。接下来就详细介绍如何操作q来实现上述需求。

    normal 模式下输入q启动recoding，q后面跟任意a-z的小写字母比如m，这个字母就是宏的名字，接下来你要执行的操作就会记录在这个宏中。
    执行我们的任务：“行末加分号”，命令是：$a;<Esc>j$，这条命令意思就是：移动行尾插入分号，退到normal模式，光标移动到下一行的末尾。
    再次输入q，表示录制结束
    录制结束后我们就可以play了，输入@m就会执行宏中的操作，m是第一步中使用的宏的名称，5@m表示重复执行5次。这样，所有行都给加上分号了，真是好使。

再举一例：实现如下效果：从1到100，每行+1。

1
2
3
...
100

命令：首先在第一行插入1，然后光标定位了“1”处，进入normal模式，开始录制：qmyyp<Ctrl>aq，（解释：yyp：拷贝一行再粘贴在新的一行，<Ctrl>a：数字+1）后然执行98@m，收工。

filetype-文件类型检测
========================

Vim可以检测到打开的文件类型，并根据不同类型的文件，调用不同的配置和插件。比如高亮显示关键字，函数名等。

<pre>
" 查看Vim的文件类型检测功能是否已打开，默认你会看到：detection:ON plugin:OFF indent:OFF"
:filetype 

" 常用命令"
:filetype off
:set filetype
:set filetype=python
:filetype plugin on
</pre>

detection：检测文件类型
----------------------

默认情况vim会对文件自动检测文件类型，也就是你看到的'detection:ON'，同样你可以手动关闭:filetype off。因此我们看到dection也是ON，所以Vim会很友好的显示文本，可以用:set filetype查看当前文件是什么类型了。类似的如果如上图中file.txt文件的filetype设置为python那么就和普通的python文件一样的显示效果了：set filetype=python。

另一种方式就是在文件内容中指定，Vim会从文件的头几行自动扫描文件是否有声明文件的类型的代码，如在文件的行首加入# vim: filetype=python，Java文件变通的做法/* vim: filetype=java */，总之就是把这行当作注释，以致于不影响文件的编译，这样Vim不通过文件名也能检测出文件是什么类型了。
 
plugin：加载相关插件
---------------------

如果plugin状态时ON，那么就会在Vim的运行时环境目录下加载该类型相关的插件。比如为了让Vim更好的支持Python编程，你就需要下载一些Python相关的插件，此时就必须设置plugin为ON插件才会生效，具体设置方法就是:filetype plugin on

indent：缩进
-------------

不同类型文件有不同的方式，比如Python就要求使用4个空格作为缩进，而c使用两个tab作为缩进，那么indent就可以为不同文件类型选择合适的缩进方式了。你可以在Vim的安装目录的indent目录下看到定义了很多缩进相关的脚本。具体设置方法：filetype indent on。

以上三个参数，可以在.vimrc文件中写成一行：filetype plugin indent on

http://liuzhijun.iteye.com/category/270228

行复制与移动
-------------

是整行移动和拷贝，涉及的命令是：:m和t。这两个命令其实是move和copy的简写形式。 其实整行的拷贝相信你能用yank解决，但是它有一个缺点就是必须把光标移到要拷贝的行才能执行该操作，然而:copy和:move命令可以在任何地方拷贝或者移动任意一行或者多行。

copy命令格式：:[range]copy{address}，range表示拷贝范围，address表示目标地址。举例来说：把下面三行if语句块拷贝到main代码段中去，不管此时你的光标在何处，现在假设光标在main那行：

if choise=='n':newuser()
if choise=='e':olduser()
if choise=='q':done=True

if __name__=='__main__':

我们可以用:1copy.把第一行拷贝到光标的下一行（.代表当前行），如果三行全拷贝：:1,3copy.，copy的另外两个写法:co或者:t。 常用命令：
:3t. 拷贝第三行到当前光标的下一行
:t3 拷贝当前行到第三行的下一行
:t. 拷贝当前行到光标的一下行，相当于Yp和yyp
:t$ 拷贝当前行到最后一行
:'<,'>t0 拷贝所选区域到文本的开头处，这里的操作步骤是：现在visual 模式下选中文本，然后输入:，接着t0。

move：move的操作完全和copy是一样的，它的简写格式有mo和m。可以对照上面的例子重复操作一遍。更多帮助可以查看:h :move和:h copy。


跨行执行〈Normal模式下的〉命令
-------------------------------

以往，要想在多行执行normal 模式下命令可以通过定义宏来重复操作，今天讲个新鲜的。:normal命令。之前讲过一个列子，实现注释多行代码这样一个需求，可选的方法如下三种方式：（当然你还可以相出更多的办法来）

import urllib2
def html():
    f = urllib2.urlopen("http://www.douban.com")
    print f.read()

    替换：:%s/^/#/g
    visual block：gg<Ctrl-v>I#<Esc>
    注释第一行后用.重复执行每一行

我们可以在第三种方法之上用normal命令实现上述需求，步骤：

    光标定位到首行，执行：I#<Esc>
    jVG选中之后的所有行
    :'<,'>normal .这样刚刚选中的行都将执行.代表的最后一次操作。注：只要输入:就能实现:'<,'>，你可以注意VIm的左下角的提示。

第四种方法：:%normal I#，%代表这个文件，当然你可以选择具体的范围，如：:1,4normal I#

总结：:normal命令可以执行任何normal 模式下的命令，更多帮助：:help normal。对了，上面这个例子你还可以用“宏，record”来达到要求，如果没有想起来，翻开Recode/Play试试吧。


高亮所有搜索模式匹配

今天的内容很简单:-)

 

* 向后搜索光标所在位置的单词
# 向前搜索光标所在位置的单词
n和N可以继续向后或向前搜索匹配的字符串

:set hlsearch 高亮所有匹配字符串
:nohlsearch 临时关闭，他的缩写形式是：:noh
:set nohlsearch 彻底关闭，只有重新:set hlsearch才可以高亮搜索

:nnormap <silent> <Space> :nohlsearch<Bar>:echo<CR>按空格关闭高亮，清空所有已经显示的
如果你想在高亮与不高亮之间快速切换，可以做一个映射 ： :noremap <F4> :set hlsearch! hlsearch?<CR>
按回车，临时返回高亮搜索
:nnoremap <CR> :nohlsearch<CR><CR>


全局命令
---------

全局命令在Vim中有这举足轻重的作用，特别对于那些重复性的工作尤为有效，它能对匹配的所有行执行某个命令，先来看看它的语法：

<pre>
:[range]global[!]/{pattern}/{command}

# [range]指定作用范围，默认global命令作用于整个文件，不像:normal等命令，normal默认是作用于当前行。
# {pattern}：对于range范围内的文本，如果匹配pattern，就会执行command，'[!]'相当于取反（也可以用vglobal），也就是不匹配patten的行。
# command默认是print，打印文本行。
</pre>

举例
<pre>
# 实现Linux命令tac的功能（与cat对应的一个命令，反向输入文本行）
# 这里的^表示所有行（包括空行），.表示非空行，m 0表示将当前行移至第0行。这样就实现了tac的功能。
:g/^/m0 

# prep：其实就是的缩写，re表示regular express，p表示print
:g/re/p 

# vred：删除不匹配re的行，v就是vglobal的缩写，比如删除所有不包含href的行：
:v/href/ 

# 排序：对于下面的css片段:
div{
    border:0;
    margin:0;
    padding:0;
    font-size:12px;
    font:inherit;
    vertical-align:baseline;
}

# 我们想把{}中的样式按照字母的顺序排列成如下:
div{
    border:0;
    font-size:12px;
    font:inherit;
    margin:0;
    padding:0;
    vertical-align:baseline;
}
:g/{/ .+1,/}/-1 sort

# 这个命令看起来很复杂，好像也不符合global的语法。其实对于global语法，可以扩展成：:g/{pattern}/[range][cmd]，
# 就是说cmd前面还可以指定range.因此我们可以把上面的命令来做一次剖析： .+1,/}/-1 sort 这个命令就相当于[range][cmd]，
# 这里的range范围为：当前行（.）的下一行（+1）直到（,）匹配（}）的前一行（-1）。如此一来这条完整的命令就好解释了，
# 表示{的下一行一直到}的上一行执行sort命令。
</pre>

ctags
------

简单来说，Ctags的作用就是在一个包含有源代码文件的目录下生成一个tag文件（可以理解为索引文件），以便vim编辑器能快速定位到文件某个位置的工具。那到底怎么使用它呢？

如果你正确安装了ctags的话（参考：http://liuzhijun.iteye.com/blog/1843522 ），直接在命令行就可以直接运行ctags命令。

ctags -R *：
直接在命令行运行上面这条命令，意思是：为当前目录以及子目录的所有文件创建一个tags文件，vim启动时就会自动载入该文件，tags文件中包含什么内容呢？你可以试着打开看看。一般包含的对象包括：

    类（class）、接口（interface）、枚举 变量 成员变量，方法

gvim -t $tag：打开定义有$tag的文件，比如 gvim -t Person就会打开包含有Person变量或类型等关键字的文件。
ts：列出哪些地方出现有$tag关键字
tn：如果打开有tag出现在多处地方，就可以用tp切换，移动到下一处
tp：与上面的命令作用是一样的，移动方向相反
Ctrl+]：这个命令可以让光标直接定位到$tag的定义的地方
Ctrl+T：回到最初打开文件的位置

当然，想充分利用好tags还得和taglist等插件结合起来用
