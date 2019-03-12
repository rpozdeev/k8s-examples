**Health Checks**

Чтобы убедиться, что контейнер в Pod исправлен и правильно работает, Kubernetes предоставляет ряд механизмов проверки работоспособности. Health check или как их называют в Kubernetes - probes. `Kubelet` использует `livenessProbe`, чтобы определить когда нужно перезапустить контейнер, а так же `readinessProbe`, чтобы определить когда контейнер готов получать трафик.

В данном примере мы рассмотрим проверку состояния HTTP. Важное замечание, разработчик приложения обязан предоставить URL-адрес, который можно использовать для определения работоспособности контейнера. 

Давайте развернем Pod который имеет адрес /health и возвращает по этому адресу 200 код.

```bash
> $ kubectl create -f ./configs/health/pod.yaml
pod/healthcheck created
```

В спецификации Pod определено:

```yaml
livenessProbe:
  initialDelaySeconds: 2
  periodSeconds: 5
  httpGet:
    path: /health
    port: 5000
```

Kubernetes начнет проверять адрес /health через 2 секунды каждые 5 секунд.

```bash
> $ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
healthcheck   1/1     Running   0          5s

> $ kubectl describe pod healthcheck
Name:               healthcheck
...
    Liveness:       http-get http://:5000/health delay=2s timeout=1s period=5s  #success=1 #failure=3
....

```

Теперь давайте развернет еще один Pod, но в этом случае будем проверять адрес /bad_health, который в половине случаев возвращает код 404.

```bash
> $ kubectl create -f ./configs/health/bad_pod.yaml
pod/healthcheck-bad created

> $ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
healthcheck       1/1     Running   0          21m
healthcheck-bad   1/1     Running   3          116s

> $ kubectl describe pod healthcheck-bad
Name:               healthcheck-bad
...
    Restart Count:  3
    Liveness:       http-get http://:5000/bad_health delay=2s timeout=1s period=5s #success=1 #failure=3
...
Events:
  Type     Reason     Age                     From               Message
  ----     ------     ----                    ----               -------
  Normal   Scheduled  3d11h                   default-scheduler  Successfully assigned default/healthcheck-bad to minikube
  Normal   Pulling    3d11h (x3 over 3d11h)   kubelet, minikube  pulling image "melhiades/simpleapi:latest"
  Normal   Killing    3d11h (x2 over 3d11h)   kubelet, minikube  Killing container with id docker://apiserver:Container failed liveness probe.. Container will be killed and recreated.
  Normal   Pulled     3d11h (x3 over 3d11h)   kubelet, minikube  Successfully pulled image "melhiades/simpleapi:latest"
  Normal   Created    3d11h (x3 over 3d11h)   kubelet, minikube  Created container
  Normal   Started    3d11h (x3 over 3d11h)   kubelet, minikube  Started container
  Warning  Unhealthy  3d11h (x11 over 3d11h)  kubelet, minikube  Liveness probe failed: HTTP probe failed with statuscode: 404
```

Pod `healthcheck-bad` за 116 секунд был перезапущен 3 раза, это также видно в событиях Pod.

В дополнении к `livenessProbe` существует еще один механизм `readinessProbe`. Он служит для проверки условий работы контейнера. К примеру вашему приложению могут потребоваться конфигурационные файлы, которые формируются в момент запуска контейнера, вам может потребоваться подгружать данные с внешнего сервиса или необходимо дождаться инициализации базы данных.

Давайте создадим Pod, который ждет 10 секунд и если проверка проходит, то контейнер запускается.

```bash
> $ kubectl create -f ./configs/health/pod-ready.yaml               
pod/ready created

> $ kubectl get pods
NAME              READY   STATUS             RESTARTS   AGE
healthcheck       1/1     Running            0          37m
healthcheck-bad   0/1     CrashLoopBackOff   9          18m
ready             0/1     Running            0          7s

> $ kubectl describe pod ready
Name:               ready
...
    Readiness:      http-get http://:5000/health delay=10s timeout=1s period=10s #success=1 #failure=3
...
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
...
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3d11h  default-scheduler  Successfully assigned default/ready to minikube
  Normal  Pulling    3d11h  kubelet, minikube  pulling image "melhiades/simpleapi:latest"
  Normal  Pulled     3d11h  kubelet, minikube  Successfully pulled image "melhiades/simpleapi:latest"
  Normal  Created    3d11h  kubelet, minikube  Created container
  Normal  Started    3d11h  kubelet, minikube  Started container

> $ kubectl describe pod ready
Name:               ready
...
    Readiness:      http-get http://:5000/health delay=10s timeout=1s period=10s #success=1 #failure=3
...
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
...
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3d11h  default-scheduler  Successfully assigned default/ready to minikube
  Normal  Pulling    3d11h  kubelet, minikube  pulling image "melhiades/simpleapi:latest"
  Normal  Pulled     3d11h  kubelet, minikube  Successfully pulled image "melhiades/simpleapi:latest"
  Normal  Created    3d11h  kubelet, minikube  Created container
  Normal  Started    3d11h  kubelet, minikube  Started container
```

Теперь можно удалить все созданы Pod:

```bash
> $ kubectl delete pods healthcheck healthcheck-bad ready
pod "healthcheck" deleted
pod "healthcheck-bad" deleted
pod "ready" deleted
```

Дополнительную информацию о настройки `probes` можно почитать [здесь](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/).