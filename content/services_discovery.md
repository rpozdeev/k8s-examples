**Services Discovery**

Services Discovery - это механизм обнаружения подключения к `Services`. Несмотря на то, что есть возможность обнаруживать службы основываясь на переменных среды, предпочтительнее обнаруживать службы основываясь на DNS. DNS является кластерным дополнением.

Давайте создадим `Services` и `ReplicaSet`:

```bash
> $ kubectl create -f ./configs/services/simpliapi-rs-2.yaml
replicaset.apps/simpleapi-rs created

> $ kubectl create -f ./configs/services/services.yaml
service/simpleapi-service created
```

Теперь давайте добавим еще один Pod и попробуем обратиться внутри кластера к сервису apiserver по DNS имени.

```bash
> $ kubectl create -f ./configs/services/client.yaml
pod/client created
```

Подключимся к Pod `client` и попробуем обратиться к серверу по DNS имени:

```bash
> $ kubectl exec -ti client curl http://simpleapi-service.default.svc.cluster.local/info
"Hello, Kubernetes!!!, version: 1.0"
```

или по короткому DNS имени.

```bash
> $ kubectl exec -ti client curl http://simpleapi-service/info
"Hello, Kubernetes!!!, version: 1.0"
```

Полное DNS имя может потребоваться если мы хотим обратиться к `Services` развернутому в другом `namespaces`.

```bash
$SVC.$NAMESPACE.svc.cluster.local
```

Давайте проверим это создав еще один Pod в пространстве имен `other`.

```bash
> $ kubectl create -f ./configs/services/other-ns.yaml
namespace/other created

> $ kubectl create -f ./configs/services/client-2.yaml
pod/client created

> $ kubectl exec -ti -n other client curl http://simpleapi-service.default.svc.cluster.local/info
"Hello, Kubernetes!!!, version: 1.0"

> $ kubectl exec -ti -n other client curl http://simpleapi-service.default/info
"Hello, Kubernetes!!!, version: 1.0"
```

Как вы могли заметить из примера для обращения к сервису находящегося в другом `namespace` достаточно указать только имя `Services` и его `Namespace`. 

Обнаружение сервисов на основе DNS предоставляет гибкий и универсальный способ подключения к Pod в кластере.

Теперь можно удалить создание нами ресурсы:

```bash
> $ kubectl delete replicasets simpleapi-rs
replicaset.extensions "simpleapi-rs" deleted

> $ kubectl delete pods client
pod "client" deleted

> $ kubectl delete pods -n other client
pod "client" deleted
                                      
> $ kubectl delete namespaces other
namespace "other" deleted
```

Важно, удаление пространства имен (`namespaces`) удалит все ресурсы находящиеся в нем.