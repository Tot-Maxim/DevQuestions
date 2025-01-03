### [Назад к оглавлению](../../README.md)

## [Git](https://git-scm.com/docs/git)

## Оглавление
1. [Основы Git](#основы-git)
2. [Сценарии](#сценарии)
3. [Ветки](#ветки)
4. [Слияние](#слияние)
5. [Перенос](#перенос)
6. [Общие вопросы](#общие-вопросы)
7. [Ссылки](#ссылки)
8. [Git Diff](#git-diff)
9. [Внутренности Git](#внутренности-git)

## Основы Git

<details>
<summary>Как узнать, является ли определенная директория репозиторием git?</summary><br><b>
Вы можете проверить наличие директории ".git".
</b></details>

<details>
<summary>Объясните следующее: <code>git directory</code>, <code>working directory</code> и <code>staging area</code></summary><br>
<b>

Этот ответ взят из [git-scm.com](https://git-scm.com/book/en/v1/Getting-Started-Git-Basics#_the_three_states)

"Git-репозиторий — это место, где Git хранит метаданные и базу объектов для вашего проекта. Это самая важная часть Git, и именно она копируется, когда вы клонируете репозиторий с другого компьютера.

Рабочая директория — это одна версия проекта, которую вы извлекли из базы данных в Git-репозитории. Эти файлы извлекаются из сжатой базы данных в директории Git и помещаются на диск для вашего использования или изменения.

Область подготовки — это простой файл, который обычно находится в вашем Git-репозитории и хранит информацию о том, что будет включено в ваш следующий коммит. Иногда его называют индексом, но становится стандартом называть его областью подготовки."
</b></details>

<details>
<summary>В чем разница между <code>git pull</code> и <code>git fetch</code>?</summary><br><b>

Кратко, git pull = git fetch + git merge.

Когда вы запускаете git pull, он получает все изменения из удаленного или центрального репозитория и прикрепляет их к соответствующей ветке в вашем локальном репозитории.

git fetch получает все изменения из удаленного репозитория, хранит их в отдельной ветке в вашем локальном репозитории.
</b></details>

<details>
<summary>Как проверить, отслеживается ли файл и, если нет, то начать его отслеживание?</summary><br><b>

Существует несколько способов проверить, отслеживается ли файл или нет:
  - `git ls-files <file>` -> код выхода 0 означает, что он отслеживается
  - `git blame <file>`
  ...
</b></details>

<details>
<summary>Объясните, для чего используется файл <code>gitignore</code></summary><br><b>
Цель файлов <code>gitignore</code> — убедиться, что определенные файлы, не отслеживаемые Git, остаются вне отслеживания. Чтобы перестать отслеживать файл, который в настоящее время отслеживается, используйте команду git rm --cached.
</b></details>

<details>
<summary>Как вы можете увидеть, какие изменения были внесены до их коммита?</summary><br><b>
`git diff`
</b></details>

<details>
<summary>Что делает <code>git status</code>?</summary><br><b>

`git status` помогает вам понять статус отслеживания файлов в вашем репозитории. Скрупулезно по рабочей директории и области подготовки — вы можете узнать, какие изменения были внесены в рабочую директорию, какие изменения находятся в области подготовки и, в общем, отслеживаются ли файлы или нет.
</b></details>

<details>
<summary>Вы создали новые файлы в своем репозитории. Как убедиться, что Git отслеживает их?</summary><br><b>

`git add FILES`
</b></details>

## Сценарии

<details>
<summary>У вас есть файлы в вашем репозитории, которые вы не хотите, чтобы Git когда-либо отслеживал. Что вам следует сделать, чтобы избежать их отслеживания?</summary><br><b>

Добавьте их в файл `.gitignore`. Это гарантирует, что эти файлы никогда не будут добавлены в область подготовки.
</b></details>

<details>
<summary>В вашей организации команда разработчиков использует монорепозиторий, и он стал довольно большим, включая сотни тысяч файлов. Они говорят, что выполнение многих операций git занимает много времени (например, git status). Почему это происходит и что вы можете сделать, чтобы помочь им?</summary><br><b>

Многие операции Git связаны с состоянием файловой системы. Например, `git status` будет выполнять сравнения для сопоставления коммита HEAD с индексом и другого сравнения для сопоставления индекса с рабочей директорией. Как часть этих сравнений он будет выполнять довольно много вызовов `lstat()`. При работе с сотнями тысяч файлов это может занять секунды, если не минуты.

Одним из решений было бы использовать встроенный `fsmonitor` (монитор файловой системы) Git. С fsmonitor (который интегрируется с Watchman) Git запускает демон, который будет постоянно следить за любыми изменениями в рабочей директории вашего репозитория и кэшировать их. Таким образом, когда вы запускаете `git status`, вместо сканирования рабочей директории вы используете кэшированное состояние вашего индекса.

<p align="center">
<img src="images/design/development/git_fsmonitor.png"/>
</p>

Далее вы можете попробовать включить `feature.manyFile` с помощью `git config feature.manyFiles true`. Это делает две вещи:

1. Устанавливает `index.version = 4`, что включает сжатие префикса пути в индексе.
2. Устанавливает `core.untrackedCache=true`, которое по умолчанию установлено на `keep`. Ненаблюдаемый кэш является весьма важной концепцией. Он записывает время модификации всех файлов и директорий в рабочей директории. Таким образом, когда приходит время перебрать все файлы и директорий, он может пропустить те, у которых время модификации не было обновлено.

Перед тем как включить его, вы можете выполнить `git update-index --test-untracked-cache`, чтобы протестировать его и убедиться, что время модификации актуально в вашей системе.

У Git также есть встроенная команда `git-maintainence`, которая оптимизирует репозиторий Git, чтобы команды, такие как `git add` или `git fetch`, работали быстрее, и сам репозиторий занимал меньше места на диске. Рекомендуется периодически запускать эту команду (например, каждый день).

Кроме того, отслеживайте только то, что используется/изменяется разработчиками - в некоторых репозиториях могут быть сгенерированные файлы, которые необходимы для нормальной работы проекта (или поддерживают определенные опции доступности), но не изменяются разработчиками. В таком случае их отслеживать нецелесообразно.
Чтобы избежать заполнения этих файлов в рабочей директории, можно использовать функцию `sparse checkout` Git.

Наконец, с определенными системами сборки вы можете знать, какие файлы используются/имеют отношение к конкретному компоненту проекта, над которым работает разработчик. Это, в сочетании с `sparse checkout`, может привести к ситуации, когда только небольшая часть файлов заполняется в рабочей директории. Это делает команды, такие как `git add`, `git status`, и т. д. действительно быстрыми.
</b></details>

## Ветки

<details>
<summary>Какова стратегия ветвления (flow), которую вы знаете?</summary><br><b>

- Git flow
- GitHub flow
- Разработка на основе trunk
- GitLab flow

[Объяснение](https://www.bmc.com/blogs/devops-branching-strategies/#:~:text=What%20is%20a%20branching%20strategy,used%20in%20the%20development%20process).
</b></details>

<details>
<summary>Истинно или ложно? Ветка — это в основном простой указатель или ссылка на вершину определенной ветки разработки.</summary><br><b>

Истинно.
</b></details>

<details>
<summary>У вас есть две ветки - main и devel. Как убедиться, что devel синхронизирована с main?</summary><br><b>
<code>
```
git checkout main
git pull
git checkout devel
git merge main
```
</code>
</b></details>

<details>
<summary>Кратко опишите, что происходит за кулисами, когда вы запускаете <code>git branch <BRANCH></code></summary><br><b>

Git выполняет update-ref, чтобы добавить SHA-1 последнего коммита ветки, над которой вы находитесь, в новую ветку, которую вы хотите создать.
</b></details>

<details>
<summary>Когда вы запускаете <code>git branch <BRANCH></code>, как Git знает SHA-1 последнего коммита?</summary><br><b>

Используя файл HEAD: `.git/HEAD`.
</b></details>

<details>
<summary>Что означает <code>unstaged</code> в контексте Git?</summary><br><b>

Файл, который находится в рабочей директории, но не находится в HEAD и не в области подготовки, называется "unstaged".
</b></details>

<details>
<summary>Истинно или ложно? Когда вы <code>git checkout some_branch</code>, Git обновляет .git/HEAD на <code>/refs/heads/some_branch</code></summary><br><b>

Истинно.
</b></details>

## Слияние

<details>
<summary>У вас есть две ветки - main и devel. Как вы объединяете devel в main?</summary><br><b>

```
git checkout main
git merge devel
git push origin main
```
</b></details>

<details>
<summary>Как разрешить конфликты при слиянии git?</summary><br><b>

<p>
Сначала вы открываете файлы, которые находятся в конфликте, и определяете, в чем заключаются конфликты.
Затем, в зависимости от того, что принято в вашей компании или команде, вы либо обсуждаете конфликты с коллегами, либо разрешаете их самостоятельно.
После разрешения конфликтов, вы добавляете файлы с помощью `git add <file_name>`.
Наконец, вы выполняете `git rebase --continue`.
</p>
</b></details>

<details>
<summary>С какими стратегиями слияния вы знакомы?</summary><br><b>

Достаточно упомянуть две или три, и, вероятно, хорошо сказать, что "recursive" является стратегией по умолчанию.

recursive
resolve
ours
theirs

Эта страница объясняет это лучше всего: https://git-scm.com/docs/merge-strategies
</b></details>

<details>
<summary>Объясните слияние Octopus в Git</summary><br><b>

Вероятно, хорошо упомянуть, что это:

- Отлично подходит для случаев слияния более чем одной ветки (и также по умолчанию для таких случаев)
- Предназначено в основном для объединения тематических веток вместе

Это отличная статья о слиянии Octopus: http://www.freblogg.com/2016/12/git-octopus-merge.html
</b></details>

<details>
<summary>В чем разница между <code>git reset</code> и <code>git revert</code>?</summary><br><b>

<p>

`git revert` создает новый коммит, который отменяет изменения из последнего коммита.

`git reset` в зависимости от использования может изменить индекс или изменить коммит, на который указывает HEAD ветки.
</p>
</b></details>

## Перенос

<details>
<summary>Вы хотите переместить коммит вперед на вершину. Как вы этого добьетесь?</summary><br><b>

С помощью команды `git rebase`.
</b></details>

<details>
<summary>В каких ситуациях вы используете <code>git rebase</code>?</summary><br><b>
Предположим, команда работает над веткой `feature`, которая происходит от основной ветки репозитория. В момент, когда разработка функции завершена, и мы, наконец, желаем объединить ветку функции в основную ветку, не сохраняя историю коммитов, сделанных в ветке функции, `git rebase` будет полезен. 
</b></details>

<details>
<summary>Как вернуть конкретный файл к предыдущему коммиту?</summary><br><b>

```
git checkout HEAD~1 -- /path/of/the/file
```
</b></details>

<details>
<summary>Как объединить последние два коммита?</summary><br><b>
</b></details>

<details>
<summary>Что такое директория <code>.git</code>? Что вы можете там найти?</summary><br><b>
	Директория <code>.git</code> содержит всю информацию, необходимую для контроля версий вашего проекта, а также всю информацию о коммитах, адресах удаленных репозиториев и т. д. Все они присутствуют в этой папке. Она также содержит журнал, который хранит вашу историю коммитов, чтобы вы могли вернуться к предыдущим версиям.

Эта информация скопирована с [https://stackoverflow.com/questions/29217859/what-is-the-git-folder](https://stackoverflow.com/questions/29217859/what-is-the-git-folder).
</b></details>

<details>
<summary>Каковы некоторые антипаттерны Git? То, что следует избегать</summary><br><b>

- Не ждать слишком долго между коммитами
- Не удалять директорию .git :)
</b></details>

<details>
<summary>Как удалить удаленную ветку?</summary><br><b>

Вы удаляете удаленную ветку с помощью следующего синтаксиса:

git push origin :[branch_name]
</b></details>

<details>
<summary>Знакомы ли вы с gitattributes? Когда вы бы их использовали?</summary><br><b>

gitattributes позволяют вам определять атрибуты по имени пути или паттерну пути.<br>

Вы можете использовать их, например, чтобы контролировать символы окончания строк в файлах. В системах Windows и Unix используются разные символы для новых строк (\r\n и \n соответственно). Используя gitattributes, мы можем согласовать это для Windows и Unix с помощью `* text=auto` в .gitattributes для всех, кто работает с git. Таким образом, если вы используете проект Git в Windows, вы получите \r\n, а если вы используете Unix или Linux, вы получите \n.
</b></details>

<details>
<summary>Как отменить изменения локального файла? (до коммита)</summary><br><b>

`git checkout -- <file_name>`
</b></details>

<details>
<summary>Как отменить локальные коммиты?</summary><br><b>

`git reset HEAD~1` для удаления последнего коммита.
Если вы также хотите отменить изменения, то используйте `git reset --hard`.
</b></details>

<details>
<summary>Истинно или ложно? Чтобы удалить файл из git, но не из файловой системы, следует использовать <code>git rm </code></summary><br><b>

Ложно. Если вы хотите сохранить файл в файловой системе, используйте `git reset <file_name>`.
</b></details>

## Ссылки

<details>
<summary>Как перечислить текущие ссылки git в данном репозитории? </summary><br><b>

`find .git/refs/`
</b></details>

## Git Diff

<details>
<summary>Что делает git diff?</summary><br><b>

git diff может сравнивать два коммита, два файла, дерево и область подготовки и т. д.
</b></details>

<details>
<summary>Что быстрее? <code>git diff-index HEAD</code> или <code>git diff HEAD</code> </summary><br><b>

`git diff-index` быстрее, но, чтобы быть честным, это потому, что он делает меньше. `git diff index` не будет смотреть на содержимое, а только на метаданные, такие как метки времени.
</b></details>

<details>
<summary>Какие другие команды Git использует git diff?</summary><br><b>

Механизм разницы используется командой `git status`, чтобы произвести сравнение и сообщить пользователю, какие файлы отслеживаются.
</b></details>

## Внутренности Git

<details>
<summary>Опишите, как работает <code>git status</code></summary><br><b>

Кратко, он выполняет `git diff` дважды:

1. Сравнение между HEAD и областью подготовки
2. Сравнение области подготовки и рабочей директории
   </b></details>

<details>
<summary>Если <code>git status</code> должен выполнить разницу по всем файлам в коммите HEAD с теми, что в области подготовки/индексе, а также другую разницу между областью подготовки/индексом и рабочей директорией, как это выполняется довольно быстро? </summary><br><b>

Одна из причин заключается в структуре индекса, коммитов и т. д.

- Каждый файл в коммите хранится в объекте дерева.
- Индекс затем представляет собой упрощенную структуру объектов дерева.
- Все файлы в индексе имеют заранее рассчитанные хеши.
- Операция разности, таким образом, сравнивает хеши.

Еще одной причиной является кэширование.

- Индекс кэширует информацию о рабочей директории.
- Когда Git имеет информацию о конкретном файле в кэше, нет необходимости заглядывать в файл рабочей директории.
</b></details>

### [Назад к оглавлению](../../README.md)