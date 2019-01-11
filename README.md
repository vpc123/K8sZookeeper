# K8sZookeeper

## Kubernetes-在Kubernetes集群上搭建Stateful Zookeeper集群


### 1准备工作


前提要求：

1. 至少4个节点的k8s集群
1. 每个节点至少2CPUS 4G 内存
1. 需配置了Dynamic Volume Provisioning特性，即，需配置了storageclass。

#### 1.1 获取官方镜像

    $k8s.gcr.io/kubernetes-zookeeper:1.0-3.4.10

已经在同级目录中放入压缩包。

#### 1.2 解压

    $tar -zxvf K8s-Zk.tar.gz

#### 1.3 导入所需镜像到本地

    $docker  load -i kubernetes-zookeeper_1.0-3.4.10.tar.gz

#### 1.4 Zookeeper集群需要用到存储，这里需要准备持久卷（PersistentVolume，简称PV），我这里以yaml文件创建3个PV，供待会儿3个Zookeeper节点创建出来的持久卷声明（PersistentVolumeClaim，简称PVC）绑定。


注：同样我们可以更换其他存储方式，存储只是一种持久化数据的方式，这个需要根据资源需求自行调整。


    $kubectl create -f persistent-volume.yaml

#### 1.5 查看PV

    $kubectl get pv -o wide

### 部署Zookeeper集群

    $kubectl create -f zookeeper.yaml


###### 标注：创建完后会出现一个问题，就是所有的Zookeeper pod都启动不起来，查看日志发现是用户对文件夹【/var/lib/zookeeper】没有权限引起的，文件夹的权限是root用户。我们需要在每一个运行zookeeper的node上面执行如下命令：

    $chmod -R 777 /var/lib/zookeeper



#### 通过命令查看pod

    $kubectl get pod -o wide

#### 查看PV，发现持久卷声明已经绑定上了。

    $kubectl get pv -o wide
#### 查看PVC

    $kubectl get pvc -o wide

#### 最后来验证Zookeeper集群是否正常，查看集群节点状态

    $for i in 0 1 2; do kubectl exec zk-$i zkServer.sh status; done

一个leader，两个follower，成功！！！


### 部署失败的可能因素：

1  系统资源不足，当选中部署zookeeper的节点因为过高的负载可能造成系统资源不足造成部署应用失败。

2  集群节点个数最好或者必须要奇数个，这样可以正常进行leader选举。

3  由于k8s部署问题，造成的node节点不健康会使得zk集群启动失败，所以需要保证部署zk的k8s集群同官方一致。不然会因为探针测试node节点不健康使得检测失败，zk集群会一直处于重启状态中。

