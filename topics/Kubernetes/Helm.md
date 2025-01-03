### [Назад к оглавлению](../../README.md)

### [Helm](https://helm.sh/docs/)

### Helm

<details>
<summary>Что такое Helm?</summary><br><b>

Диспетчер пакетов для Kubernetes. Грубо говоря, это возможность упаковать YAML-файлы и распространять их среди других пользователей и применять их в кластер(ах).

Как концепция, это довольно распространено и можно найти во многих платформах и сервисах. Например, думайте о диспетчерах пакетов в операционных системах. Если вы используете Fedora/RHEL, это будет dnf. Если вы используете Ubuntu, тогда apt. Если вы не используете Linux, тогда следует задать другой вопрос — почему? Но это уже другая тема :)
</b></details>

<details>
<summary>Зачем нам Helm? Какой сценарий его использования?</summary><br><b>

Иногда, когда вы хотите развернуть определенное приложение в своем кластере, вам необходимо создать несколько YAML-файлов/компонентов, таких как: Секрет, Сервис, ConfigMap и т.д. Это может быть утомительной задачей. Поэтому будет целесообразно упростить процесс, предоставив что-то, что позволит нам делить эти пакеты YAML каждый раз, когда мы хотим добавить приложение в наш кластер. Это как раз и есть Helm.

Распространенный сценарий — это наличие нескольких кластеров Kubernetes (prod, dev, staging). Вместо того чтобы индивидуально применять различные YAML в каждом кластере, имеет больше смысла создать одну и ту же Чартию и установить её в каждый кластер.

Еще один сценарий — вы хотите поделиться тем, что создали, с сообществом. Для людей и компаний будет проще развернуть ваше приложение в их кластере.
</b></details>

<details>
<summary>Объясните "Helm Charts"</summary><br><b>

Helm Charts — это набор файлов YAML. Пакет, который вы можете использовать из репозиториев или создать свой собственный и опубликовать его в репозитории.
</b></details>

<details>
<summary>Говорят, что Helm также является Шаблонизатором. Что это означает?</summary><br><b>

Это полезно в сценариях, когда у вас несколько приложений, и все они похожи, так что есть небольшие различия в их конфигурационных файлах, и большинство значений одинаковы. С помощью Helm вы можете определить общий шаблон для всех них, а значения, которые не фиксированы и могут изменяться, могут быть местозаполнителями. Это называется файл шаблона и выглядит примерно так

```
apiVersion: v1
kind: Pod
metadata:
  name: {[ .Values.name ]}
spec:
  containers:
  - name: {{ .Values.container.name }}
  image: {{ .Values.container.image }}
  port: {{ .Values.container.port }}
```

Сам текст значений будет находиться в отдельном файле:

```
name: some-app
container:
  name: some-app-container
  image: some-app-image
  port: 1991
```
</b></details>

<details>
<summary>Какие сценарии могут быть использованы с файлом шаблона Helm?</summary><br><b>

* Развертывание одного и того же приложения в нескольких разных средах.
* CI/CD.
</b></details>

<details>
<summary>Объясните структуру каталога Helm Chart</summary><br><b>

someChart/     -> название чарта
  Chart.yaml   -> метаинформация о chart
  values.yaml  -> значения для файлов шаблона
  charts/      -> зависимости чартов
  templates/   -> файлы шаблонов :)
</b></details>

<details>
<summary>Как Helm поддерживает управление версиями?</summary><br><b>

Helm позволяет вам обновлять, удалять и откатиться к предыдущим версиям чартов. В версии 2 Helm это было с тем, что называется "Tiller". В версии 3 это было удалено из-за соображений безопасности.
</b></details>

#### Команды

<details>
<summary>Как искать чарты?</summary><br><b>

`helm search hub [some_keyword]`
</b></details>

<details>
<summary>Можно ли переопределить значения из файла values.yaml при установке чарт?</summary><br><b>
Да. Вы можете передать другой файл значений:
`helm install --values=override-values.yaml [CHART_NAME]`

Или напрямую в командной строке: `helm install --set some_key=some_value`
</b></details>

<details>
<summary>Как вывести список развернутых релизов?</summary><br><b>

`helm ls` или `helm list`
</b></details>

<details>
<summary>Как выполнить откат?</summary><br><b>

`helm rollback RELEASE_NAME REVISION_ID`
</b></details>

<details>
<summary>Как просмотреть историю ревизий для определенного релиза?</summary><br><b>

`helm history RELEASE_NAME`
</b></details>

<details>
<summary>Как обновить релиз?</summary><br><b>

`helm upgrade RELEASE_NAME CHART_NAME`
</b></details>

