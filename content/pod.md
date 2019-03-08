**Pod**

Pod - коллекция контейнеров, использующие одно пространство имен (namespace), единое сетевое адресное пространство и имеют доступ к общему локальному хранилищу узла (node). Все контейнеры в pod размещаются на одном узле (node).

Чтобы создать и запустить pod использую docker образ `melhiades/simpleapi:latest` необходимо выполнить следующую команду:

```bash
kubectl run --generator=run-pod/v1 apiserver --image=melhiades/simpleapi:latest
```

Убедимся, что pod заработал:

```bash
> $ kubectl get pods                                                                                                                
NAME        READY   STATUS    RESTARTS   AGE
apiserver   1/1     Running   0          4s

> $ kubectl describe pod apiserver | grep IP:                                                                                       
IP:                 172.17.0.6
```

Внутри кластера этот pod доступен по адресу `172.17.0.6`. Мы можем подключиться по ssh к узлу и проверить работу сервера (для minikube воспользуйтесь командой `minikube ssh`). 

```bash
$ curl http://172.17.0.6:5000/info
"Hello, Kubernetes!!!, version: 1.0"
```

Чтобы удалить pod необходимо выполнить ```kubectl delete pods apiserver```

**Использование файла конфигурации**

Pod так же можно создавать используя файлы конфигурации. В этом примере pod содержит два контейнера simpleapi и alpine-curl. [конфигурация](https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/pod/pod.yaml)

```bash
> $ kubectl create -f https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/pod/pod.yaml
pod/twocontainers created

> $ kubectl get pods                                                                                                
NAME            READY   STATUS    RESTARTS   AGE
twocontainers   2/2     Running   0          7s
```

Теперь можно подключиться к контейнеру client и получить доступ к контейнеру apiserver.

```bash
> $ kubectl exec twocontainers -c client -ti sh                                                                         
/ # curl http://localhost:5000/info
"Hello, Kubernetes!!!, version: 1.0"
```

Что-бы указать ограничения по использованию CPU и памяти (memory) pod'ом, можно воспользоваться директивой `resources`. (в данной конфигурации лимит `memory: 128Mi/cpu: 0.5` запрошенные ресурсы `memory: 64Mi/cpu: 0.25`)  [конфигурация](https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/pod/limits-pod.yaml)

```bash
> $ kubectl apply -f https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/pod/limits-pod.yaml
> $ kubectl describe pod limits-pod  
...
Containers:
  apiserver:
...
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        250m
      memory:     64Mi
...
```

Узнать больше информации об ограничении ресурсов можно из документации [здесь](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/) и [здесь](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/).

Что удалить все созданные pod нужно выполнить команды:

```bash
> $ kubectl delete pods twocontainers
> $ kubectl delete pods limits-pod
```

Создавать pod в Kubernetes очень легко, но нужно помнить, что такой подход имеет серьезные ограничение в обслуживании. Необходимо самостоятельно заботиться о работе контейнеров в случае сбоев, а так же при масштабировании. Лучшим способом контроля работы pod, это использовать контроллеры. Контроллеры дадут гораздо больший контроль над жизненным циклом pod, включая развертывание новой версии.
