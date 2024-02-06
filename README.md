# Домашнее задание к занятию «Запуск приложений в K8S»

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: 1-deployment
  labels:
    app: 1-deployment
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: 1-deployment
  template:
    metadata:
      labels:
        app: 1-deployment
      namespace: default
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        imagePullPolicy: IfNotPresent
      - name: multitool
        image: wbitt/network-multitool
        imagePullPolicy: IfNotPresent
        env:
          - name: HTTP_PORT
            value: '8080'
          - name: HTTPS_PORT
            value: '8081'
```

<p align="center">
    <img width="1200 height="600" src="/img/error_deploy.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/start_deploy.png">
</p>

2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.

<p align="center">
    <img width="1200 height="600" src="/img/change_deploy.png">
</p>

4. Создать Service, который обеспечит доступ до реплик приложений из п.1.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-deploy
  namespace: default
spec:
  ports:
    - name: tcp
      port: 80
      targetPort: 9376
  selector:
    app: 1-deployment
```

<p align="center">
    <img width="1200 height="600" src="/img/service_deploy.png">
</p>

5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: multitool
  name: pod-multitool
  namespace: default
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    ports:
    - containerPort: 8080
```

<p align="center">
    <img width="1200 height="600" src="/img/pod-multitool.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/curl_pod.png">
</p>

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: 2-deployment
  labels:
    app: 2-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: 2-deployment
  template:
    metadata:
      labels:
        app: 2-deployment
      namespace: default
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
      initContainers:
      - name: init-service-deploy
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup service-deploy.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
```

2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.

<p align="center">
    <img width="1200 height="600" src="/img/init_pod_deploy.png">
</p>

3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

<p align="center">
    <img width="1200 height="600" src="/img/init_pod_create_service.png">
</p>

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
