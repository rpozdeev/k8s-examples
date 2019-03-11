**Servises**

Services - это абстракция предоставляющая Pod виртуальный IP адрес (VIP). Pod могут создаваться и удаляться. ReplicaSet может динамически создавать и удалять Pod (при включенном auto scaling). Каждый раз Pod будет получать новый ip адрес. Данная служба позволяет надежно подключаться к контейнерам работающих внутри Pod. VIP это не фактический ip адрес, подключены к сетевому интерфейсу, цель данной службы перенаправить трафик на один или несколько Pod, а так же контролировать доступ при помощи политик. Обновлением соответствия VIP между Pod занимается kube-proxy.

Давай создадим `ReplicaSet` и `Services`:

```bash
> $ kubectl create -f ./configs/services/simpliapi-rs.yaml
replicaset.apps/simpleapi-rs created

> $ kubectl create -f ./configs/services/services.yaml
service/simpleapi-service created
```

Теперь у нас есть управляемый Pod c IP адресом 172.17.0.6:

```bash
> $ kubectl get pods -l app=backend
NAME                 READY   STATUS    RESTARTS   AGE
simpleapi-rs-fdxqr   1/1     Running   0          13s

> $ kubectl describe pod simpleapi-rs-fdxqr
Name:               simpleapi-rs-fdxqr
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.211.55.120
Start Time:         Fri, 08 Mar 2019 13:38:23 +0300
Labels:             app=backend
Annotations:        <none>
Status:             Running
IP:                 172.17.0.6
```

Если подключиться к клакеру, то мы можем получить доступ по назначенному ему IP.

```bash
> $ minikube ssh
There is a newer version of minikube available (v0.35.0).  Download it here:
https://github.com/kubernetes/minikube/releases/tag/v0.35.0

To disable this notification, run the following:
minikube config set WantUpdateNotification false
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl http://172.17.0.6:5000/info
"Hello, Kubernetes!!!, version: 1.0"
$ 
```

Однако, такой способ подключения не рекомендуется, т.к. IP адрес Pod может измениться в будущем. 

Давайте посмотрим подробнее не `Services` который мы создали.

```bash
> $ kubectl get services
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP   3d21h
simpleapi-service   ClusterIP   10.102.82.168   <none>        80/TCP    4m17s

> $ kubectl describe services simpleapi-service
Name:              simpleapi-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=backend
Type:              ClusterIP
IP:                10.102.82.168
Port:              <unset>  80/TCP
TargetPort:        5000/TCP
Endpoints:         172.17.0.6:5000
Session Affinity:  None
Events:            <none>
```

В данном примере сервис отслуживает Pod при помощи метки `app=backend`.

Теперь внутри кластера мы можем получить доступ к сервису по ip адресу 10.102.82.168:

```bash
> $ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl http://10.102.82.168/info
"Hello, Kubernetes!!!, version: 1.0"
```

Внутри `Services` работает iptables, это длинный список правил, который сообщает ядру Linux, что делать с определенным ip пакетом. Давайте взглянем на эти правила:

```bash
$ sudo iptables-save | grep simpleapi
-A KUBE-SERVICES -d 10.102.82.168/32 -p tcp -m comment --comment "default/simpleapi-service: cluster IP" -m tcp --dport 80 -j KUBE-SVC-ZDNBUNH5UBM2YUVP
```

Давайте теперь увеличим реплику до двух Pod и посмотрим на `Services`:

```bash
> $ kubectl scale --replicas=2 replicaset simpleapi-rs
replicaset.extensions/simpleapi-rs scaled

> $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
simpleapi-rs-5g497   1/1     Running   0          30s
simpleapi-rs-fdxqr   1/1     Running   0          15m

> $ kubectl describe services simpleapi-service
Name:              simpleapi-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=backend
Type:              ClusterIP
IP:                10.102.82.168
Port:              <unset>  80/TCP
TargetPort:        5000/TCP
Endpoints:         172.17.0.6:5000,172.17.0.7:5000
Session Affinity:  None
Events:            <none>
```

Как мы видим в `Services` добавился ip адрес второго Pod. Теперь трафик будет равномерно распределяться между этими Pod.

Теперь можно удалить `ReplicaSet` и `Services`:

```bash
> $ kubectl delete services simpleapi-service
service "simpleapi-service" deleted

> $ kubectl delete replicasets simpleapi-rs
replicaset.extensions "simpleapi-rs" deleted
```

Дополнительную информацию о `Services` можно почитать [здесь](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).