PHP调试和性能优化
===============================

php性能优化分析工具XDebug 大型网站调试工具
发布：dxy 字体：[增加 减小] 类型：转载
大型网站调试工具之一（php性能优化分析工具XDebug） ,开发php的朋友可以参考下。有助于解决php代码的多种问题。
一、安装配置
　　1、下载PHP的XDebug扩展，网址：http://xdebug.org/

　　2、在Linux下编译安装XDebug

引用
tar -xzf xdebug-2.0.0RC3.gz
cd xdebug-2.0.0RC3
/usr/local/php/bin/phpize
./configure --enable-xdebug
cp modules/xdebug.so /usr/local/php/lib/php/extensions/no-debug-non-zts-20020429/

　　注：/usr/local/php/lib/php/extensions/no-debug-non-zts-20020429/不同的PHP版本路径不同，也不一定要放在该路径，可以在zend_extension_ts中自行指定xdebug.so所在位置。

引用
vi /usr/local/php/lib/php.ini

　　修改php.ini，去除PHP加速模块，增加以下配置信息支持XDebug扩展
复制代码 代码如下:

[Xdebug]
zend_extension_ts="/usr/local/php/lib/php/extensions/no-debug-non-zts-20020429/xdebug.so"
xdebug.profiler_enable=on
xdebug.trace_output_dir="/tmp/xdebug"
xdebug.profiler_output_dir="/tmp/xdebug"
xdebug.profiler_output_name="script"

引用
mkdir -p /tmp/xdebug
chmod 755 /tmp/xdebug
chown www:www /tmp/xdebug
/usr/local/apache/bin/apachectl -k restart


　　3、客户端（Windows）：WinCacheGrind
　　下载地址：http://sourceforge.net/projects/wincachegrind/

　　二、分析过程
　　1、访问你的网站，将首页上各种链接点击几遍，XDebug在/tmp/xdebug目录生成以下文件：
　　usr_local_apache_htdocs_app_checknum_chknum_php_cachegrind.out
　　usr_local_apache_htdocs_app_login_showHeaderLogin_php_cachegrind.out
　　usr_local_apache_htdocs_app_play_play_php_cachegrind.out
　　usr_local_apache_htdocs_app_user_member_php_cachegrind.out
　　usr_local_apache_htdocs_tag_tags_php_cachegrind.out
　　usr_local_apache_htdocs_top_top_php_cachegrind.out

　　2、将以上文件拷贝到Windows上，用客户端软件WinCacheGrind打开每个文件，发现以下PHP程序执行所耗费的时间最长：
　　/usr/local/apache/htdocs/tag/tags.php　　　　　　耗时840ms

　　三、分析结果：
　　1、/usr/local/apache/htdocs/tag/tags.php



　　(1)耗时最长的filter_tags函数出现在/usr/local/apache/htdocs/tag/tags.php的第158行：
　　$tags .= filter_tags($videos[$i]['tags'])." ";

　 　(2)filter_tags函数引自/usr/local/apache/htdocs/include /misc.php，getForbiddenTags函数被filter_tags函数调用了21次，filter_tags函数耗费的时间中绝大多数 因getForbiddenTags函数所致。getForbiddenTags函数的内容如下：
复制代码 代码如下:

function getForbiddenTags()
{

$tagsPath=TEMPLATE_FILE_PATH."tags/forbidden_tags.txt";
if(file_exists($tagsPath))
{
$fp = fopen($tagsPath, "r");
$arrconf = array ();
if ($fp)
{
while (!feof($fp))
{
$line = fgets($fp, 1024);
$line = trim($line);
$rows = explode("#", $line);
$coumns = explode("=", trim($rows[0]));
if(""!=trim($coumns[0]))
{
$arrconf[trim($coumns[0])] = trim($coumns[1]);
}
}
}
return $arrconf;
}
}

(4)对getForbiddenTags函数进行分析，其中的PHP函数trim被调用了16827次。
　　

　　(5)可能造成瓶颈的原因：
　　要过滤的156个关键字逐行存放在/usr/local/apache/template/tags/forbidden_tags.txt文件中，文本数据库的效率不高。
　　逐行读取函数fgets、以及去除字符串两边的空白或者指定的字符的函数trim在高负载下的效率低，可以测试fopen、fread、fscanf之类的文件读取函数，对比一下。