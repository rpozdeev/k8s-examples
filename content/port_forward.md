**Port Forward**

При разработки приложения в Kubernetes часто бывает полезно получить быстрый доступ к сервису из локальной сети. В этом случае мы можем воспользоваться перенаправление портов `Port Forward`. 

Давай развернем приложение `SimpleApi` и `Services`:

```bash
> $ kubectl create -f ./configs/portforward/simpliapi-rs.yaml
replicaset.apps/simpleapi-rs created

> $ kubectl create -f ./configs/portforward/services.yaml
service/simpleapi-service created
```

Допустим мы хотим получить доступ к приложению `SimpleApi` с нашего ноутбука, через порт 8080.

```bash
> $ kubectl port-forward service/simpleapi-service 8080:80
Forwarding from 127.0.0.1:8080 -> 5000
Forwarding from [::1]:8080 -> 5000
Handling connection for 8080
```

Теперь мы можем обратиться к нашему сервису локально.

```bash
> $ curl http://localhost:8080/info
"Hello, Kubernetes!!!, version: 1.0"
```

Теперь можно удалить `ReplicaSet` и `Services`:

```bash
> $ kubectl delete services simpleapi-service
service "simpleapi-service" deleted

> $ kubectl delete replicasets simpleapi-rs
replicaset.extensions "simpleapi-rs" deleted
```

`Port Forward` предназначено только для разработки и экспериментов с приложением. Для промышленной эксплуатации такой способ не подходит.

Дополнительную информацию о `Port Forward` можно почитать [здесь](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).