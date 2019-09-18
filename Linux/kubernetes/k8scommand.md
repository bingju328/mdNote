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
kubectl get pod -n spkitty --show-labels #显示Pod的标签
kubectl get rs # 查看目前所有的replica set
kubectl get deployment # 查看目前所有的deployment
kubectl describe po my-nginx # 查看my-nginx pod的详细状态
kubectl describe rs my-nginx # 查看my-nginx replica set的详细状态
kubectl describe deployment my-nginx # 查看my-nginx deployment的详细状态
kubectl delete rs --all #删除所有RS

#查看日志
kubectl logs -f pods/monitoring-influxdb-fc8f8d5cd-dbs7d -n kube-system
kubectl logs --tail 200 -f kube-apiserver -n kube-system |more
kubectl logs kibana-logging-7445dc9757-pvpcv -n kube-system -f
kubectl logs --tail 200 -f podname -n jenkins
#如果一个Pod中有多个容器，需要指定 -c <容器名称>
kubectl logs myapp-pod -c test


kubectl edit deployment nginx
sudo kubectl edit deployment send-service -n spkitty

sudo kubectl cluster-info

sudo docker exec -ti <your-container-name> /bin/bash
sudo kubectl exec -ti <your-pod-name> -n <your-namespace>  /bin/bash
sudo kubectl exec -ti <your-pod-name> -n <your-namespace>  /bin/sh

sudo kubectl top pod --all-namespaces
sudo kubectl top pod -n spkitty
sudo kubectl top node

kubectl -n <namespace> create secret generic jmx-ssl \
  --from-file=java-app.keystore \
  --from-file=java-app.truststore
kubectl get secret
kubectl get secret -n spkitty

在没有pod 的yaml文件时，强制重启某个pod

kubectl get pod PODNAME -n NAMESPACE -o yaml | kubectl replace --force -f -

#获取资源
sudo kubectl get ingress gateway -n spkitty -o yaml
#-w 的意思是 监视这个命令的输出，有新的状态变化立刻输出出来
sudo kubectl get pod -n kube-system -w 

#扩容
kubectl get deployment #得到要扩容的deployment Name
kubectl scale --replicas=3 deployment/nginx-deployment #扩容到3个副本

#暴露一个Service端口 比如niginx-deployment
kubectl get deployment
kubectl expose deployment nginx-deployment --port=30000 --target-port=8000
kubectl get svc -o wide #就会出现了 
curl 10.97.154.59:30000 #就可以访问了 而且是轮询访问的
ipvsadm -Ln | grep 10.97.154.59 #能看到IP

#暴露一个外网IP
kubectl edit svc nginx-deployment
#修改 type: ClusterIP 为  type: NodePort
kubectl get svc #查看会多了一个端口 30000:31859/TCP
netstat -anpt | grep :31859 #会看到有这个端口了
192.168.0.246:31859 192.168.0.247:31859 就都可以访问了

#命令的记录  --record参数可以记录命令，我们可以很方便的查看每次 revision 的变化
kubectl create -f https://kubernetes.io/docs/user-guide/nginx-deployment.yaml --record
#命令的记录 查看（最好还是自己记录一下，这个记录没想象中那么好用）
kubectl create history .......
kubectl rollout history deployment/nginx-deployment

```
