kubectl

```shell
#获取节点
sudo kubectl get nodes
#查看某个节点的明细
sudo kubectl describe node worker-node9
#查看运行状态
sudo kubectl get apiservice
sudo kubectl get apiservice v1beta1.metrics.k8s.io -o yaml

sudo kubectl delete -f metrics-server-deployment.yaml
sudo kubectl create -f metrics-server-deployment.yaml
sudo kubectl get pods -n kube-system |grep metrics-server

sudo kubectl get namespaces
sudo kubectl get svc -n kube-system|grep metrics-server

kubectl get po # 查看目前所有的pod
kubectl get rs # 查看目前所有的replica set
kubectl get deployment # 查看目前所有的deployment
kubectl describe po my-nginx # 查看my-nginx pod的详细状态
kubectl describe rs my-nginx # 查看my-nginx replica set的详细状态
kubectl describe deployment my-nginx # 查看my-nginx deployment的详细状态

#查看日志
kubectl logs -f pods/monitoring-influxdb-fc8f8d5cd-dbs7d -n kube-system
kubectl logs --tail 200 -f kube-apiserver -n kube-system |more
kubectl logs kibana-logging-7445dc9757-pvpcv -n kube-system -f
kubectl logs --tail 200 -f podname -n jenkins


kubectl edit deployment nginx
sudo kubectl edit deployment send-service -n spkitty

sudo kubectl cluster-info

sudo docker exec -ti <your-container-name> /bin/#!/bin/sh
sudo kubectl exec -ti <your-pod-name> -n <your-namespace>  /bin/bash

```
