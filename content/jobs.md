**Jobs**

Jobs - это супервизор Pod'ов. Job создает один или несколько Pod'ов и отслеживает успешное выполнение процесса, например вычисление чего-нибудь, операции резервного копирования и т.д. 

Давайте создадим Job который считает от 9 до 1:

```bash
> $ kubectl create -f ./configs/jobs/job.yaml
job.batch/job-count created

> $ kubectl get jobs
NAME        COMPLETIONS   DURATION   AGE
job-count   1/1           31s        50s

> $ kubectl get pods
NAME              READY   STATUS      RESTARTS   AGE
job-count-jx82v   0/1     Completed   0          54s
```

 Давайте посмотрим подробнее на задание:

```bash
> $ kubectl describe jobs job-count
Name:           job-count
Namespace:      default
Selector:       controller-uid=3a73252b-4230-11e9-80b7-001c428d67b9
Labels:         controller-uid=3a73252b-4230-11e9-80b7-001c428d67b9
                job-name=job-count
Annotations:    <none>
Parallelism:    1
Completions:    1
Start Time:     Sat, 09 Mar 2019 08:57:31 +0300
Completed At:   Sat, 09 Mar 2019 08:58:02 +0300
Duration:       31s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=3a73252b-4230-11e9-80b7-001c428d67b9
           job-name=job-count
  Containers:
   client:
    Image:      melhiades/alpine-curl:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      for i in 9 8 7 6 5 4 3 2 1; do echo $i; sleep 3; done
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  4d2h  job-controller  Created pod: job-count-jx82v
  
> $ kubectl logs job-count-jx82v
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
> $ kubectl delete jobs job-count
job.batch "job-count" deleted
```

Дополнительную информацию о Job'ах можно почитать [здесь](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/).