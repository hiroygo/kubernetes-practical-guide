# kubernetes-practical-guide
* https://github.com/kubernetes-practical-guide/examples

## kubectl get
* `-l` でラベル指定できる
```sh
$ kubectl get po -l app=db
```

## kubectl create
```sh
% kubectl create deployment nginx --image nginx
deployment.apps/nginx created
% kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           11s
% kubectl expose --type NodePort --port 80 deployment nginx
service/nginx exposed
% minikube service nginx
```
## kubectl run(ワンショットな pod 起動)
```sh
% kubectl run pingtest -i --rm --image k8spracticalguide/busybox:1.28 --restart=Never -- ping -c 1 172.17.0.6
PING 172.17.0.6 (172.17.0.6): 56 data bytes
64 bytes from 172.17.0.6: seq=0 ttl=64 time=0.119 ms

--- 172.17.0.6 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.119/0.119/0.119 ms
pod "pingtest" deleted
```

## kubectl apply -f
```sh
% kubectl apply -f mattermost-deploy.yaml
```

## --dry-run
### deployment, confingmap, secret
```sh
% kubectl create deploy mattermost --image k8spracticalguide/mattermost:4.10.2 -o yaml --dry-run=client > mattermost-deploy.yaml
% kubectl create deploy db --image k8spracticalguide/mysql:5.7.22 -o yaml --dry-run=client > db-deploy.yaml
% kubectl create cm common-env -o yaml --dry-run=client --from-literal MYSQL_USER=myuser --from-literal MYSQL_PASSWORD=mypassword --from-literal MYSQL_DATABASE=mattermost > cm.yaml
% kubectl create secret generic common-env -o yaml --dry-run=client --from-literal MYSQL_ROOT_PASSWORD=rootpassword --from-literal MYSQL_PASSWORD=mypassword > secret.yaml
```

### clusterip, externalname(クラスタ内での名前解決)
* clusterip はクラスタ内サービスの名前解決をする。spec.clusterIP を None にすると ClusterIP が付与されず pod の IP が直接返る(Headless Service)
```sh
% kubectl create svc clusterip mattermost-db --tcp 3306 -o yaml --dry-run=client > db-service.yaml
```

* externalname はクラスタ内からクラスタ外のサービスの名前解決をする。ExternalName の CLUSTER-IP は付与されない
```sh
% kubectl create svc externalname ext-mattermost-db --external-name www.google.com
% kubectl get svc
NAME                TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)    AGE
ext-mattermost-db   ExternalName   <none>       www.google.com   <none>     98s
kubernetes          ClusterIP      10.96.0.1    <none>           443/TCP    30d
mattermost-db       ClusterIP      10.98.57.1   <none>           3306/TCP   25m
```

### nodeport, ingress, loadbalancer(アプリケーションを外部に公開する)
* nodeport は CLUSTER-IP が使われているので、クラスタ内から名前解決できる
```sh
# expose だと mattermost という deployment が実際に存在しないとエラーになる
% kubectl expose --type NodePort --port 8065 deploy mattermost
% kubectl get svc mattermost -o wide
NAME         TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
mattermost   NodePort   10.102.79.83   <none>        8065:32427/TCP   74s   app=mattermost
% kubectl run nsloolup -i --rm --image busybox --restart=Never -- nslookup mattermost
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   mattermost.default.svc.cluster.local
Address: 10.102.79.83
% minikube ip
192.168.49.2
# https://minikube.sigs.k8s.io/docs/handbook/host-access/
% minikube ssh
docker@minikube:~$ curl 192.168.49.2:32427 > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3261  100  3261    0     0  3184k      0 --:--:-- --:--:-- --:--:-- 3184k
# create だと mattermost という deployment が実際に存在しなくても OK
% kubectl create service nodeport test --tcp 8080 --node-port 12345 --dry-run=client
```

* ingress
```sh
% kubectl create ingress mattermost --rule=chat.foo.nip.io/=mattermost:8065 --dry-run=client -o=yaml
% minikube ssh
docker@minikube:~$ curl --resolve chat.foo.nip.io:80:127.0.0.1 chat.foo.nip.io > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3261  100  3261    0     0  1592k      0 --:--:-- --:--:-- --:--:-- 1592k
```

* loadbalancer
```sh
% kubectl expose deployment mattermost --port 8065 --type=LoadBalancer --name=lb
% kubectl get svc
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP          32d
lb              LoadBalancer   10.106.202.206   <pending>     8065:31786/TCP   10s
mattermost      NodePort       10.102.79.83     <none>        8065:32427/TCP   2d1h
mattermost-db   ClusterIP      10.98.57.1       <none>        3306/TCP         2d2h
# 以下は別のターミナルで実行する
 % minikube tunnel
🏃  Starting tunnel for service lb.
% kubectl get svc
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP          32d
lb              LoadBalancer   10.106.202.206   127.0.0.1     8065:31786/TCP   2m14s
mattermost      NodePort       10.102.79.83     <none>        8065:32427/TCP   2d1h
mattermost-db   ClusterIP      10.98.57.1       <none>        3306/TCP         2d2h
% curl 127.0.0.1:8065 > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3261  100  3261    0     0   297k      0 --:--:-- --:--:-- --:--:-- 1061k
```

