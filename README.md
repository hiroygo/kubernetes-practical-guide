# kubernetes-practical-guide
* https://github.com/kubernetes-practical-guide/examples

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

## マニフェストの雛形作成
```sh
% kubectl create deploy nginx --image k8spracticalguide/mattermost:4.10.2 -o yaml --dry-run=client > mattermost-deploy.yaml
% kubectl create deploy nginx --image k8spracticalguide/mysql:5.7.22 -o yaml --dry-run=client > db-deploy.yaml
```

## kubectl apply -f
