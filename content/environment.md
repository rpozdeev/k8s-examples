**Переменные окружения**

Помимо переменных окружения Kubernetes вы можете передать в окружение контейнеров свои собственные переменные окружения.

Давайте создадим Pod и передадим переменную окружения TEST = test_env.

```bash
> $ kubectl create -f ./configs/environment/pod.yaml
pod/env created
```

Теперь давайте проверим переменные окружения Pod:

```bash
> $ kubectl describe pods env | grep IP:
IP:                 172.17.0.6

> $ minikube ssh         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl http://172.17.0.6:5000/env 
"environ({'PATH': '/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin', 'HOSTNAME': 'env', 'TEST': 'test_env', 'KUBERNETES_PORT_443_TCP_PROTO': 'tcp', 'KUBERNETES_PORT_443_TCP_PORT': '443', 'KUBERNETES_PORT_443_TCP_ADDR': '10.96.0.1', 'KUBERNETES_SERVICE_HOST': '10.96.0.1', 'KUBERNETES_SERVICE_PORT': '443', 'KUBERNETES_SERVICE_PORT_HTTPS': '443', 'KUBERNETES_PORT': 'tcp://10.96.0.1:443', 'KUBERNETES_PORT_443_TCP': 'tcp://10.96.0.1:443', 'LANG': 'C.UTF-8'...})"
```

Как мы может увидеть вместе с переменными окружение, которые были сформированы автоматически у нас есть переменная `'TEST': 'test_env'`. 

Есть очень удобный способ просмотреть переменные среды при помощи `kubectl exeC`:

```bash
> $ kubectl exec env -- printenv
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=env
TEST=test_env
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
LANG=C.UTF-8
...
```

Теперь можно удалить созданный Pod:

```bash
> $ kubectl delete pods env
pod "env" deleted
```

Подробнее о передачи переменных окружения можно почитать [тут](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/).