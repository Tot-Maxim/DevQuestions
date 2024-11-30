### [Назад к оглавлению](/DevQuestions/README.md)

## Shell Scripting

### Shell Scripting Exercises


## Скриптование в оболочке - Самооценка

<details>
<summary>Что означает эта строка в скриптах оболочки?: <code>#!/bin/bash</code></summary><br><b>


`#!/bin/bash`  — это шебанг (shebang)

/bin/bash — это наиболее распространенная оболочка, используемая в качестве оболочки по умолчанию для входа пользователя в систему Linux. Название оболочки — это акроним для Bourne-again shell. Bash может выполнять подавляющее большинство скриптов и поэтому широко используется, поскольку обладает большим набором возможностей, хорошо разработан и имеет лучшую синтаксис.

</b></details>

<details>
<summary>Правда или ложь? Когда определенная команда/строка завершается неудачей в скрипте оболочки, скрипт оболочки по умолчанию завершится и перестанет выполняться</summary><br><b>

Зависит от языка и настроек.
Если скрипт написан на Bash, то это утверждение истинно. Когда скрипт, написанный на Bash, не удается выполнить определенную команду, он продолжает работать и выполнит все остальные команды, указанные после той команды, которая завершилась неудачей.

Чаще всего мы хотим, чтобы произошло наоборот. Чтобы Bash завершался, когда определенная команда не удалась, используйте 'set -e' в вашем скрипте.
</b></details>

<details>
<summary>Что вы часто включаете в каждый скрипт, который пишете?</summary><br><b>

Несколько примеров:

Комментарии о том, как его запустить и/или что он делает
Если это скрипт оболочки, добавление "set -e", так как я хочу, чтобы скрипт завершался, если определенная команда не удалась
У вас может быть совершенно другой ответ. Это зависит только от вашего опыта и предпочтений.
</b></details>

<details>
<summary>Сегодня у нас есть такие инструменты и технологии, как Ansible, Puppet, Chef ... Зачем кому-то все еще использовать скриптование в оболочке?</summary><br><b>

 * Скорость
 * Гибкость
 * Модуль, который нам нужен, не существует (возможно, это слабая точка, так как большинство технологий управления конфигурацией позволяет использовать так называемый "shell" модуль)
 * Мы передаем скрипты клиентам, которые не имеют доступа к публичной сети и не обязательно имеют Ansible установленным на своих системах.
</b></details>

#### Скриптование в оболочке - Переменные

<details>
<summary>Как определить переменную со значением "Hello World"?</summary><br><b>

`HW="Hello World`
</b>
</details>

<details>
<summary>Как определить переменную со значением текущей даты?</summary><br><b>

`DATE=$(date)`
</b>
</details>

<details>
<summary>Как напечатать первый аргумент, переданный скрипту?</summary><br><b>

`echo $1`

</b>
</details>

<details>
<summary>Напишите скрипт, чтобы напечатать "yay", если аргумент не был передан, а затем напечатайте этот аргумент</summary><br><b>

```
echo "${1:-yay}"
```
</b>
</details>

<details>
<summary>Какой будет вывод следующего скрипта?

```
#!/usr/bin/env bash
NINJA_TURTLE=Donatello
function the_best_ninja_turtle {
        local NINJA_TURTLE=Michelangelo
        echo $NINJA_TURTLE
}
NINJA_TURTLE=Raphael
the_best_ninja_turtle
```
</summary><br><b>
Michelangelo
</b>
</details>

<details>
  <summary>Объясните, каков будет результат каждой команды:

  * <code>echo $0</code>
  * <code>echo $?</code>
  * <code>echo $$</code>
  * <code>echo $#</code>
  </summary>
  <b>Результат выполнения команд в данном скрипте:</b>
  
```
bash script.sh arg1 arg2
```
  <p>Результат выполнения команд будет следующим:</p>
  <p><code>echo $0</code> → script.sh (или bash, если запущено напрямую)
  <p><code>echo $?</code> → 0 (если предыдущий код завершился успешно)</p>
  <p><code>echo $$</code> → 12345 (PID текущего процесса, например)</p>
  <p><code>echo $#</code> → 2 (количество аргументов, переданных скрипту)</p>
</details>

<details>
<summary>Что такое <code>$@</code>?</summary><br>
<p>Важно отметить, что <code>$@</code> и <code>$*</code> — это разные вещи:</p>
<ul>
    <li><code>"$@"</code>: Каждый аргумент обрабатывается отдельно. Если вы используете <code>"$@"</code>, это означает, что каждый аргумент будет передан отдельно, сохраняя пробелы, если они есть.</li>
    <li><code>"$*"</code>: Все аргументы объединяются в одну строку. Если вы используете <code>"$*"</code>, то они будут объединены в строку, и пробелы заменятся на единственный пробел.</li>
