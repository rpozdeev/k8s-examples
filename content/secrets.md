**Secrets**

Secrets - это объект позволяющий хранить и управлять конфиденциальной информацией, такой как пароли, API ключи, OAuth токены или ssh ключи. Секреты обладают следующими свойствами:

- Это объекты пространства имен, т.е. существуют только в контексте `namespaces`
- Вы можете получить к ним доступ через volume или environments из контейнера, запущенного в Pod
- Эти объекты хранятся на node в томах `tmpfs`
- Существует ограничение на размер одного объекта в 1Mib
- Сервер API хранит secrets в виде открытого текста в etcd

Давайте создадим secrets, `apikey` который содержит ключ API:

```bash
> $ echo -n "AAAHHHRRRNNN1892" > ./apikey.txt

> $ kubectl create secret generic apikey --from-file ./apikey.txt
secret/apikey created

> $ kubectl describe secrets apikey
Name:         apikey
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
apikey.txt:  16 bytes
```

Теперь давайте передадим secrets в Pod через том:

```bash
> $ kubectl create -f ./configs/secrets/pod.yaml
pod/secret created

> $ kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
secret   1/1     Running   0          6s

> $ kubectl exec -ti secret -- sh
/ # mount | grep apikey
tmpfs on /tmp/apikey type tmpfs (ro,relatime)
/ # cat /tmp/apikey/apikey.txt 
AAAHHHRRRNNN1892
```

Мы можем попробовать развернуть Pod и передать этот секрет через переменные окружения:

```bash
> $ echo -n "AAAHHHRRRNNN1892" | base64
QUFBSEhIUlJSTk5OMTg5Mg==

> $ kubectl create -f ./configs/secrets/secrets.yaml
secret/api-secret created
                                                                                                                                                
> $ kubectl create -f ./configs/secrets/pod-env.yaml
pod/secret-env created

> $ kubectl exec -ti secret-env -- sh
/ # echo $SECRET_API
AAAHHHRRRNNN1892
```

Теперь можно удалит развернутые объекты:

```bash
> $ kubectl delete secrets/apikey secrets/api-secret pods/secret-env
secret "apikey" deleted
secret "api-secret" deleted
pod "secret" deleted
pod "secret-env" deleted
```

Дополнительную информацию о `Secrets` можно почитать [здесь](https://kubernetes.io/docs/concepts/configuration/secret/).