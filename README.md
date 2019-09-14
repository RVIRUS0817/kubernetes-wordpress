# kubernetes-wordpress

![kubernetes-do](https://user-images.githubusercontent.com/5633085/62842749-4279e080-bcef-11e9-802f-6f2b1482458f.png)

## Environment
- DigitalOcean
- k8s 1.15.3-do.2

![スクリーンショット 2019-09-14 19 12 26](https://user-images.githubusercontent.com/5633085/64906615-9ed38400-d723-11e9-91d9-b106852dcce1.jpg)


```
Some config data            (k8s ConfigMaps and Secrets)

MySQL Container             (k8s replicaset)
MySQL Service               (k8s service)
    |
    |
WordPress Container         (k8s deployment)
[ apache and php-fpm ]
    |
    |
DO Loadbalancer             (k8s service)
```

## Preparation

- install command

```
$ brew install kubernetes-cli doctl
```

- download config.yaml

```
$ cp -r ~/Desktop/adachin-wordpress-cluster-kubeconfig.yaml ~/.kube
$ kubectl get services
```

## MySQL Setup

- add root pass
```
$ echo -n "x97-XinfGkMg" | base64
eDk3LVhpbmZHa01n%
$ vim wordpress/secrets/secrets.yaml
MYSQL_ROOT_PASSWORD: eDk3LVhpbmZHa01n%
```

- apply

```
$ cd wordprees
$ kubectl apply -f secrets/secrets.yaml
$ kubectl apply -f manifests/mysql-volume-claim.yaml
$ kubectl apply -f manifests/mysql-replicaset.yaml
$ kubectl apply -f manifests/mysql-service.yaml

```

- make db

```
$ kubectl get pods
$ kubectl exec -it mysql-abcde -- bash 
$ mysql -u root -p

> CREATE DATABASE wordpress;
```

- check out k8s
```
$ kubectl get pv
$ kubectl get secrets
$ kubectl get replicasets
$ kubectl get pods
$ kubectl describe pod $YOURPOD
$ kubectl logs $YOURPOD
```

## Wordpress Setup

```
$ kubectl apply -f manifests/wordpress-datavolume-claim.yaml
$ kubectl apply -f manifests/wordpress-deployment.yaml
```

## Load Balancer Setup

https://www.digitalocean.com/docs/kubernetes/how-to/configure-load-balancers/  

- certificate check

```
$ curl -X GET -H "Content-Type: application/json" -H "Authorization: Bearer api-key" "https://api.digitalocean.com/v2/certificates"
```

- add certificate-ids

```
$ vim wordpress/manifests/DO-loadbalancer.yaml
service.beta.kubernetes.io/do-loadbalancer-certificate-id: "xxxxxxxxxxxxxx"
```

```
$ kubectl apply -f manifests/DO-loadbalancer.yaml
```

- work!
```
$ kubectl get services
```