</ul>
<pre>
#!/bin/bash
echo "All arguments: $@"
</pre>

<p>Если вы выполните этот скрипт, передав ему несколько аргументов:</p>
<pre>
bash myscript.sh arg1 "arg2 with spaces" arg3
</pre>

<p>Вывод будет:</p>
<pre>
All arguments: arg1 arg2 with spaces arg3
</pre></details>

<details>
<summary>В чем разница между <code> $ @</code> и <code> $ *</code>?</summary><br>

`$@` — это массив всех аргументов, переданных скрипту.<br> 
`$*` — это одна строка всех аргументов, переданных скрипту.
</details>

<details>
<summary>Как получить ввод от пользователя в скриптах оболочки?</summary><br><b>

Используя ключевое слово <code>read</code>, например, <code>read x</code> будет ожидать ввод пользователя и сохранит его в переменной x.
</b>
</details>

<details>
<summary>Как сравнить длину переменных?</summary><br><b>

```
if [ ${#1} -ne ${#2} ]; then
    ...
```
</b></details>

#### Скриптование в оболочке - Условные конструкции

<details>
<summary>Объясните условные конструкции и продемонстрируйте, как их использовать</summary><br><b>
</b></details>

<details>
<summary>Как в скриптах оболочки отрицать условие?</summary><br><b>
</b></details>

<details>
<summary>Как в скриптах оболочки проверить, является ли заданный аргумент числом?</summary><br><b>

```
regex='^[0-9]+$'
if [[ ${var//*.} =~ $regex ]]; then
...
```
</b></details>

#### Shell Scripting - Скриптование в оболочке - Арифметические операции

<details>
<summary>Как выполнять арифметические операции с числами?</summary><br><b>

Один из способов: `$(( 1 + 2 ))`<br>
Другой способ: `expr 1 + 2`
</b>
</details>

<details>
<summary>Как проверить, что данное число является делителем 4?</summary><br><b>

`if [ $(($1 % 4)) -eq 0 ]; then`
</b>
</details>

#### Shell Скриптование в оболочке - Циклы

<details>
<summary>Что такое цикл? С какими типами циклов вы знакомы?</summary><br><b>
</b></details>

<details>
<summary>Продемонстрируйте, как использовать циклы.</summary><br><b>
</b></details>

#### Скриптование в оболочке - Устранение неполадок

<details>
<summary>Как отлаживать скрипты оболочки?</summary><br><b>

Ответ зависит от языка, который вы используете для написания своих скриптов. Если, например, используется Bash:

  * Добавляя -x к скрипту, который я запускаю в Bash
  * Добро старый способ — добавление операторов echo

Если Python, то использование pdb очень полезно.
</b>
</details>

<details>
<summary>Почему, когда мы запускаем следующий скрипт Bash, мы не получаем 2 в результате?

```
x = 2
echo $x
```
</summary><br><b>

Должно быть `x=2`
</b>
</details>

#### Скриптование в оболочке - Подстрока

<details>
<summary>Как извлечь все после последней точки в строке?</summary><br><b>

`${var//*.}`
</b>
</details>

<details>
<summary>Как извлечь все до последней точки в строке?</summary><br><b>

`${var%.*}`
</b>
</details>

#### Скриптование в оболочке - Разное

<details>
<summary>Сгенерируйте 8-значное случайное число</summary><br><b>

`shuf -i 9999999-99999999 -n 1`
</b>
</details>

<details>
<summary>Можете привести примеры лучших практик написания Bash?</summary><br><b>
</b>
</details>

<details>
<summary>Что такое тернарный оператор? Как его использовать в Bash?</summary><br><b>

Это краткий способ использования конструкции if/else. Пример:

`[[ $a = 1 ]] && b="yes, equal" || b="nope"`
</b></details>

<details>
<summary>Что делает следующий код и когда его стоит использовать?<code>diff <(ls /tmp) <(ls /var/tmp)</code></summary><br>
Это называется 'замена процессов' (process substitution). Это позволяет передавать вывод одной команды другой команде, когда использование конвейера <code>|</code> невозможно. Это может быть полезно, когда команда не поддерживает <code>STDIN</code> или вам нужен вывод нескольких команд. https://superuser.com/a/1060002/167769 </details>
</details>

<details>
<summary>Что вы используете для тестирования скриптов оболочки?</summary><br><b>

bats
</b></details>
### [Назад к оглавлению](/DevQuestions/README.md)