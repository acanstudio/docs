Vdebug
=======

安装，需要注意的是，Vim的运行环境应事先安装好Python，同时Vim要支持python.
<pre>
# 通过Vundle安装
" for PHP
Bundle 'joonty/vdebug.git'
</pre>

调试快捷键：

<pre>
<F5>: start/run (to next breakpoint/end of script)
<F2>: step over
<F3>: step into
<F4>: step out
<F6>: stop debugging
<F7>: detach script from debugger
<F9>: run to cursor
<F10>: set line breakpoint
<F11>: show context variables (e.g. after "eval")
<F12>: evaluate variable under cursor
:Breakpoint <type> <args>: set a breakpoint of any type (see :help VdebugBreakpoints)
:VdebugEval <code>: evaluate some code and display the result
<Leader>e: evaluate the expression under visual highlight and display the result

主要就是F3步进，F10设断点，F12，查看当前变量值这几个功能了。
</pre>

调试步骤
    * 按F5开始监听，此时需要在五秒钟内用浏览器访问相关页。
    * 访问指定页面，http://localhost.com/?XDEBUG_SESSION_START=1
    * 注意：Url中一定要加上XDEBUG_SESSION_START=1参数开启调试。
