## 1.virtual hosts
当rabbitmq给单一项目提供服务时：使用默认的虚拟主机virtual host(/)就可以了。  

给不同项目提供服务时：每个项目配置一个虚拟主机，例如：  
project1_development, project1_production, project2_development, project2_production

## 2.users
生产环境删除默认用户guest，guest只能从localhost连接（密码guest。。。认证太简单）。不需考虑使远程连接可用，只需要创建一个有管理员权限的用户并设置密码。
推荐每个应用使用一个单独的用户。例如，移动端app，web app及数据聚合系统，三个系统用三个用户。这确保一些事情更简单化：

* 应用程序的连接相互影响
* 使用细粒度的权限控制
* 滚动认证（例如周期性或者在认证失败情况下）  

在一个应用有很多个实例的情况下，在更好的安全性和更方便的配置之间需要做权衡。

## 3.monitoring和resource limits
rabbitmq的节点收到许多资源的限制，物理的（ram大小）和软件的（一个进程可以开启的文件处理器数量）都有。在投入生产环境之前评估资源配置，之后持续监控资源消耗是重要的。

## 4.monitoring
监控系统的各个方面：基础设施，内核指标。当监控需要耗费时间资源时，捕获问题和提早注意潜在问题的趋势更有效。

## 5.memory
rabbitmq使用资源驱动警报来限制publisher，当consumer跟不上时。默认的，当检测到使用超过40%的内存时（vm_memory_high_watermark），rabbitmq不再接收任何新消息。这是一个安全的默认值，修改需谨慎，即使是专门用途的rabbitmq节点。
os和文件系统使用内存来加速系统进程的处理。如果没有足够的可用内存，将对系统性能起到不利影响（由于os swapping），并且可能会导致rabbitmq进程终止。

若要调整vm_memory_high_watermark的值，有一些建议：

* rabbitmq节点应一直保证有至少128M可用内存
* vm_memory_high_watermark的推荐值范围为0.4到0.66
* 0.7以上不推荐。os和文件系统必须留至少30%的内存空间，否则性能将严重下降（由于paging分页）

## 6.磁盘空间 disk space
开发环境50M可用磁盘空间即可。生产环境要求更大的可用磁盘空间，以免节点失败和数据丢失（写入磁盘失败）。  

生产环境推荐值：

* {disk_free_limit, {men_relative, 1.0}} 专用rabbitmq节点，4G内存，可用磁盘空间不应小于4G，此为最低推荐值。
* {disk_free_limit, {men_relative, 1.5}} 4G内存，可用磁盘空间不应小于6G，此为安全推荐值。
* {disk_free_limit, {men_relative, 2.0}} 此为最保守生产环境推荐值

## 7.打开 file handler limit
操作系统限制最大并发文件处理器数（包含网络socket）。确保设置了并发连接和队列的限制。  

确保对有效的rabbitmq用户设置了至少50k文件描述符。  

根据经验，将95%的并发连接数乘以2，然后加上队列总数，以计算建议的打开文件句柄限制。生产环境推荐500k，够用了也不会消耗许多硬件资源。

## 8.日志收集 log collection
强烈推荐收集所有rabbitmq节点和应用程序的日志。

## 9.应用程序层面的考虑
应用程序设计和使用rabbitmq的方式将影响系统的稳定性。应用程序没有有效地使用资源或者资源泄漏将显著影响系统运转。例如，一个app持续的打开连接却从不关闭连接，将导致节点的文件描述符耗尽，以至于无法接受新的连接。

因素很复杂，但可以通过监控来及早发现问题（例如：频道泄漏，文件描述符使用持续增长（连接池管理））

## 10.连接管理
消息协议通常假定为长连接。一些应用在启动时连接rabbitmq，并且只在终止时关闭连接。其他的打开和关闭连接更动态。对后者来说，当连接不需要再使用时关闭它们很重要。

连接可以被应用程序以外的原因关闭。rabbitmq支持的消息协议可使用heartbeats的特性。开发者小心不要让心跳timeout值太低（低于5s），因为可能在网络拥堵或者系统负载上升的时候产生错误反应。  

非常短的活跃连接应该尽量避免。

## 11.连接丢失 connection churn
上面提到，消息协议通常假定长连接。一些应用可能会打开一个新的连接来执行单个操作（例如：发布一条消息），然后关闭连接。这是极不推荐的，因为打开连接是昂贵的操作（和复用已存在的连接相比较）。这样的工作负载也容易造成连接丢失。连接丢失率高将导致tcp连接更快释放，否则将导致文件处理器或内存耗尽，导致停止接受新的连接。
如果不能减少长连接的数量，那么连接池可以帮助资源使用率削峰。

## 12.连接失败的恢复
一些客户端库，例如java，.net， ruby，支持网络失败后的自动重连。如果客户端库支持这个特性，建议使用它而不是自己开发重连机制。  

其他的库（go, pika）不支持自动重连，但提供示例来说明如何从连接失败中恢复连接。

## 13.过度使用channel
channels也会消耗客户端和服务器的资源。应用程序应尽可能减少使用频道的数量，并且在不再使用时关闭频道。频道和连接一样，被认为是持久的。  

注意：关闭连接将自动关闭此连接的所有频道。

## 14.轮询消费者
开发者应避免使用轮询消费者（使用basic.get来消费），因为大多数情况下轮询是低效的。

## 15.安全方面的考虑
### Users and Permissions
查看上面关于虚拟主机，用户和认证的部分。
### Inter-node 和 cli 工具认证
rabbitmq节点之间的认证使用存储在文件中的共享密钥。在linux和其他类unix系统，限制只有能运行rabbitmq和cli工具的os用户能访问cookie文件是必要的。

此密钥应用相当安全的方式来生成。通常在部署初始化时由自动化部署工具来完成。这些工具可使用默认值或占位值，不要依赖它们。

允许运行时在一个节点生成cookie文件并拷贝到所有其他节点也并非好的实践，因为生成算法都知道，容易预测。

### 防火墙配置
### 端口
### tls 传输层安全协议

## 16.网络配置
用户访问高并发要求调整网络配置

## 17.集群
集群大小
* 期望吞吐量
* 期望复制（镜像的数量）
* 数据位置

分区处理策略

节点时间同步
