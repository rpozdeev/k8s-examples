# k8s-example
Практические примеры работы с дистрибутивом Kubernetes.

В данных примерах используется docker образ melhiades/simpleapi:latest. Это простой api сервер. Возвращающие значения:

- | ссылка   | данные               |
  | -------- | -------------------- |
  | /healthy | OK                   |
  | /info    | Hello, Kubernetes!!! |

- [pod](content/pod.md)
- 

Все эти примеры можно выполнить локально у себя на компьютере воспользовавшись дистрибутивом Minikube. Как его установить можно почитать [здесь](https://kubernetes.io/docs/tasks/tools/install-minikube/).