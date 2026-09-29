1)Отменить последний commit  
Ситуация:  
Кто-то из разработчиков запушил репозиторий чувствительные данные (пароли, персональная информация)  
Задача:  
удалить commit из git истории, добавить файл с паролем в git-ignore и запушить правильную версию

ВЫВОД: Были получены навыки управления историей Git и исключения чувствительных данных из отслеживания. Был добавлен файл с паролем в commit. С помощью команды `git reset --soft HEAD~1` ошибочный commit был удалён из текущей истории с сохранением изменений. После этого файл `password.txt` был добавлен в `.gitignore` и удалён из staging area. Проверка с помощью `git check-ignore` и `git ls-files` подтвердила, что файл больше не отслеживается Git. В результате была получена корректная история репозитория без ошибочного commit и настроено предотвращение повторного добавления файла с паролем.

ОТЧЕТ
необходимо было смоделировать ситуацию, при которой разработчик случайно добавил чувствительные данные в Git, удалить ошибочный commit из локальной истории и настроить `.gitignore`.
Сначала был создан каталог проекта:

```
mkdir ~/git-security-test
```
Затем был выполнен переход в него, после этого был инициализирован Git-репозиторий:

```
git init
```

Команда `git init` создаёт в текущем каталоге скрытый каталог `.git`, в котором Git хранит историю, настройки и служебную информацию репозитория.
## Настройка пользователя Git

Были заданы имя и электронная почта:

```
git config --global user.name "WallopGrey"
git config --global user.email "Alexprokati777@gmail.com"
```

Параметр `--global` означает, что настройки применяются ко всем Git-репозиториям данного пользователя.

Для проверки настроек использовалась команда:

```
git config --global --list
```

В результате были получены:

```
user.name=WallopGrey
user.email=Alexprokati777@gmail.com
```
## Создание первоначального commit

Был создан файл `config.txt`:

```
APP_NAME=MyApplication
PORT=8080
```

После этого файл был добавлен в индекс:

```
git add config.txt
```

Команда `git add` помещает изменения в staging area - область подготовки следующего commit.

Первый commit был создан командой:

```
git commit -m "Initial commit"
```

Ключ `-m` позволяет сразу указать сообщение commit.

Первоначальная история имела вид:

```
b4a2607 (HEAD -> main) Initial commit
```

Ветка была переименована в `main`:

```
git branch -m main
```
## Имитация ошибочного commit

Для моделирования ситуации с утечкой чувствительных данных был создан файл:

```
password.txt
```

Содержимое файла:

```
DB_PASSWORD=SuperSecret123
```

Затем по ошибке был добавлен весь каталог:

```
git add .
```

После чего был создан ошибочный commit:

```
git commit -m "Add database configuration"
```

В результате история стала:

```
722b6f9 (HEAD -> main) Add database configuration
b4a2607 Initial commit
```

Commit `722b6f9` являлся ошибочным, поскольку в нём находился файл `password.txt` с чувствительными данными.
## Удаление ошибочного commit

Для удаления последнего commit была использована команда:

```
git reset --soft HEAD~1
```

`HEAD~1` означает предыдущий commit относительно текущего.

Параметр `--soft` удаляет commit из истории, но сохраняет изменения в staging area.

После выполнения команды история стала:

```
b4a2607 (HEAD -> main) Initial commit
```

Таким образом, ошибочный commit `722b6f9` исчез из текущей истории, однако файл `password.txt` остался подготовленным к commit.
## Создание `.gitignore`

Для предотвращения повторного добавления файла с паролем был создан файл:

```
.gitignore
```

В него была добавлена строка:

```
password.txt
```

Файл `.gitignore` содержит правила для файлов и каталогов, которые Git не должен отслеживать.

После этого файл `password.txt` был удалён из staging area:

```
git restore --staged password.txt
```

Команда `git restore --staged` убирает файл из индекса, не удаляя сам файл с диска.

Проверка правила `.gitignore` выполнялась командой:

```
git check-ignore -v password.txt
```

Результат:

```
.gitignore:1:password.txt password.txt
```

Это подтверждало, что файл действительно игнорируется Git.
## Создание исправленного commit

Файл `.gitignore` был добавлен в индекс:

```
git add .gitignore
```

После этого был создан новый commit:

```
git commit -m "Add gitignore for sensitive files"
```

Итоговая история:

```
785426f (HEAD -> main) Add gitignore for sensitive files
b4a2607 Initial commit
```

Ошибочного commit `722b6f9` в текущей истории больше нет.
## Проверка результата

Состояние рабочего каталога проверялось:

```
git status
```

Результат:

```
нечего коммитить, нет изменений в рабочем каталоге
```

Список отслеживаемых Git файлов проверялся:

```
git ls-files
```

Результат:

```
.gitignore
config.txt
```

Файл `password.txt` отсутствовал в списке отслеживаемых файлов.

При этом сам файл продолжал существовать на диске:

```
ls -la
```

и его содержимое можно было посмотреть:

```
cat password.txt
```

Таким образом, файл с чувствительными данными физически сохранился в рабочем каталоге, но Git больше его не отслеживал.

# Набор команд  - Git



### Создание проекта

```
mkdir ~/git-security-test
```

Создали каталог проекта.

```
cd ~/git-security-test
```

Перешли в него.

```
git init
```

Создали Git-репозиторий.

---

### Настройка Git

```
git config --global user.name "WallopGrey"
```

Задали имя автора commit.

```
git config --global user.email "Alexprokati777@gmail.com"
```

Задали email автора.

```
git config --global --list
```

Проверили настройки.

---

### Первый commit

```
git add config.txt
```

Добавили `config.txt` в staging area.

```
git commit -m "Initial commit"
```

Создали первый commit.

```
git branch -m main
```

Переименовали ветку в `main`.

```
git log --oneline
```

Посмотрели историю commit в кратком виде.

---

### Ошибочный commit

```
git add .
```

Добавили все изменения, включая ошибочный `password.txt`.

```
git commit -m "Add database configuration"
```

Создали commit, в котором оказался пароль.

```
git log --oneline
```

Проверили, что ошибочный commit появился в истории.

---

### Исправление

```
git reset --soft HEAD~1
```

Удалили последний commit из истории, **сохранив его изменения в staging area**.

```
git status
```

Проверили состояние репозитория.

```
nano .gitignore
```

Создали/отредактировали `.gitignore`.

В него записали:

```
password.txt
```

```
git restore --staged password.txt
```

Убрали пароль из staging area.

```
git check-ignore -v password.txt
```

Проверили, что `password.txt` действительно игнорируется.

```
git add .gitignore
```

Добавили `.gitignore`.

```
git commit -m "Add gitignore for sensitive files"
```

Создали исправленный commit.

```
git status
```

Проверили, что рабочее дерево чистое.

```
git ls-files
```

Проверили список файлов, которые Git отслеживает.

Результат:

```
.gitignore
config.txt
```

`password.txt` отсутствует.

