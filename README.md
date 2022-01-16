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
% kubectl create deploy mattermost --image k8spracticalguide/mattermost:4.10.2 -o yaml --dry-run=client > mattermost-deploy.yaml
% kubectl create deploy db --image k8spracticalguide/mysql:5.7.22 -o yaml --dry-run=client > db-deploy.yaml
% kubectl create cm common-env -o yaml --dry-run=client --from-literal MYSQL_USER=myuser --from-literal MYSQL_PASSWORD=mypassword --from-literal MYSQL_DATABASE=mattermost > cm.yaml
```

## kubectl apply -f
