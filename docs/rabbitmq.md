一个发送消息的程序就是一个生产者：  
![生产者](https://engeltt.github.io/images/1.png)

一个队列就是rebbitmq中的邮箱。消息只能存储在队列中，队列受到服务器的内存和硬盘大小的限制，是一个巨大的消息缓冲区。许多生产者可以发送消息给同一个队列，许多消费者
可以从这同一个队列中接收消息。我们这样表示一个队列：  
![队列](https://engeltt.github.io/images/2.png)
