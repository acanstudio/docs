URL重写
=============================

Apache的mod_rewrite模块在Web应用中实现URL重写功能。重定向是根据一组规则，将输入请求中的URL地址，转换为一个完全不同的URL地址，如可以将输入的一个web上不存在的URL：http://seophp.example.com/Products/123.html重定向为web上存在的http://seophp.example.com/product.php?product_id=123。重写是由Web服务器实现的，PHP不用对它作任何处理。重定向的意义就是，可以使复杂的动态网站对搜索引擎是友好的。使用了重定向，访问者和搜索引擎可以利用一目了然的、对搜索引擎友好的静态URL地址访问Web站点，然而在后台，PHP脚本仍将使用参数驱动的方式相应请求。

mod_rewrite的安装和使用
-------------------------

通常会使用Apache的rewrite模块来实现URL的重定向，Apache需要安装mod_rewrite模块后，才能够使用强大的重定向功能。当rewrite是以模块的形式安装的时候，需要在apache的配置文件./conf/httpd.conf中使用命令“LoadModule rewrite_module modules/mod_rewrite.so”将rewrite模块引导到apache服务器中。需要重启apache使模块生效。

