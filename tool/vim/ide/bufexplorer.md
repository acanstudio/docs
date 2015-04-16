bufexplorer安装以及简单小结
============================

bufexplorer插件可以打开历史文件列表以达到快速切换文件的目的，有了前面几个插件的安装经验，这个就非常简单了，而且这个插件安装后不需配置_vimrc文件（当然随着学习深入，可能需要增加一些配置以适应特定需求）。

1、直接从 http://www.vim.org/scripts/script.php?script_id=42 下载bufexplorer，解压后得到两个文件夹plugin和doc，直接与~vim/vim73下的同名目录合并即可，如下图：

     

   重启Vim，打开一个文件，可能观察不到效果，这是因为只有打开多个文件才能看到效果

 

2、还看不到效果？

    试试命令模式下输入\be（我的这个命令不管用）或:BufExplorer进入Buffer列表。

 

3、小结

    到目前为止完成了Vim的配色方案设置、ctags、taglist、winmanager和bufexplorer安装。

    taglist的打开关闭方法是命令模式下输入 :Tlist

    winmanger的打开关闭方法是命令模式下输入F8或者:WMToggle

    bufexplorer的打开关闭方法是命令模式下输入\be或者:BufExplorer

   
