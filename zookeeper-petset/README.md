Instructions of running zookeeper cluster with persistent volume in CfC


# Create the persistent volumes for zookeeper containers

## NFS persistent volume

### Configure your NFS server

Ubuntu example:
```
apt-get install -y nfs-kernel-server
service nfs-kernel-server start
mkdir /k8spv
mkdir /k8spv/zookeeper-0 /k8spv/zookeeper-1 /k8spv/zookeeper-2
Add the line "/k8spv *(rw,no_root_squash,sync,no_subtree_check)" into /etc/exports
exportfs -a
```

### Update the pv-nfs.yaml 

Replace the ip address of the nfs server

### Create the persistent volume

```
kubectl create -f pv-nfs.yaml
```

## TODO - the other persistent volume types

# Create the zookeeper cluster

```
kubectl create -f zookeeper-svc.yaml
kubectl create -f zookeeper-petset.yaml
```

# Verify zookeeper cluster is working correctly

```
kubectl run --attach bbox --image=busybox --restart=Never -- sh -c 'while true; do for i in 0 1 2; do echo zoo-$i $(echo stats | nc zoo-$i.zk.default.svc.cluster.local 2181 | grep Mode); sleep 1; done; done';
```

will get something like:

```
zoo-0 Mode: follower
zoo-1 Mode: follower
zoo-2 Mode: leader
```

Run the following commands to verify the key-value pair could be created successfully:

```
kubectl exec zoo-0 -- zkCli.sh create /foo bar
kubectl exec zoo-0 -- zkCli.sh get /foo
kubectl exec zoo-1 -- zkCli.sh get /foo
kubectl exec zoo-2 -- zkCli.sh get /foo
```

# Access the zookeeper service

From any that needs zookeeper service, could access zookeeper service through any of the following endpoint:

```
zk:2181
zoo-0.zk.default.svc.cluster.local:2181
zoo-1.zk.default.svc.cluster.local:2181
zoo-2.zk.default.svc.cluster.local:2181
```
