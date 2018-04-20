创建
```go
./kafka-topics.sh --create  --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 1 --topic test
```

查看tipic
```go
# ./kafka-topics.sh --list  --zookeeper 127.0.0.1:2181

./kafka-topics.sh --describe --topic k8s-log  --zookeeper 127.0.0.1:2181

```


测试发送数据
```go
./kafka-console-producer.sh --broker-list 10.39.15.16:9092  --topic test
```

测试接收数据
```go
 ./kafka-console-consumer.sh --zookeeper 127.0.0.1:2181 --topic test
```

列出消费群组
```go
./kafka-consumer-groups.sh  --bootstrap-server 10.39.15.15:9092 --list

./kafka-consumer-groups.sh  --bootstrap-server 10.39.15.13:9092 --describe  --group logstash
```

修改分区数
```go
 bin/kafka-topics.sh --alter --zookeeper 127.0.0.1:2181 --topic k8s-log --partitions 2
```


性能测试
```go
./kafka-producer-perf-test.sh --topic test --num-records 1000000 --record-size 1000 --throughput 20000 --producer-props bootstrap.servers=10.39.15.13:9092
kafka-producer-perf-test.sh 脚本命令的参数为：
--topic topic名称，本例为test_perf
--num-records 总共需要发送的消息数，本例为1000000
--record-size 每个记录的字节数，本例为1000
--throughput 每秒钟发送的记录数，本例为20000
--producer-props bootstrap.servers=localhost:9092 发送端的配置信息，本例只指定了kafka的链接信息
可以看到本例中，每秒平均向kafka写入了16.92 MB的数据，大概是17737条消息，每次写入的平均延迟为962.23毫秒，最大的延迟为7415.00毫秒，831 ms内占50%




./kafka-consumer-perf-test.sh --zookeeper localhost:2181 --topic test --fetch-size 1048576 --messages 1000000 --threads 1

--zookeeper 指定zookeeper的链接信息，本例为localhost:2181 ，如果使用新的纯java客户端则使用另外的配置
--topic 指定topic的名称，本例为test_perf
--fetch-size 指定每次fetch的数据的大小，本例为1048576，也就是1M
--messages 总共要消费的消息个数，本例为1000000，100w
可以看到本例中，总共消费了553.8979M的数据，每秒为56.3936， 总共消费了580804条消息，每秒为59132.9668

```




java -cp KafkaOffsetMonitor-assembly-0.2.1.jar \
     com.quantifind.kafka.offsetapp.OffsetGetterWeb \
     --offsetStorage kafka \
     --zk 10.39.15.13 \
     --port 8080 \
     --refresh 10.seconds \
     --retain 2.days