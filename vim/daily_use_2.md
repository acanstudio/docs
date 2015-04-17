文件保存高级篇
--------------

以下部分命令在之前的篇幅中有涉及过，有句话说的好：*vim对新手最痛苦的是选择太多，不知所措，对老手来说最让人快乐的是一个问题总有不同的解决方法，而对寻找最优方法乐此不疲，细心的读者相信您都能从中总结出自己的规律以及经验来。

<pre>
:w new_file # 将缓冲区内容保存为new_file文件，原文件内容不更改。
:20,$w new_file # 将文件20行处到结尾保存为new_file文件
:.,20w new_file # 将光标所在行到第20行保存为new_file 文件
:20,30w >> new_file # 追加20至30行内容到new_file文件中
</pre>

一个文件copy到另一文件

<pre>
:r filename # 把filename中的内容插入到光标所在行的下一行
:100r filename # 把filename中的内容插入到100行的后面
:$r filename # 插入行尾
:0r filename # 插入行首
:/parttern/r filename # 还可以使用正则表达式，插入到匹配出的后面一行，需要注意的是如果有多处匹配，它只插入到首个匹配的地方。
</pre>

标记 标记又称为书签，在某个位置打上标记后，在别处编辑完，通过命令可以回到标记处（以下命令模式中执行）

