使用TamperData观察实时的响应头
==============================

响应头是在服务器发送页面的HTML代码之前，从服务器发送到浏览器的。头信息中包含了诸如：通信方式、页面类型、截止日期和内容类型等重要的元数据信息。响应头是攻击者用于查找应用特有信息的地方，Web平台的相关信息很容易通过响应头泄露出去。firebug和WebScarab都可以找到头信息，这里推荐使用TamperData来查看请求头和响应头。响应头和响应本身是不同的，响应头是元数据通常包括状态（Status）、内容类型（Content-Type）、内容编码（Content-Encoding）、内容长度（Content-Length）、截止日期（Expire）、最后修改时间（Last-Modified）。有些响应头会显示服务器软件及相应发出的日期和时间。Content_Type头信息可能引用外部应用或引起异常的浏览器行为，这些异常之处正是攻击可能潜入的地方。