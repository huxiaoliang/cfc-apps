Instructions of running Cassandra cluster with persistent storage in CfC

Contact liguangc@cn.ibm.com for any comments or questions

# Create the persistent volume

   ## NFS persistent volume

    ### Configure your NFS server

        Ubuntu example:

```
            apt-get install -y nfs-kernel-server
            service nfs-kernel-server start
            mkdir /k8spv
            mkdir /k8spv/cassandra0 /k8spv/cassandra1 /k8spv/cassandra2
            Add the line "/k8spv *(rw,no_root_squash,sync,no_subtree_check)" into /etc/exports
            exportfs -a
```

    ### Update the pv-nfs.yaml to replace the ip address of the nfs server

    ### Create the persistent volume

```
            kubectl create -f pv-nfs.yaml
```

   ## TODO - the other persistent volume types

# Create the Cassandra petset

```
kubectl create -f cassandra-petset.yaml
```

After the Cassandra petset is up and running, you will see something similar to:

```
root@c910f04x19k02:~# kubectl get pvc
NAME                         STATUS    VOLUME          CAPACITY   ACCESSMODES   AGE
cassandra-data-cassandra-0   Bound     cassandra-pv1   1Gi        RWO           47m
cassandra-data-cassandra-1   Bound     cassandra-pv2   1Gi        RWO           47m
cassandra-data-cassandra-2   Bound     cassandra-pv3   1Gi        RWO           47m
root@c910f04x19k02:~# 
root@c910f04x19k02:~# kubectl get petset
NAME        DESIRED   CURRENT   AGE
cassandra   3         3         33m
root@c910f04x19k02:~# 
root@c910f04x19k02:~# kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
cassandra-0   1/1       Running   0          34m
cassandra-1   1/1       Running   0          33m
cassandra-2   1/1       Running   0          31m
root@c910f04x19k02:~# 
```

# Verify Cassandra is working correctly

Run the following commands, replace the POD name with your own one, you will get something similar to:

```
root@c910f04x19k02:~# kubectl exec cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address      Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.16.76.2  82.68 KiB  32           67.4%             5926c238-6a44-48d7-ad85-0cfbecc7811f  Rack1-K8Demo
UN  172.16.87.3  149.36 KiB  32           75.3%             6fb6e066-64fa-4552-b05a-b37c67231ff3  Rack1-K8Demo
UN  172.16.77.5  65.25 KiB  32           57.3%             144a6535-c45b-41e4-bf92-dd937e824573  Rack1-K8Demo

root@c910f04x19k02:~# kubectl exec cassandra-0 -- cqlsh -e 'desc keyspaces'

system_traces  system_schema  system_auth  system  system_distributed

root@c910f04x19k02:~# 
```


# Then from spark or any other service, you could access the Cassandra through any of the following endpoints

```
 cassandra:9042
 cassandra-0:9042
 cassandra-1:9042
 cassandra-2:9042
```
