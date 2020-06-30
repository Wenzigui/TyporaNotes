1.  从官网下载es压缩包，解压

![image-20200621190338933](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200621190338933.png)

![image-20200621190427983](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200621190427983.png)

2.  修改elasticsearch.yml配置文件（集群部署的方式）

```shell
# ------- 节点1的配置 -------
cluster.name: es-cluster
node.name: node-1
network.host: 0.0.0.0
discovery.seed_hosts: ["192.168.66.105", "192.168.66.106", "192.168.66.107"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
# ------- 节点2的配置 -------
cluster.name: es-cluster
node.name: node-2
network.host: 0.0.0.0
discovery.seed_hosts: ["192.168.66.105", "192.168.66.106", "192.168.66.107"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
# ------- 节点3的配置 -------
cluster.name: es-cluster
node.name: node-3
network.host: 0.0.0.0
discovery.seed_hosts: ["192.168.66.105", "192.168.66.106", "192.168.66.107"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

3.  启动es

```shell
# 前台启动
./bin/elasticsearch
# 后台启动
./bin/elasticsearch -d
```

4.  遇到的坑
    -   配置不适用于生产环境
        https://juejin.im/post/5cb81bf4e51d4578c35e727d
    -   max virtual memory areas too low
        https://blog.csdn.net/lilongsy/article/details/78708027
    -   max file descriptors too low
        [https://medium.com/@d101201007/elk%E6%95%99%E5%AD%B8-%E8%A7%A3%E6%B1%BA-max-file-descriptors-4096-for-elasticsearch-process-is-too-low-increase-to-at-least-6b9d97791ec9](https://medium.com/@d101201007/elk教學-解決-max-file-descriptors-4096-for-elasticsearch-process-is-too-low-increase-to-at-least-6b9d97791ec9)
    -   不能使用root启动
        https://www.cnblogs.com/gcgc/p/10297563.html

