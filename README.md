# ローカルのkubeにサンプルを動かしてみる

## 環境

```shell
Docker Desktop 4.12.0
Kubernetes v1.25.0
Docker version 20.10.17, build 100c701
version.BuildInfo{Version:"v3.8.1", GitCommit:"5cb9af4b1b271d11d7a97a71df3ac337dd94ad37", GitTreeState:"clean", GoVersion:"go1.17.8"}
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

# helm を加えて動かしてみる

チュートリアルのとおりにやってみる
https://helm.sh/docs/intro/quickstart/

## リポジトリにサンプルを追加

```shell
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

## 一覧がずらららと

```shell
$ helm search repo bitnami
```
## WordPress起動してみる

```shell
 $ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```

```shell
$ helm install test-release  bitnami/wordpress
NAME: test-release
LAST DEPLOYED: Mon Sep  5 16:34:00 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: wordpress
CHART VERSION: 15.1.5
APP VERSION: 6.0.2

** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    test-release-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w test-release-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default test-release-wordpress --include "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default test-release-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
```
## デプロイできた

```shell
$ helm list
NAME        	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART           	APP VERSION
test-release	default  	1       	2022-09-05 16:34:00.681849 +0900 JST	deployed	wordpress-15.1.5	6.0.2
$ kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
test-release-mariadb-0                    1/1     Running   0          72s
test-release-wordpress-586d678749-s2wvc   0/1     Running   0          72s
$ kubectl get svc
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes               ClusterIP      10.96.0.1        <none>        443/TCP                      5h20m
test-release-mariadb     ClusterIP      10.110.132.247   <none>        3306/TCP                     115s
test-release-wordpress   LoadBalancer   10.108.214.1     localhost     80:32401/TCP,443:30156/TCP   115s
$ kubectl get configmap
NAME                   DATA   AGE
kube-root-ca.crt       1      5h20m
test-release-mariadb   1      2m3s
$ kubectl get secrets
NAME                                 TYPE                 DATA   AGE
sh.helm.release.v1.test-release.v1   helm.sh/release.v1   1      2m9s
test-release-mariadb                 Opaque               2      2m9s
test-release-wordpress               Opaque               1      2m9s
```

## 削除しとく

```shell
$ helm uninstall --dry-run test-release
release "test-release" uninstalled
$ helm uninstall  test-release
release "test-release" uninstalled
```

```shell
$ helm repo list
NAME   	URL
bitnami	https://charts.bitnami.com/bitnami
$ helm repo remove bitnami
"bitnami" has been removed from your repositories
```

