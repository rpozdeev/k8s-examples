**Logging**

Логирование - это один из способов понять, что происходит внутри ваших приложений и кластера в целом. Базовое логирование в Kubernetes делает доступным вывод контейнера, это очень хорошо помогает при отладке.

Давайте создадим Pod logger, который каждую секунду будет выводит дату и время:

```bash
> $ kubectl create -f ./configs/logger/pod.yaml
pod/logger created
```

Чтобы посмотреть последние 5 строк журнала воспользуемся командой:

```bash
> $ kubectl logs --tail=5 logger -с
Sat Mar 9 05:07:58 UTC 2019
Sat Mar 9 05:07:59 UTC 2019
Sat Mar 9 05:08:00 UTC 2019
Sat Mar 9 05:08:01 UTC 2019
Sat Mar 9 05:08:02 UTC 2019
```

Если мы хотим, чтобы журнал выводился потоком, воспользуемся командой:

```bash
> $ kubectl logs -f logger -c client
Sat Mar 9 05:06:46 UTC 2019
Sat Mar 9 05:06:47 UTC 2019
Sat Mar 9 05:06:48 UTC 2019
...
```

В данном случае мы получили все спори журнала, так же мы можем указать с какой записи нам интересен вывод, при помощи —since

```bash
> $ kubectl logs -f --since=10s logger -c client
Sat Mar 9 05:11:20 UTC 2019
Sat Mar 9 05:11:21 UTC 2019
Sat Mar 9 05:11:22 UTC 2019
...
```

Так же можно посмотреть журнал Pod которые работают или которые уже завершили свою работу. Давай создадим Pod, который выводит значения от 9 до 1, а затем прекращает свою работу.

```bash
> $ kubectl create -f ./configs/logger/pod-endl.yaml 
pod/logger-count created

> $ kubectl get pods
NAME           READY   STATUS             RESTARTS   AGE
logger         1/1     Running            0          10m
logger-count   0/1     CrashLoopBackOff   1          14s

> $ kubectl logs logger-count -c client
9
8
7
6
5
4
3
2
1
```

Теперь можно удалить создание объекты:

```bash
> $ kubectl delete pods logger logger-count
pod "logger" deleted
pod "logger-count" deleted
```

Дополнительную информацию о логировании можно почитать [здесь](https://kubernetes.io/docs/concepts/cluster-administration/logging/).