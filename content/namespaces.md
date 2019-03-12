**Namespaces**

Namespaces - виртуальные кластера, которые поддерживается одним и темнее физическим кластером. Обычно `namespaces` используются для разделения ресурсов кластера между несколькими пользователями (при помощи [квот](https://kubernetes.io/docs/concepts/policy/resource-quotas/)).

Давайте посмотрим какие пространства имен использует `minikube`:

```bash
> $ kubectl get namespaces
NAME          STATUS   AGE
default       Active   4d11h
kube-public   Active   4d11h
kube-system   Active   4d11h
```

При помощи `describe` можно получить подробную информацию о `namespace`:

```bash
> $ kubectl describe namespaces default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No resource limits.
```

Давайте создадим namespace:

```bash
> $ kubectl create -f ./configs/namespaces/namespaces.yaml
namespace/test created

> $ kubectl get namespaces
NAME          STATUS   AGE
default       Active   4d11h
kube-public   Active   4d11h
kube-system   Active   4d11h
test          Active   14s
```

Так же можно создать `namespace` используя команду `kubectl create namespace test`.

Чтобы развернуть Pod в новом namespace выполним следующую команду:

```bash
> $ kubectl create --namespace=test -f ./configs/namespaces/pod.yaml               
pod/ns-test created
```

В данном примере `namespace` передает в момент выполнения конфигурации. Так же `namespace` можно передать в конфигурации.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: ns-test
  namespace: test
```

Чтобы получить объекты определенного пространство имен, нужно выполнить следующую команду:

```bash
> $ kubectl get pods --namespace test
NAME      READY   STATUS    RESTARTS   AGE
ns-test   1/1     Running   0          2m58s
```

Теперь давайте удалим создание объекты.

```bash
> $ kubectl delete deployments simpleapi-deploy
deployment.extensions "simpleapi-deploy" deleted
```

> Важно, удаление `namespaces` удалит все ресурсы находящиеся в нем.

Дополнительную информацию о `namespace` можно почитать [здесь](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/).