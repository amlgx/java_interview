## kafka性能高的原因是什么？

磁盘顺序写。

充分利用操作系统缓存。

消息从内核直接发送。

分区写入，多个消费者同时消费。

## kafka如何保证消息的安全？

通过分区副本来保证。将acks设置为all，可以等到消息分布到副本集上再返回。

kafka 可以脱离 zookeeper 单独使用吗？为什么？

kafka 有几种数据保留的策略？

kafka 同时设置了 7 天和 10G 清除数据，到第五天的时候消息达到了 10G，这个时候 kafka 将如何处理？

什么情况会导致 kafka 运行变慢？

使用 kafka 集群需要注意什么？