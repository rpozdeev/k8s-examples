**Persistent Volumes**

Persistent Volumes (PV) - представляет собой ресурс кластера, который можно использовать для хранения данных, которые существуют вне жизненного цикла Pod. PV использует сетевые хранилища данных, такие как EBS или NFS, iSCSI или FC, распределенные файловые системы, такие как Ceph или etcd.

Чтобы воспользоваться PV необходим запросить его, через `PersistentVolumeClaim`. Это чем-то похоже на Pod, Pod использует ресурсы node, а PVC использует ресурсы PV. PVC запрашивает PV с определенной спецификацией (размер, скорость и т.д.) у Kubernetes, а затем связывает его с Pod, где его можно будет смонтировать как Volume. 

Итак, давайте создадим PVC размером в 1Gi  и классом обслуживанию по умолчанию:

```bash
> $ kubectl create -f ./configs/persistentvolumes/pvc.yaml
persistentvolumeclaim/testclaim created

> $ kubectl get persistentvolumeclaims
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
testclaim   Bound    pvc-5413a371-420e-11e9-80b7-001c428d67b9   1Gi        RWO            standard       6m58s
```

Для того чтобы понять как работают PV, давайте создадим `Deployments` c подключенным PVC в `/tmp/persistent`:

```bash
> $ kubectl create -f ./configs/persistentvolumes/pvc-pod.yaml
deployment.apps/test-pvc created
```

Для того чтобы проверить, что данные сохраняться после удаления Pod, давайте создадим файл в каталоге `/tmp/persistent`:

```bash
> $ kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
test-pvc-657c598ddc-62chh   1/1     Running   0          3m41s

> $ kubectl exec -ti test-pvc-657c598ddc-62chh -- sh
/ # touch /tmp/persistent/data
/ # ls /tmp/persistent/
data
```

Теперь давайте удалим этот Pod и посмотрим, что будет в каталоге  `/tmp/persistent` после того как `Deployments` развернет новый Pod:

```bash
> $ kubectl delete pods test-pvc-657c598ddc-62chh
pod "test-pvc-657c598ddc-62chh" deleted

> $ kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
test-pvc-657c598ddc-kxvsl   1/1     Running   0          50s

> $ kubectl exec -ti test-pvc-657c598ddc-kxvsl -- sh
/ # ls /tmp/persistent/
data
```

Как мы видим файл `data` остался и доступен в новом Pod.

> Обратите внимание, что поведение по умолчанию не удаляет PVC и PV при удалении Deployments. Такое поведение позволяет защитить данные. Если вы уверены, что данные вам больше не нужны удалите PVC и вместе в ним удалится PV.

Теперь можно удалит развернутые объекты:

```bash
> $ kubectl delete deployments test-pvc
deployment.extensions "test-pvc" deleted

> $ kubectl delete persistentvolumeclaims testclaim
persistentvolumeclaim "testclaim" deleted
```

Дополнительную информацию о `Persistent Volumes` можно почитать [здесь](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).