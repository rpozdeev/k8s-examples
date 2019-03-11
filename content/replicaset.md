**ReplicaSet/ReplicationController**

ReplicaSet - контроллер, который поддерживает стабильный набор работающих Pod в любой момент времени. Он часто используется, чтобы гарантировать наличие определенного количество идентичных Pod.

replicaSet является следующей версией ReplicationController. Они ведут себя одинаково и предназначены для одной и той же цели, за исключением того, что ReplicationController не поддерживает селекторы на основе набора.

Т.к. рекомендуется использовать ReplicaSet, ReplicationController рассматриваться не будет.

Давайте создадим `replicaset` который разворачивать две реплики Pod.

```bash
> $ kubectl create -f ./configs/replicaset/simpliapi-rs.yaml
replicaset.apps/simpleapi-rs created
```

 Теперь мы можем взглянуть на `pod` и `replicaset` которые мы создали.

```bash
> $ kubectl get rs
NAME           DESIRED   CURRENT   READY   AGE
simpleapi-rs   2         2         2       7s

> $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
simpleapi-rs-8xw5d   1/1     Running   0          10s
simpleapi-rs-fftmv   1/1     Running   0          10s
```

Обратите внимание на генерированные имена `pod`, которые получились из имени `replicaset`.

Давайте посмотрим описание `replocaset`:

```bash
> $ kubectl describe rs simpleapi-rs                                                                                       ⬡ 8.12.0 [±master ●]
Name:         simpleapi-rs
Namespace:    default
Selector:     app=backend
Labels:       app=simpleapi
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=backend
  Containers:
   apiserver:
    Image:        melhiades/simpleapi:latest
    Port:         5000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  2d20h  replicaset-controller  Created pod: simpleapi-rs-8xw5d
  Normal  SuccessfulCreate  2d20h  replicaset-controller  Created pod: simpleapi-rs-fftmv
```

Здесь нужно обратить внимание на `selector`. В нем нужно указывать `labels` Pod, который необходимо отреплицировать ( `app=backend`).

Данный контроллер вы будете использовать редко. Рекомендуется использовать `Deployments` т.к. В нем реализован весь функционал `ReplicaSet` контроллера плюс дополнительные возможности по работе с `pod`.

Теперь можно удалить `ReplicaSet` и связанные с ним `pod`.

```bash
> $ kubectl delete replicasets simpleapi-rs
replicaset.apps "simpleapi-rs" deleted
```

Дополнительную информацию о `ReplicaSet` можно почитать [здесь](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).