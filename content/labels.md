**Labels**

Labels (метки) - это механизм организации и связывания всех объектов Kubernetes. Labels представляет собой пару ключ:значение `env: development`. Labels имеют некоторые ограничения, прочитать об этом можно [тут](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set).

Давайте создадим pod, который будет иметь label  `env: development` [конфигурация](https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/labels/pod.yaml)

```bash
> $ kubectl create -f https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/labels/pod.yaml                                                                     
pod/apiserver created

> $ kubectl get pods --show-labels                                                                                  
NAME        READY   STATUS    RESTARTS   AGE   LABELS
apiserver   1/1     Running   0          18s   env=development
```

Параметр `—show-labels` выводит значения всех меток в дополнительном столбце.

Так же можно добавить labels к уже работающему pod:

```bash
> $ kubectl label pods apiserver owner=rpozdeev                                                                    
pod/apiserver labeled

> $ kubectl get pods --show-labels                                                                                 
NAME        READY   STATUS    RESTARTS   AGE     LABELS
apiserver   1/1     Running   0          5m13s   env=development,owner=rpozdeev
```

**Label selectors**

При помощи `selector` можно искать нужные pod по `labels`. Для примера давайте создадим еще один pod с следующими labels `env: production, owner: rpozdeev`. [конфигурация](https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/labels/pod-production.yaml)

```bash
> $ kubectl create -f https://raw.githubusercontent.com/rpozdeev/k8s-examples/master/configs/labels/pod-production.yaml          
pod/apiserver-prod created

> $ kubectl get pods --show-labels                                                                                  
NAME             READY   STATUS    RESTARTS   AGE   LABELS
apiserver        1/1     Running   0          12m   env=development,owner=rpozdeev
apiserver-prod   1/1     Running   0          8s    env=production,owner=rpozdeev
```

Чтобы использовать `labels` для фильтрации pod необходимо использовать параметр `--selector` или `-l`.

```bash
> $ kubectl get pods --selector owner=rpozdeev                                                                      
NAME             READY   STATUS    RESTARTS   AGE
apiserver        1/1     Running   0          15m
apiserver-prod   1/1     Running   0          3m20s
                                                                                                                                          
> $ kubectl get pods -l env=production                                                                             
NAME             READY   STATUS    RESTARTS   AGE
apiserver-prod   1/1     Running   0          3m41s
```

 Селекторы могут искать pod по нескольким `labels`. А также использовать операторы  in, &&, =, ==, !=. Подробнее [тут](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors).

```bash
> $ kubectl get pods -l 'env in (production, development)'                                                                
NAME             READY   STATUS    RESTARTS   AGE
apiserver        1/1     Running   0          19m
apiserver-prod   1/1     Running   0          7m50s
                                                                                                                                                
> $ kubectl get pods -l 'env !=  development'                                                                             
NAME             READY   STATUS    RESTARTS   AGE
apiserver-prod   1/1     Running   0          8m3s
```

Другие команды в Kubernetes так же поддерживают селекторы. В качестве примера мы можем удалить создание pod при помощи селектора.

```bash
> $ kubectl delete pods -l 'env in (production, development)                                                                                                                                                           
pod "apiserver-prod" deleted                                                                                                      
pod "apiserver" deleted
```

Эта команда аналогична двум следующим:

```bash
> $ kubectl delete pods apiserver-prod
> $ kubectl delete pods apiserver
```

Важно заметить, что `labels` не ограничиваются работой с pod, их можно применять к разным объектам, например к `node` или `services`.