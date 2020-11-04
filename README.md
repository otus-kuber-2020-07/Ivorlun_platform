# Ivorlun_platform
Ivorlun Platform repository

## Homework1

Разворачивал по инструкции https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/

Minikube запускает докер контейнер (ubuntu 2004 wsl2 win 2004), внутри которого разворачивает ещё несколько докер контейнеров с инфраструктурой kubernetes
```
docker ps
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS                                                                                                      NAMES
8b39f76dd716        gcr.io/k8s-minikube/kicbase:v0.0.13   "/usr/local/bin/entr…"   4 hours ago         Up 3 hours          127.0.0.1:32775->22/tcp, 127.0.0.1:32774->2376/tcp, 127.0.0.1:32773->5000/tcp, 127.0.0.1:32772->8443/tcp   minikube
```

с помощью команды minikube ssh можно перейти в хост-контейнер, где увидеть все запущенные контейнеры с подами, в том числе, составляющими основные компоненты архитектуры kubernetes подробности есть в [kubernetes-intro/summary/1hw.md](kubernetes-intro/summary/1hw.md)

Почему, если удалить системные поды , то они восстановятся?

Если для pods выполнить команду describe, то можно будет обнаружить, какая сущность отвечает за контроль над ними:
```
Name:                 coredns-f9fd979d6-kx6jp
Controlled By:  ReplicaSet/coredns-f9fd979d6

Name:                 etcd-minikube
Controlled By:  Node/minikube

Name:                 kube-apiserver-minikube
Controlled By:  Node/minikube
```
Здесь видно, что поды контроллируются самим Node/minikube, но coredns, в отличие от других подов, управляется replicaset-контроллером.  

Поды запускаются из-за статических файлов конфигурации, которые храняться в `/etc/kubernetes/manifests`: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#init-workflow


### Hipster-shop frontend

```
kubectl get -A all
NAMESPACE              NAME                                             READY   STATUS    RESTARTS   AGE
default                pod/frontend                                     0/1     Error     0          46m
default                pod/web                                          1/1     Running   1          83m
kube-system            pod/coredns-f9fd979d6-kx6jp                      1/1     Running   1          2d23h
```

Не запускается из-за того, что приложение требует для запуска переменные среды.  

```
kubectl logs frontend
...
panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
...
```
Которые должны быть определены при запуске.

https://github.com/GoogleCloudPlatform/microservices-demo/blob/5372b01cdcaf66eb9afd8750cc5b2693146637fb/kubernetes-manifests/frontend.yaml#L52

Если добавить их определение в манифест пода [kubernetes-intro/web/frontend-pod-healthy.yaml](kubernetes-intro/web/frontend-pod-healthy.yaml), то ошибка при развёртке исчезнет:
```
kubectl get pod frontend
NAME       READY   STATUS    RESTARTS   AGE
frontend   1/1     Running   0          21s
```

---

