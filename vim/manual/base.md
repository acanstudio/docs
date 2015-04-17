Vim基本操作
==============

获取帮助：:help<Enter>；
光标移动：hkjl；
关闭当前窗口：:q<Enter>；
离开Vim：:qa!<Enter>；(慎重操作，会丢失所有修改信息)
移动到标签：CTRL-]；
从标签返回：CTRL-T,CTRL-o；(重复会继续向后)
启用鼠标：:set mouse=a；

获取特定帮助：:help [command]<Enter>；
* 类别           前缀     例子
* 普通模式命令   无       :help x
* 可视模式命令   v_       :help v_u
* 插入模式命令   i_       :help i_<Esc>
* 命令行模式命令 :        :help :quit
* 命令行编辑     c_       :help c_<Del>
* Vim命令参数    -        :help -r
* 选项           ''        :help 'textwidth'
获取相关帮助信息：:help word -> CTRL-D


