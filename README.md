# k8s-examples
Практические примеры работы с дистрибутивом Kubernetes.

В данных примерах используется docker образ melhiades/simpleapi:latest. Это простой api сервер. Возвращающие значения:

| адрес    | данные               |
| -------- | -------------------- |
| /healthy | OK                   |
| /info    | Hello, Kubernetes!!! |

- [Pod](content/pod.md)
- [Labels](content/labels.md)
- [ReplicaSet (ReplicationController)](content/replicaset.md)
- [Deployments](content/deployments.md)
- [Services](content/services.md)
- [Services Discovery](content/services_discovery.md)
- [Health](content/health.md)

Все эти примеры можно выполнить локально у себя на компьютере воспользовавшись дистрибутивом Minikube. Как его установить можно почитать [здесь](https://kubernetes.io/docs/tasks/tools/install-minikube/).

