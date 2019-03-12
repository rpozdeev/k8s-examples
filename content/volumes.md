**Volumes**

Volume (Том) - это, по сути, каталог, подключенный к Pod и доступный для всех контейнеров. При перезапуске контейнера данные в томах сохраняются. Kubernetes поддерживает несколько типов `volumes`. Подробнее [тут](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes).

Давайте создадим Pod с двумя контейнерами и подключенным к ним `emptyDir` томом:

```bash
> $ kubectl create -f ./configs/volumes/pod.yaml               
pod/volumes created

> $ kubectl describe pod volumes
Name:               volumes
Namespace:          default
...
Containers:
  client1:
    ...
    Mounts:
      /tmp/data1 from data (rw)
  client2:
    ...
    Mounts:
      /tmp/data2 from data (rw)
...
Volumes:
  data:
    Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:  
  ...
```

Давайте генерируем данные в контейнере `client1`, а затем прочитаем их через контейнер `client2`:

```bash
> $ kubectl exec -it volumes -c client1 -- sh
/ # mount | grep data1
/dev/sda1 on /tmp/data1 type ext4 (rw,relatime,data=ordered)
/ # echo 'data of client1' > /tmp/data1/data
/ # exit
                                                                                                                                                
> $ kubectl exec -it volumes -c client2 -- sh
/ # mount | grep data2
/dev/sda1 on /tmp/data2 type ext4 (rw,relatime,data=ordered)
/ # cat /tmp/data2/data 
data of client1
```

> Обратите внимание, что `emptyDir` нельзя ограничить в ресурсах и в каждом Pod нужно самостоятельно определить куда будет монтироваться `volume`.

Теперь можно удалить создание объекты:

```bash
> $ kubectl delete pods volumes
pod "volumes" deleted
```

> Важно, удаление Pod удали volume вместе с данными.

Дополнительную информацию о `Volumes` можно почитать [здесь](https://kubernetes.io/docs/concepts/storage/volumes/).