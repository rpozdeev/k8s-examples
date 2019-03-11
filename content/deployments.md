**Deployments**

Deployments - супервизор для ваших Pod. Этот контроллер расширяет возможности ReplicaSet контроллера давай детальный контроль над тем, как и когда выкатывается новая версия Pod, а так же предоставляет возможность вернуться к предыдущему состоянию.

Давайте создадим `deployments` который контролирует две реплики (`replicaset`) Pod.

```bash
> $ kubectl create -f ./configs/deployments/simpliapi-deploy-09.yaml
deployment.apps/simpleapi-deploy created
```

 Теперь мы можем взглянуть на Pod и `replicasets` которые создал контроллер `deployments`.

```bash
> $ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
simpleapi-deploy   2/2     2            2           14s

> $ kubectl get replicasets
NAME                          DESIRED   CURRENT   READY   AGE
simpleapi-deploy-6cd6bb88b5   2         2         2       26s

> $ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
simpleapi-deploy-6cd6bb88b5-pd6z2   1/1     Running   0          37s
simpleapi-deploy-6cd6bb88b5-vr6mp   1/1     Running   0          37s
```

ОБратите внимание на генерированные имена `replicasets` и Pod, которые получились из имени `deployments`.

Сейчас развернут сервер версии 0.9. Давайте это проверим.

```bash
> $ kubectl exec simpleapi-deploy-6cd6bb88b5-pd6z2 -c client -ti sh
/ # curl http://localhost:5000/info
"Hello, Kubernetes!!!, version: 0.9"
```

Теперь давайте попробуем обновить `deployments` и посмотрим, что произойдет.

```bash
> $ kubectl create -f ./configs/deployments/simpliapi-deploy-10.yaml
deployment.apps/simpleapi-deploy created

> $ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
simpleapi-deploy-6cd6bb88b5   0         0         0       13m
simpleapi-deploy-7dd47cdc77   2         2         2       2m4s

> $ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
simpleapi-deploy-6cd6bb88b5-pd6z2   2/2     Running             0          75s
simpleapi-deploy-6cd6bb88b5-vr6mp   2/2     Running             0          72s
simpleapi-deploy-7dd47cdc77-72tsd   0/2     ContainerCreating   0          2s

> $ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
simpleapi-deploy-6cd6bb88b5-pd6z2   2/2     Running             0          78s
simpleapi-deploy-6cd6bb88b5-vr6mp   2/2     Terminating         0          75s
simpleapi-deploy-7dd47cdc77-4rqkg   0/2     ContainerCreating   0          1s
simpleapi-deploy-7dd47cdc77-72tsd   2/2     Running             0          5s

> $ kubectl get pods
NAME                                READY   STATUS        RESTARTS   AGE
simpleapi-deploy-6cd6bb88b5-pd6z2   2/2     Terminating   0          107s
simpleapi-deploy-6cd6bb88b5-vr6mp   2/2     Terminating   0          104s
simpleapi-deploy-7dd47cdc77-4rqkg   2/2     Running       0          30s
simpleapi-deploy-7dd47cdc77-72tsd   2/2     Running       0          34s
                                       
> $ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
simpleapi-deploy-7dd47cdc77-4rqkg   2/2     Running   0          2m23s
simpleapi-deploy-7dd47cdc77-72tsd   2/2     Running   0          2m27s
```

Мы можем увидеть, что параллельно разворачиваться Pod с версией сервера 1.0, в этот момент существуют две `replicasets`. Когда новые Pod развернуты, старые Pod удаляются. Такого же эффекта можено было достичь изменив конфигурацию `deployments`.

```bash
> $ kubectl edit deployments.apps simpleapi-deploy
```

Давайте убедимся, что новая версия api севера развернута.

```bash
> $ kubectl exec simpleapi-deploy-7dd47cdc77-4rqkg -c client -ti sh 
/ # curl http://localhost:5000/info
"Hello, Kubernetes!!!, version: 1.0"
```

История всех изменений в контролере `deployments` доступна по команде:

```bash
> $ kubectl rollout history deployment simpleapi-deploy
deployment.extensions/simpleapi-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

Если во время новой выкати произошла ошибка или по какой-то причине Pod не может запуститься, `deployments` автоматически откатится к предыдущей версии. Также вы можете явно откатиться к любой предыдущей версии Pod.

```bash
> $ kubectl rollout undo deployment simpleapi-deploy --to-revision=1
deployment.extensions/simpleapi-deploy rolled back

> $ kubectl rollout history deployment simpleapi-deploy
deployment.extensions/simpleapi-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

Давайте убедимся, что предыдущая версия api севера развернута.

```bash
> $ kubectl exec simpleapi-deploy-6cd6bb88b5-vnhhk -c client -ti sh   
/ # curl http://localhost:5000/info
"Hello, Kubernetes!!!, version: 0.9"
```

Мы вернулись на предыдущую версию api сервера версии 0.9.

Теперь давайте удалим наш `deployments`.

```bash
> $ kubectl delete deployments simpleapi-deploy
deployment.extensions "simpleapi-deploy" deleted
```

Дополнительную информацию о `Deployments` можно почитать [здесь](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).