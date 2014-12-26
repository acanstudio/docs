检测JavaScript事件
==============================

测试Web安全的一个基本原则是：绕开应用期望在你的浏览器中运行的JavaScript。开发人员通常会在页面的<body>、<form>、<select>、<checkbox>、<img>等标签上使用onClick、onMouseOver、onFocus、onBlur、onLoad、onSubmit等js事件，作为测试人员，需要知道这些js事件及相关的js代码的存在，并查找绕过这种检查的方法，以确保这些js事件不会影响到服务器上的应用。