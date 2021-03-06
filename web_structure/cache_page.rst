﻿动态内容缓存
================

通过改进Web服务器的I/O模型和并发策略提升Web服务器性能。当我们关注到Web服务器之后，会发现，网站内容的生成过程涉及到了更多的CPU计算和I/O操作，尤其是访问数据库服务器的时候，这个过程存在很大的优化空间，其中之一就是通过缓存动态内容来提升Web性能。

重复的开销
-----------

动态内容在生成过程中耗费了很多的系统资源，花费了较长的时间，而且这些属于重复的开销，因为每次访问都会重新生成一次。

缓存与速度
-----------

这里的页面缓存只是Web缓存优化中的一部分，其他的还有代码解释器缓存、Web服务器缓存等。

缓存就是把需要耗费较多的服务器资源和时间的计算结果保存起来，在以后需要的时候直接使用这些结果，避免重复计算。使用缓存时要留意缓存的命中率问题，命中率低的缓存不但起不到优化效果，反而增加额外的缓存开销。

页面缓存
---------

页面缓存就是将动态生成的HTML页面内容进行缓存。缓存方法有很多，选择自己熟悉的即可。如可以使用Smarty模板提供的页面缓存机制，也可以选择ZendFramework等框架的页面缓存机制，甚至可以实现自己的页面缓存机制。

将动态内容的结果缓存在磁盘上，即是页面缓存的持久化。可以通过对缓存目录分级解决缓存文件过多的问题。

缓存的过期问题，由于动态内容不断更新，所有有些缓存里的内容可能是过期的，所有使用缓存时要有有效的过期检查机制。

还可以通过将缓存的内容放在内存里来避免磁盘I/O的开销。如PHP的APC或XCACHE模块可以实现将缓存内容存在内存里。

对更加复杂的缓存可以使用缓存服务器来实现，就是将缓存内容存储在其他的服务器上，如存储到memcached的缓存系统里。需要注意的是使用缓存服务器存在TCP socket开销。

局部无缓存
-----------

有些网站的某一块内容不应该使用缓存，即局部无缓存，如页面上的即时公告，页面上的用户行为信息等。局部无缓存可以使用多种手段来实现，如通过Ajax来实现等

静态化内容
------------

以上谈到的缓存都是通过动态代码来实现的，整个过程中，还是存在代码执行的开销的。可以把结果直接生产完整的HTML静态页面，即静态化内容。使用静态化要有合理的更新策略。

局部静态化，可以结合Web服务端的SSI技术实现局部静态化。

总之，动态内容缓存的方式有多种，应根据具体需求使用最合适的缓存方法。
