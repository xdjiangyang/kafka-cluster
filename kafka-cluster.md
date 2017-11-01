kafka 依赖于 zookeeper 提供的分布式协作功能，因此部署 kafka 集群之前要先部署 zookeeper 集群。

1. zookeeper 集群(zookeeper-3.4.10)
    
    
	- 配置文件 zoo.cfg
	
			tickTime=2000
			dataDir=/var/lib/zookeeper
			clientPort=2181
			initLimit=5
			syncLimit=2
			server.1=192.168.18.151:2888:3888
			server.2=192.168.18.152:2888:3888
			server.3=192.168.18.153:2888:3888

	- /var/lib/zookeeper/myid 文件
		
			192.168.18.151 机器上此文件的内容为 1，与 server.1 对应

	- 启动一个节点

			bin/zkServer.sh start 

	- 查看一个节点状态

			bin/zkServer.sh status 
	
2. kafka 集群(kafka_2.11-0.11.0.1)
	
	- 修改 server.properties 中的某些内容
	
			broker.id=1 // 与 myid 文件中的数字一致
			listeners=PLAINTEXT://192.168.18.152:9092 // 本机对外的 IP
			zookeeper.connect=192.168.18.153:2181,192.168.18.152:2181,192.168.18.151:2181 // zookeeper 集群的地址和端口

	- 启动一个节点

			bin/kafka-server-start.sh -daemon config/server.properties 

	- 创建一个名为 bbb 的 topic

			bin/kafka-topics.sh --create --zookeeper 192.168.18.153:2181 --replication-factor 2 --partitions 1 --topic bbb 
			# 192.168.18.153 也可以替换为 zookeeper 集群的其他节点

	- 生产数据
	
			bin/kafka-console-producer.sh --broker-list 192.168.18.153:9092 --topic bbb
			# 192.168.18.153 也可以替换为 kafka 集群的其他节点

	- 消费数据

			bin/kafka-console-consumer.sh --bootstrap-server 192.168.18.151:9092 --from-beginning --topic bbb
			# 192.168.18.151 也可以替换为 kafka 集群的其他节点

	kafka 集群配置正确后，producer 可以向任意节点写数据，consumer 可以从任意节点读数据。

	listeners 最好配置为 IP 地址，配置主机名可能会有些异常(按官方文档中的示例进行操作时可能无法正常生产/消费)。