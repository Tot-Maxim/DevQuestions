### [Назад к оглавлению](../../README.md)

# CKA (Сертифицированный администратор Kubernetes)

- [CKA (Сертифицированный администратор Kubernetes)](#cka-сертифицированный-администратор-kubernetes)
  - [Настройка](#настройка)
  - [Поды](#поды)
    - [Устранение неполадок с подами](#устранение-неполадок-с-подами)
  - [Пространства имен](#пространства-имен)
  - [Узлы](#узлы)
  - [Сервисы](#сервисы)
  - [ReplicaSets](#replicasets)
    - [Устранение неполадок с ReplicaSets](#устранение-неполадок-с-replicasets)
  - [Развертывания](#развертывания)
    - [Устранение неполадок с развертыванием](#устранение-неполадок-с-развертыванием)
  - [Планировщик](#планировщик)
    - [Привязанность к узлу](#привязанность-к-узлу)
  - [Метки и селекторы](#метки-и-селекторы)
    - [Селектор узлов](#селектор-узлов)
  - [Загрязнения](#загрязнения)
  - [Ограничения ресурсов](#ограничения-ресурсов)
  - [Мониторинг](#мониторинг)
  - [Планировщик](#планировщик-1)

## Настройка

* Настройте кластер Kubernetes. Используйте один из следующих методов:
   1. Minikube для локального, бесплатного и простого кластера.
   2. Управляемый кластер (EKS, GKE, AKS).

* Установите псевдонимы

```
alias k=kubectl
alias kd=kubectl delete
alias kds=kubectl describe
alias ke=kubectl edit
alias kr=kubectl run
alias kg=kubectl get
```

## Поды

<details>
<summary>Запустите команду для просмотра всех подов в текущем пространстве имен</summary><br><b>

`kubectl get pods`

Примечание: создайте псевдоним (`alias k=kubectl`) и привыкните использовать `k get po`.
</b></details>

<details>
<summary>Запустите под с именем "nginx-test" с использованием образа "nginx"</summary><br><b>

`k run nginx-test --image=nginx`
</b></details>

<details>
<summary>Предположим, у вас есть под с именем "nginx-test", как его удалить?</summary><br><b>

`k delete nginx-test`
</b></details>

<details>
<summary>В каком пространстве имен работает под <code>etcd</code>? Перечислите поды в этом пространстве имен</summary><br><b>

`k get po -n kube-system`

Предположим, вы не знали, в каком пространстве имен он находится. Вы можете запустить `k get po -A | grep etc`, чтобы найти под и увидеть, в каком пространстве имен он находится.
</b></details>

<details>
<summary>Перечислите поды из всех пространств имен</summary><br><b>

`k get po -A`

Длинная версия будет `kubectl get pods --all-namespaces`.
</b></details>

<details>
<summary>Напишите YAML для пода с двумя контейнерами и используйте файл YAML для создания пода (используйте любые образы на ваш выбор)</summary><br><b>

```
cat > pod.yaml <<EOL
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: alpine
    name: alpine
  - image: nginx-unprivileged
    name: nginx-unprivileged
EOL

k create -f pod.yaml
```

Если вы спрашиваете себя, как я могу запомнить написание всего этого? Не беспокойтесь, вы можете просто запустить `kubectl run some_pod --image=redis -o yaml --dry-run=client > pod.yaml`. Если вы задаетесь вопросом "как мне запомнить эту длинную команду", пора изменить отношение ;)
</b></details>

<details>
<summary>Создайте YAML для пода, не запуская сам под с помощью команды kubectl (используйте любой образ на ваш выбор)</summary><br><b>

`k run some-pod -o yaml --image nginx-unprivileged --dry-run=client > pod.yaml`
</b></details>

<details>
<summary>Как проверить, является ли манифест действительным?</summary><br><b>

С помощью флага `--dry-run`, который на самом деле не создаст его, но протестирует его и позволит вам найти любые синтаксические ошибки.

`k create -f YAML_FILE --dry-run`
</b></details>

<details>
<summary>Как проверить, какой образ использует конкретный под?</summary><br><b>

`k describe po <POD_NAME> | grep -i image`
</b></details>

<details>
<summary>Как проверить, сколько контейнеров запущено в одном поде?</summary><br><b>

`k get po POD_NAME` и посмотрите число в колонке "READY".

Вы также можете запустить `k describe po POD_NAME`.
</b></details>

<details>
<summary>Запустите под с именем "remo" с последним образом redis и меткой 'year=2017'</summary><br><b>

`k run remo --image=redis:latest -l year=2017`
</b></details>

<details>
<summary>Перечислите поды и их метки</summary><br><b>

`k get po --show-labels`
</b></details>

<details>
<summary>Удалите под с именем "nm"</summary><br><b>

`k delete po nm`
</b></details>

<details>
<summary>Перечислите все поды с меткой "env=prod"</summary><br><b>

`k get po -l env=prod`

Для их подсчета: `k get po -l env=prod --no-headers | wc -l`
</b></details>

<details>
<summary>Создайте статический под с образом <code>python</code>, который выполняет команду <code>sleep 2017</code></summary><br><b>

Сначала перейдите в директорию, отслеживаемую kubelet для создания статического пода: `cd /etc/kubernetes/manifests` (вы можете убедиться в пути, прочитав конфигурационный файл kubelet).

Теперь создайте определение/манифест в этой директории

`k run some-pod --image=python --command sleep 2017 --restart=Never --dry-run=client -o yaml > static-pod.yaml`
</b></details>

<details>
<summary>Опишите, как вы будете удалять статический под</summary><br><b>

Найдите директорию статических подов (посмотрите на `staticPodPath` в конфигурационном файле kubelet).

Перейдите в эту директорию и удалите манифест/определение статического пода (`rm <STATIC_POD_PATH>/<POD_DEFINITION_FILE>`).
</b></details>

### Устранение неполадок с подами

<details>
<summary>Вы пытаетесь запустить под, но видите статус "CrashLoopBackOff". Что это значит? Как определить проблему?</summary><br><b>

Контейнер не удалось запустить (по разным причинам), и Kubernetes пытается запустить под снова после некоторой задержки (= время BackOff).

Некоторые причины для его сбоя:
  - Неправильная конфигурация - опечатки, неподдерживаемые значения и т. д.
  - Ресурс недоступен - узлы отключены, PV не смонтирован и т. д.

Некоторые способы отладки:

1. `kubectl describe pod POD_NAME`
   1. Сосредоточьтесь на `State` (который должен быть Waiting, CrashLoopBackOff) и `Last State`, которые расскажут, что произошло до этого (почему он не удалось).
2. Выполните `kubectl logs mypod`
   1. Это должно предоставить точный вывод.
   2. Для конкретного контейнера вы можете добавить `-c CONTAINER_NAME`.
3. Если вы все еще не понимаете, почему он не удался, попробуйте `kubectl get events`.
</b></details>

<details>
<summary>Что означает ошибка <code>ImagePullBackOff</code>?</summary><br><b>

Скорее всего, вы неверно написали имя образа, который пытаетесь загрузить и запустить. Или, возможно, он не существует в регистре.

Вы можете подтвердить это с помощью `kubectl describe po POD_NAME`.
</b></details>

<details>
<summary>Как проверить, на каком узле запущен определенный под?</summary><br><b>

`k get po POD_NAME -o wide`
</b></details>

<details>
<summary>Запустите следующую команду: <code>kubectl run ohno --image=sheris</code>. Сработает ли это? Почему нет? Исправьте это, не удаляя под и используя любой образ на ваш выбор</summary><br><b>

Потому что такого образа `sheris` не существует. По крайней мере, пока :)

Чтобы исправить это, выполните `kubectl edit ohno` и измените строку `- image: sheris` на `- image: redis` или любой другой образ, который вам нравится.
</b></details>

<details>
<summary>Вы пытаетесь запустить под, но он находится в состоянии "Pending". В чем может быть причина?</summary><br><b>

Одной из возможных причин является то, что планировщик, который должен планировать поды на узлах, не работает. Чтобы проверить это, вы можете запустить `kubectl get po -A | grep scheduler` или проверить прямо в пространстве имен `kube-system`.
</b></details>

<details>
<summary>Как просмотреть логи контейнера, работающего в поде?</summary><br><b>

`k logs POD_NAME`
</b></details>

<details>
<summary>Внутри пода "some-pod" есть два контейнера. Что произойдет, если вы запустите <code>kubectl logs some-pod</code></summary><br><b>

Это не сработает, потому что внутри пода два контейнера, и вам нужно указать один из них с помощью `kubectl logs POD_NAME -c CONTAINER_NAME`.
</b></details>

## Пространства имен

<details>
<summary>Перечислите все пространства имен</summary><br><b>

`k get ns`
</b></details>

<details>
<summary>Создайте пространство имен с именем 'alle'</summary><br><b>

`k create ns alle`
</b></details>

<details>
<summary>Проверьте, сколько пространств имен существует</summary><br><b>

`k get ns --no-headers | wc -l`
</b></details>

<details>
<summary>Проверьте, сколько подов существует в пространстве имен "dev"</summary><br><b>

`k get po -n dev`
</b></details>

<details>
<summary>Создайте под с именем "kartos" в пространстве имен dev. Под должен использовать образ "redis".</summary><br><b>

Если пространство имен еще не существует: `k create ns dev`

`k run kratos --image=redis -n dev`
</b></details>

<details>
<summary>Вы ищете под с именем "atreus". Как проверить, в каком пространстве имен он работает?</summary><br><b>

`k get po -A | grep atreus`
</b></details>

## Узлы

<details>
<summary>Запустите команду для просмотра всех узлов кластера</summary><br><b>

`kubectl get nodes`

Примечание: создайте псевдоним (`alias k=kubectl`) и привыкните к `k get no`.
</b></details>

<details>
<summary>Создайте список всех узлов в формате JSON и сохраните его в файл с именем "some_nodes.json"</summary><br><b>

`k get nodes -o json > some_nodes.json`
</b></details>

<details>
<summary>Проверьте, какие метки есть у одного из ваших узлов в кластере</summary><br><b>

`k get no minikube --show-labels`
</b></details>

## Сервисы

<details>
<summary>Проверьте, сколько сервисов работает в текущем пространстве имен</summary><br><b>

`k get svc`
</b></details>

<details>
<summary>Создайте внутренний сервис с именем "sevi", чтобы открыть приложение 'web' на порту 1991</summary><br><b>
 
`kubectl expose pod web --port=1991 --name=sevi`
</b></details>

<details>
<summary>Как сослаться на сервис с именем "app-service" в том же пространстве имен?</summary><br><b>

app-service
</b></details>

<details>
<summary>Как проверить TargetPort сервиса?</summary><br><b>

`k describe svc <SERVICE_NAME>`
</b></details>

<details>
<summary>Как проверить, какие конечные точки у сервиса?</summary><br><b>

`k describe svc <SERVICE_NAME>`
</b></details>

<details>
<summary>Как сослаться на сервис с именем "app-service" в другом пространстве имен, называемом "dev"?</summary><br><b>

app-service.dev.svc.cluster.local
</b></details>

<details>
<summary>Предположим, у вас есть развертывание, и вам нужно создать службу для открытия подов. Вот что нужно/известно:

* Имя развертывания: jabulik
* Целевой порт: 8080
* Тип службы: NodePort
* Селектор: jabulik-app
* Порт: 8080
</summary><br><b>

`kubectl expose deployment jabulik --name=jabulik-service --target-port=8080 --type=NodePort --port=8080 --dry-run=client -o yaml -> svc.yaml`

`vi svc.yaml` (убедитесь, что селектор установлен на `jabulik-app`)

`k apply -f svc.yaml`
</b></details>

## ReplicaSets

<details>
<summary>Как проверить, сколько реплика сетов определено в текущем пространстве имен?</summary><br><b>

`k get rs`
</b></details>

<details>
<summary>У вас есть реплика сет, определенный для запуска 3 подов. Вы удалили один из этих 3 подов. Что произойдет дальше? Сколько подов будет?</summary><br><b>

Теоретически будет все еще 3 пода, потому что цель реплика сета — гарантировать это. Если вы удалите один или несколько подов, он запустит дополнительные поды, чтобы всегда было 3 пода.
</b></details>

<details>
<summary>Как проверить, какой образ контейнера использовался в реплика сете с именем "repli"?</summary><br><b>

`k describe rs repli | grep -i image`
</b></details>

<details>
<summary>Как проверить, сколько подов готовы в реплика сете с именем "repli"?</summary><br><b>

`k describe rs repli | grep -i "Pods Status"`
</b></details>

<details>
<summary>Как удалить реплика сет с именем "rori"?</summary><br><b>

`k delete rs rori`
</b></details>

<details>
<summary>Как изменить реплика сет с именем "rori", чтобы использовать другой образ?</summary><br><b>

`k edit rs rori`
</b></details>

<details>
<summary>Увеличьте реплика сет с именем "rori", чтобы запустить 5 подов вместо 2</summary><br><b>

`k scale rs rori --replicas=5`
</b></details>

<details>
<summary>Уменьшите реплика сет с именем "rori", чтобы запустить 1 под вместо 5</summary><br><b>

`k scale rs rori --replicas=1`
</b></details>

### Устранение неполадок с ReplicaSets

<details>
<summary>Исправьте следующее определение ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaCet
metadata:
  name: redis
  labels:
    app: redis
    tier: cache
spec:
  selector:
    matchLabels:
      tier: cache
  template:
    metadata:
      labels:
        tier: cachy
    spec:
      containers:
      - name: redis
        image: redis
```
</summary><br><b>

kind должен быть ReplicaSet, а не ReplicaCet :)
</b></details>

<details>
<summary>Исправьте следующее определение ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: redis
  labels:
    app: redis
    tier: cache
spec:
  selector:
    matchLabels:
      tier: cache
  template:
    metadata:
      labels:
        tier: cachy
    spec:
      containers:
      - name: redis
        image: redis
```
</summary><br><b>

Селектор не соответствует метке (cache vs cachy). Чтобы решить это, исправьте cachy так, чтобы это было cache вместо этого.
</b></details>

## Развертывания

<details>
<summary>Как перечислить все развертывания в текущем пространстве имен?</summary><br><b>

`k get deploy`
</b></details>

<details>
<summary>Как проверить, какой образ использует определенное развертывание?</summary><br><b>

`k describe deploy <DEPLOYMENT_NAME> | grep image`
</b></details>

<details>
<summary>Создайте файл определения/манифеста развертывания с именем "dep", с 3 репликами, использующего образ 'redis'</summary><br><b>

`k create deploy dep -o yaml --image=redis --dry-run=client --replicas 3 > deployment.yaml`
</b></details>

<details>
<summary>Удалите развертывание `depdep`</summary><br><b>

`k delete deploy depdep`
</b></details>

<details>
<summary>Создайте развертывание с именем "pluck", используйте образ "redis" и убедитесь, что оно запускает 5 реплик</summary><br><b>

`kubectl create deployment pluck --image=redis --replicas=5`
</b></details>

<details>
<summary>Создайте развертывание со следующими свойствами:

* с именем "blufer"
* использует образ "python"
* запускает 3 реплики
* все поды будут размещены на узле, который имеет метку "blufer"
</summary><br><b>

`kubectl create deployment blufer --image=python --replicas=3 -o yaml --dry-run=client > deployment.yaml`

Добавьте следующий раздел (`vi deployment.yaml`):

```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: blufer
            operator: Exists
```

`kubectl apply -f deployment.yaml`
</b></details>

### Устранение неполадок с развертываниями

<details>
<summary>Исправьте следующий манифест развертывания

```yaml
apiVersion: apps/v1
kind: Deploy
metadata:
  creationTimestamp: null
  labels:
    app: dep
  name: dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: dep
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
status: {}
```
</summary><br><b>

Измените `kind: Deploy` на `kind: Deployment`.
</b></details>

<details>
<summary>Исправьте следующий манифест развертывания

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: dep
  name: dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: depdep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: dep
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
status: {}
```
</summary><br><b>

Селектор не соответствует метке (dep vs depdep). Чтобы решить это, исправьте depdep так, чтобы это было dep вместо этого.
</b></details>

## Планировщик

<details>
<summary>Как запланировать под на узле с именем "node1"?</summary><br><b>

`k run some-pod --image=redix -o yaml --dry-run=client > pod.yaml`

`vi pod.yaml` и добавьте:

```
spec:
  nodeName: node1
```

`k apply -f pod.yaml`

Примечание: если у вас нет узла node1 в вашем кластере, под будет застревать в состоянии "Pending".
</b></details>

### Привязанность к узлу

<details>
<summary>Используя привязанность к узлу, установите под для планирования на узле, где ключ "region" и значение либо "asia", либо "emea"</summary><br><b>

`vi pod.yaml`

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: region
          operator: In
          values:
          - asia
          - emea
```
</b></details>

<details>
<summary>Используя привязанность к узлу, установите под так, чтобы он никогда не планировался на узле, где ключ "region" и значение "neverland"</summary><br><b>

`vi pod.yaml`

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: region
          operator: NotIn
          values:
          - neverland
```
</b></details>

## Метки и селекторы

<details>
<summary>Как перечислить все поды с меткой "app=web"?</summary><br><b>

`k get po -l app=web`
</b></details>

<details>
<summary>Как перечислить все объекты, помеченные как "env=staging"?</summary><br><b>

`k get all -l env=staging`
</b></details>

<details>
<summary>Как перечислить все развертывания с метками "env=prod" и "type=web"?</summary><br><b>

`k get deploy -l env=prod,type=web`
</b></details>

### Селектор узлов

<details>
<summary>Примените метку "hw=max" на одном из узлов вашего кластера</summary><br><b>

`kubectl label nodes some-node hw=max`
</b></details>

<details>
<summary>Создайте и запустите под с именем `some-pod` с образом `redis` и настройте его на использование селектора `hw=max`</summary><br><b>

```
kubectl run some-pod --image=redis --dry-run=client -o yaml > pod.yaml

vi pod.yaml

spec:
  nodeSelector:
    hw: max

kubectl apply -f pod.yaml
```
</b></details>

<details>
<summary>Объясните, почему селекторы узлов могут быть ограничены</summary><br><b>

Предположим, вы хотите запустить свой под на всех узлах, у которых либо `hw` установлен на max, либо на min, а не только на max. Это невозможно с селекторами узлов, которые достаточно упрощены, и здесь вы можете рассмотреть возможность использования `node affinity`.
</b></details>

## Загрязнения

<details>
<summary>Проверьте, есть ли загрязнения на узле "master"</summary><br><b>

`k describe no master | grep -i taints`
</b></details>

<details>
<summary>Создайте загрязнение на одном из узлов вашего кластера с ключом "app" и значением "web" и эффектом "NoSchedule". Проверьте, было ли оно применено</summary><br><b>

`k taint node minikube app=web:NoSchedule`

`k describe no minikube | grep -i taints`
</b></details>

<details>
<summary>Вы применили загрязнение с <code>k taint node minikube app=web:NoSchedule</code> на единственном узле в вашем кластере, а затем выполнили <code>kubectl run some-pod --image=redis</code>. Что произойдет?</summary><br><b>

Под останется в статусе "Pending" из-за того, что единственный узел в кластере имеет загрязнение "app=web".
</b></details>

<details>
<summary>Вы применили загрязнение с <code>k taint node minikube app=web:NoSchedule</code> на единственном узле в вашем кластере, а затем выполнили <code>kubectl run some-pod --image=redis</code>, но под находится в состоянии ожидания. Как это исправить?</summary><br><b>

`kubectl edit po some-pod` и добавьте следующее

```
  - effect: NoSchedule
    key: app
    operator: Equal
    value: web
```

Выйдите и сохраните. Под должен теперь находиться в состоянии Running.
</b></details>

<details>
<summary>Удалите существующее загрязнение с одного из узлов вашего кластера</summary><br><b>

`k taint node minikube app=web:NoSchedule-`
</b></details>

## Ограничения ресурсов

<details>
<summary>Проверьте, есть ли какие-либо ограничения на одном из подов в вашем кластере</summary><br><b>

`kubectl describe po <POD_NAME> | grep -i limits`
</b></details>

<details>
<summary>Запустите под с именем "yay" с образом "python" и запросами ресурсов на 64Mi памяти и 250m CPU</summary><br><b>

`kubectl run yay --image=python --dry-run=client -o yaml > pod.yaml`

`vi pod.yaml`

```
spec:
  containers:
  - image: python
    imagePullPolicy: Always
    name: yay
    resources:
      requests:
        cpu: 250m
        memory: 64Mi
```

`kubectl apply -f pod.yaml`
</b></details>

<details>
<summary>Запустите под с именем "yay2" с образом "python". Убедитесь, что у него есть запросы ресурсов на 64Mi памяти и 250m CPU, а ограничения составляют 128Mi памяти и 500m CPU</summary><br><b>

`kubectl run yay2 --image=python --dry-run=client -o yaml > pod.yaml`

`vi pod.yaml`

```
spec:
  containers:
  - image: python
    imagePullPolicy: Always
    name: yay2
    resources:
      limits:
        cpu: 500m
        memory: 128Mi
      requests:
        cpu: 250m
        memory: 64Mi
```

`kubectl apply -f pod.yaml`
</b></details>

## Мониторинг

<details>
<summary>Разверните metrics-server</summary><br><b>

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`
</b></details>

<details>
<summary>Используя metrics-server, просмотрите следующее:

* поды с наилучшей производительностью в кластере
* узлы с наилучшей производительностью
</summary><br><b>

* топ узлы: `kubectl top nodes`
* топ поды: `kubectl top pods`
</b></details>

## Планировщик

<details>
<summary>Можете ли вы развернуть несколько планировщиков?</summary><br><b>

Да, это возможно. Вы можете запустить другой под с командой, подобной следующей:

```
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --leader-elect=true
    - --scheduler-name=some-custom-scheduler
...
```
</b></details>

<details>
<summary>Предположим, у вас есть несколько планировщиков, как узнать, какой планировщик был использован для данного пода?</summary><br><b>

Запустив `kubectl get events`, вы можете увидеть, какой планировщик был использован.
</b></details>

<details>
<summary>Вы хотите запустить новый под и хотите, чтобы его запланировал пользовательский планировщик. Как этого достичь?</summary><br><b>

Добавьте следующее в спецификацию пода:

```
spec:
  schedulerName: some-custom-scheduler
```
</b></details>