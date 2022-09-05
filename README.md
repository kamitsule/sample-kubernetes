ローカルのkubeにサンプルを動かしてみる

## 環境

```shell
Docker Desktop 4.12.0
Kubernetes v1.25.0
Docker version 20.10.17, build 100c701
```

## 事前準備

- Docker for Macの Kubernetes をEnableにする
- contextを該当のものにする 今回は `docker-desktop` (Versionによっては別名のときもあるので注意)

## デプロイしてみる

試しにnodeサーバーをあげてみる

- 

### build

```shell
docker build --no-cache -t test-app .
```

### deploy

```shell
$ kubectl apply -f k8s/deployment.yaml
deployment.apps/test-app created

$ kubectl apply -f k8s/service.yaml
service/test-app-service created
```

### deploy OK 結果

```shell
$ kubectl get pod
NAME                        READY   STATUS             RESTARTS   AGE
test-app-8665c79657-87ct2   0/1     ImagePullBackOff   0          48s
test-app-8665c79657-9chb9   0/1     ImagePullBackOff   0          48s
```

### 削除

```shell
$ kubectl delete -f k8s/deployment.yaml
deployment.apps "test-app" deleted
$ kubectl delete -f k8s/service.yaml
service "test-app-service" deleted
```
