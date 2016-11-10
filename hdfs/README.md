Instructions of running HDFS cluster with persistent storage in CfC

Contact liguangc@cn.ibm.com for any comments or questions

# Download the hdfs yaml files onto your CfC master node

# Create the persistent volume

## NFS persistent volume

### Configure your NFS server

Ubuntu example:

```
            apt-get install -y nfs-kernel-server

            service nfs-kernel-server start

            mkdir /k8spv

            mkdir /k8spv/hdfs-namenode

            Add the line "/k8spv *(rw,no_root_squash,sync,no_subtree_check)" into /etc/exports

            exportfs -a
```

### Update the pv-nfs.yaml to replace the ip address of the nfs server

### Create the persistent volume

kubectl create -f pv-nfs.yaml

## TODO - the other persistent volume types

# Create the HDFS namenode service and deployment

```
kubectl create -f hdfs-namenode-svc.yaml

kubectl create -f hdfs-namenode-dp.yaml
```

Wait for several minutes, make sure the namenode is up and running

# Crete the HDFS data node deployment or DaemonSet

Depends on the use case, you could select to use DaemonSet or Deployment to run the HDFS data nodes containers:

kubectl create -f hdfs-datanode-ds.yaml

or

kubectl create -f hdfs-datanode-dp.yaml

TODO: the datanode containers are using hostPath as the persistent volume, with the deployment, need to use the anti-affinity to make sure there is only one datanode run on each worker node.

# Verify HDFS is working correctly:

```
root@c910f04x19k07:~/yaml/hdfs# kubectl exec -it hdfs-namenode-577116430-up5nw -- /bin/bash
root@hdfs-namenode-577116430-up5nw:/# hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - root supergroup          0 2016-10-28 05:13 /testdir
root@hdfs-namenode-577116430-up5nw:/# hdfs dfs -mkdir /tmp
root@hdfs-namenode-577116430-up5nw:/# hdfs dfs -copyFromLocal /var/log/dpkg.log /tmp/                  
root@hdfs-namenode-577116430-up5nw:/# hdfs dfs -ls /tmp/  
Found 1 items
-rw-r--r--   3 root supergroup     145296 2016-10-28 06:26 /tmp/dpkg.log
root@hdfs-namenode-577116430-up5nw:/# 
```

# Access the HDFS service

From spark or any other service, you could access the HDFS through the endpoint: 

hdfs://hdfs-namenode:8020
