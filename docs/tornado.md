# tornado框架原理
## tornado.ioloop - Main event loop
典型的tornado应用只用单个IOLoop对象，在main方法里面调用IOLoop.start方法。  
非典型的tornado应用可能使用超过一个IOLoop，一个线程或一个单元测试用例一个IOLoop。  
另外对于I/O事件，ioloop能调度基于时间的事件。IOLoop.add_timeout是time.sleep的非阻塞实现。  
![tornado ioloop](https://engeltt.github.io/images/tornado_1.png)

## tornado处理request请求
post 请求  
框架默认解析form data和urlencode两种形式的post数据  
针对post json格式数据，需要直接获取到body里到内容并解析出来  
