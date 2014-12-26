动态跟踪元素属性
==============================

元素属性可以被样式表或js动态改变，测试由js事件驱动的动态Web应用，是测试的一个重要组成部分。下面的一段测试代码尝试跟踪元素动态变更的信息。

::

    var test_element = document.getElementById('your_id');

    function display_attribute()
    {
        alert("New Element! \n ID:" + test_element.lastChild.id + "\n HTML:" + test_element.lastChild.innerHTML);
    }

    test_element.addEventListener("DOMNodeInserted", display_attribute, 'false');

    new_element = document.getElementById('some_other_element');
    test_element.appendChild(new_element);