mx  将当前位置标记诚x（此处的x可以是任意字母）
<pre>
'x  （单引号）光标移到标记x处的行首
`x   (反引号）光标移到标记x处
``  （双反引号）当前光标处于标记处来回切换
''  （双引号） 当前光标所在行处与标记处来回切换，光标定位在行首
</pre>

ab与map命令
-----------

### ab命令

ab命令可以将一长串字符用缩写来定义，这有点象Linux中的alias，比如Linux中的ll命令就是ls -l的别名，ab的语法为：

:ab abbr phrase

abbr就是对phrase的简写，在insert 模式中，输入abbr，再按非字母字符（比如空格，点号等）Vim就自动把phrase插入到光标处位置。此情景一般用在频繁输入的字符中，通常建议abbr选择使用频率很低的字符，比如Eclipse常用的一个快捷键syso，你可以定义成如下：

:ab syso  public static void main(String[] args) 

这样一来，每次输入syso的时候，整个main方法就自动插入到文本行了。如果你就是想输入'syso'这个原生字符串，那么您可以用下面这个命令取消：

:unab syso

列出当前定义的缩写有哪些可以用命令：

:ab

### map命令

图

上图是执行:map命令显示的内容，我们暂且先不对图做说明，稍后再分析，map的功能比ab更强大，它不仅可以在insert 模式下定义宏（快捷键）而且可以normal，visual等模式下定义。其语法举例说明：（目标：在normal下用lv选中光标所在行）

:map lv 0v$ 

解析：0代表光标移至行首，v就是visual模式（该模式下可以通过hjkl来选中文本），$代表行尾，这样一来，在normal模式输入lv就能选中光标所在行了。

:map 列出所有已定义的映射命令
:unmap lv 取消lv映射的命令
:mapclear  清空所有映射

需要注意的是：

    * 默认情况下，map命令是作用在normal模式下的
    * 如果是想在virsual模式下新建某个命令的宏，可以使用:vmap，举例：:vmap d <esc>dd就可以在virsual模式下把光标所在行删除。<esc>是纯粹的5个原始字符，意思是回到normal模式。
    * 默认情况下，map是采用递归映射的，比如a映射成b，:map a b，然后c 又映射成了a，:map c a ，那么最终c也会自动映射成b，等同于:map c b，您现在可以试一试a,b,c的效果都是光标向前移动一个单词的长度。如果要不想使用递归，那么就要用:noremap
    * 现在你应该能看明白上图的内容了吧，第一列就是宏会在哪中模式下生效，第二列代表快捷键了，第三列就是真正的命令序列集合了。您可以注意一下最后一个命令：（Shift+Insert）就是前些天分享过的，代表在normal模式下粘贴系统剪切板中的内容。

实用例子：

<pre>
:map <C-a> <Esc>ggVG   实现类似于Widnows下的Ctrl+a全选 
:inoremap ( ()<esc>i   插入模式下输入'('后自动补全')'，同理还可以实现'['，'{'
</pre>

ps："+y可以把光标所在行或选选中的字符copy到系统剪切板中。
更多的例子就要靠您的创造力和想象力的，如果您能把基本的命令学好了，这些命令组合在一起使用的话，威力无比。

多窗口
-------

题外话之Vim的简史：Vim是vi演化过来的，其全名叫vi Improved.最初是由一个叫做Bram的大神在vi的基础上开发出来的。她的设计目标是成为一个可靠而且可以为专业程序员所依赖的编辑器。

之前谈过一点点多文件编辑的内容，今天稍微详细的讲解多窗口的编辑。

默认情况下，Vim只为一个session打开一个窗口，可以用参数**-o**来打开多个窗口，如:vim -o file1 fiel2，默认这个session会水平分割两个窗口显示，另外参数o后面还可以跟数字:vim -o3 file1 file2 这样Vim会打开三个窗口，最后一个窗口会留空白.
打开窗口

如果vim已开启，那么在normal模式如下命令使用：
水平分割窗口

:split             当前窗口一分为二，两个窗口显示相同内容。  
:10split           新窗口的高度10行
:split otherfile   新窗口中打开otherfile   
:new               功能和split一样  
:sp                split的缩写形式  
ctrl+w+s           分割窗口的快捷方式
:q                 关闭当前窗口

垂直分割窗口

:vsplit 以上所有命令都适用于打开垂直分割窗口，只要在前面加v(vetical)
窗口光标移动：
鼠标操作

gvim默认支持鼠标移动光标操作。
vim可以设置 :set mouse=a,我猜a就是available的意思。
键盘操作

ctrl+w+k  使用ctrl+w（window)结合hjkl来移动。先按住CTRL+w，在按k，光标就移到上面窗口。hjkl前面可加数字，移动多个窗口
ctrl+w+T  大写T）移动当前窗口至新的标签页（tab，下节专业讲讲标签页）
ctrl+w+K  （大写K）HJKL四个组合命令（移动并回流窗口命令，窗口和光标一起移动）

调整窗口尺寸

gvim鼠标支持拖拉动作来改变窗口大小。我想你不会这么做，命令行才是高效率工作。
ctrl+w结合+-= 当然+-=前面可以接数字，分表代表增大、减小、均分窗口。
resize -4 明确指定窗口减少多少
ctrl+w结合< > 增加窗口宽度

Visual 模式
------------

Visual模式，即可视化模式，可以对所选择的文本进行各种操作，Virsual模式可以分为三种，字符（Characters）、行（line）、矩形块（rectangular block），既然是Visual模式，肯定是和字母v相关的操作，前面的一些篇章也用到过v模式。
viwc

今天呢，就只讲一点点有关V模式的用途吧，在windows中替换一个单词惯用的手段就是先找到这个单词，鼠标双击该单词，选中之后直接输入新的单词就Ok了，但是使用Vim，你就应该摒弃鼠标，甚至四个方向键也不要去碰。那么在Vim中，概括起来就是四个字<E>f{char}viwc（请看小标题，这里貌似有十多个字儿，且慢，一个个解析下：<E>：Esc，进入normal模式，f：查找字符串，当然还可以用“;”或者“，”继续往后或往前找，v：visual模式，iw：选中整个单词，c：删除单词，进入插入模式），这样整个单词就会删除，接着就可以插入你想替换的单词了。其次，在Visual模式下，hjkl光标移动的键同样是可用的。对了，在normal模式下“.”可以重复执行上一次操作，有点象Python中的下划线“_”表示最后一个表达式的值一样。例如你最后执行的命令dd，那么按“.”就会继续删除当前行。（以后如果突然想起一个实用的东东，如果前面没介绍过的，我就顺便查到文章里头了）。

<pre>
# 另附：为了彻底甩掉对四个箭头移动光标的依赖，在.vimc文件中可配置:
nnoremap <up> <nop>
nnoremap <left> <nop>
nnoremap <right> <nop>
nnoremap <down> <nop>
</pre>

Visual 模式的三种子模式(基于字符，行，块）可以对不同文本域进行处理，这一小节看看如何使用这三种模式以及他们之间如何切换。

字符可视化模式可以对任何单个字符或字符串甚至是多行进行处理，通常适用于处理单词或者词组，如果是想处理整行，那么就可以使用（line）行可视化模式，*块可视化* 则可以对文档区域操作，支持列操作。normal 模式下，命令对应的Visual表如下：

v         基于字符的Visual模式
V         基于行的Visual模式
Ctrl+v    基于块的Visual 模式
gv        重新选取最后一次使用Visual模式选中的文本

Visual模式之间的切换

如果当前是在字符Visual模式下，V就能切换到基于行的Visual模式，Ctrl+v就是切换到基于块的Visual模式下，来回的按v能在normal模式和字符Visual模式下切换。此规则同样适用与另外两种Vrsual模式。
光标在选择区域首尾切换

首先我们在看这么一个图：
vim17_1
当前光标在第一行的h位置，我想实现的效果是通过光标在选择区域两端切换的方式把_here to here_ 都选中，那么命令o就能用来区域首尾切换的。其对应的命令如下图所以：
vim17_2

### Visual-Block 模式

Visual-Block模式一个非常强大的功能就是它支持列操作，比如在某个代码块每行的行首插入注释符号。举例说明：假如有如下Python代码，我想把它全部注释

for e in exclude:
if e.endswith(".py"):
        try:
        os.remove("%sc" % e)
        except:
                pass
                try:
                with open(e, "r") as f:
        exclude[e] = (f.read(), os.stat(e))
        os.remove(e)
        except:
                pass

    光标定位到代码块的行首，Ctrl+v进入Visual-block模式
    光标向下移动，直到选择所有代码行的第0列
    输入I（光标前插入字符），此时你会发现光标跳到了代码块的开头处，此时已经是insert 模式了，现在就插入python的注释字符'#'

    按Esc键，此时你会发现代码块所选区域都打上了注释符号，如下所示：

    #for e in exclude:
    #if e.endswith(".py"):
    #   try:
    #       os.remove("%sc" % e)
    #   except:
    #      pass
    #      try:
    #      with open(e, "r") as f:
    #   exclude[e] = (f.read(), os.stat(e))
    #   os.remove(e)
    #   except:
    #        pass

第二个例子：下面是一段JavaScript片断：

var foo = "a"
var bar = "bcd"
var fb = foo+bar

我们知道js中大部分浏览器都能忍受后面没有分号结尾的语句，但是并不推荐这样做，因为我们有必要给他们在行末都加上分号，我们知道vim吸引人的地方之一就是一个问题往往有不同的解决方案，这里我们至少有两种方法，1.替换法：:1,3s/$/;/g（这里的1，3是第一到第三行）2.在Visual block模式下append（追加）。

我们观察上段代码发现每行的语句的长短不一，那有如何批量的加上";"呢，这里关键的一个命令是$，美元符号定位了行未。操作步骤基本还是和第一个例子差不多。只需在选中代码块的时候要注意是：Ctrl+v jj$，这样就能选中到每一行的行末。接着输入A命令表示在行末追加字符，输入“;”再按Esc大功告成了。最终的效果：

var foo = "a";
var bar = "bcd";
var fb = foo+bar;

Vim 编码设置
-------------

Vim的编码选项

vim编码涉及四个概念，分别是enc,fenc,fencs,tenc,一般乱码多是因这些参数设置不正确引起的，要想彻底摆脱vim的乱码问题，还是把这四个概念理清楚了，下面详细介绍之。

enc(encoding)，这是Vim内部使用的编码，如buffer，寄存器中的字符串。在Vim打开文本后，如果它的编码方式与它的内部编码不一致，Vim会先把编码转换成内部编码，如果它用的编码中含有没法转换为内部编码的字符，那么这些字符就会丢失掉。默认值是系统的locale来决定的，比如在windows下一般就是gbk或gb2312的，而在linux下就是utf-8。可以用命令:set enc查看当前vim的enc是什么值，笔者的windows显示的是cp936这里的cp936其实相当于gb2312，指系统的第936号编码格式。

fenc(fileencoding)，fenc为当前缓冲区（当前Vim打开这个文件）文件自身的编码，从磁盘读文件时，Vim会对文件编码检查，如果文件的编码与Vim内部编码（enc）不同，Vim就会对文本做编码转换，将fenc设置为文件的编码。Vim写文件到磁盘时，如果enc与fenc不一样，Vim就做编码转换，转换成编码fenc保存文件。在windows下你可以借由notepad++等编辑器检查文件是什么编码的。由于fenc是在打开文件时由Vim自动检测的，所以如果文章中有乱码也没法通过重新设置fenc来纠正，设置fenc只能改变文本的编码格式。

fencs(fileencodings)，这是一个字符编码的列表，编码的自动识别就是通过设置fencs实现的。当打开一个文件时，Vim会按照fencs中编码的顺序进行解码操作，如果匹配成功就用该编码来进行解码，并把这种编码设为fenc的值。这里的匹配成功指的是Vim能正确解码，不会出错，但是不保证没有乱码，所以fencs编码列表的顺序设置很关键，由于lanin1是iso8859-1，属于国际化的标准编码，他能表示任何字符，也就用于也不会出错，但是我们看到的可以是“乱码”。 所以一般fencs设置的顺序是这样子的：lan1放到最后面 
<pre>
set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
</pre>

tenc(termencoding)，终端使用文本编码，或者说是Vim用于屏幕显示时的编码，显示的时候Vim会把内部编码转换为屏幕编码再输出，也就是说我们从屏幕上看到的字符都是tenc编码的字符，如果为空，默认就是enc。windows平台Gvim会忽略掉tenc。一般就是从一个终端远程登陆到linux系统时候tenc会起作用。

乱码问题

如果碰到了乱码问题，只要你把enc，fenc统一设成utf-8问题都会解决了。下面这段配置就是我的Vimrc文件的关于解决乱码配置的代码段：

<pre>
" 设置vim内部编码格式
set encoding=utf-8

" 解决windows下如果encoding设置utf-8，菜单会乱码问题
set langmenu=zh_CN.UTF-8
language message zh_CN.UTF-8
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim

"默认文件编码
set fileencoding=utf-8 
set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
</pre>

