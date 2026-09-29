# Инструкция по добавлению SSH-ключа в GitHub

## 1. Проверка наличия SSH-ключа

Открой терминал Linux и выполни:

```bash
ls -la ~/.ssh
```

Если файлов `id_ed25519` и `id_ed25519.pub` нет, создай новый ключ:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

На вопрос о расположении файла можно нажать **Enter**, чтобы использовать стандартный путь:

```text
/home/username/.ssh/id_ed25519
```

При необходимости можно установить парольную фразу для защиты ключа.

---

## 2. Запуск SSH-agent

Запусти SSH-agent:

```bash
eval "$(ssh-agent -s)"
```

Добавь приватный ключ в агент:

```bash
ssh-add ~/.ssh/id_ed25519
```

Проверить добавленный ключ можно командой:

```bash
ssh-add -l
```

---

## 3. Получение публичного ключа

Выведи содержимое публичного ключа:

```bash
cat ~/.ssh/id_ed25519.pub
```

Получится строка примерно такого вида:

```text
ssh-ed25519 AAAAC3... your_email@example.com
```

**Копировать нужно всю строку целиком.**

> Важно: файл `id_ed25519` — это приватный ключ. Его нельзя никому отправлять или публиковать. Файл `id_ed25519.pub` — публичный ключ, его можно добавлять в GitHub.

---

## 4. Добавление ключа в GitHub

Открой GitHub и перейди:

**Profile → Settings → SSH and GPG keys → New SSH key**

В поле **Title** укажи название устройства, например:

```text
SunShine Debian
```

В поле **Key** вставь содержимое:

```bash
cat ~/.ssh/id_ed25519.pub
```

Затем нажми **Add SSH key**.

---

## 5. Проверка подключения

В терминале выполни:

```bash
ssh -T git@github.com
```

При первом подключении GitHub может спросить:

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Введи:

```text
yes
```

При успешной настройке GitHub сообщит, что аутентификация прошла успешно.

---

## 6. Проверка SSH-адреса репозитория

Перейди в каталог проекта:

```bash
cd ~/ResumeProject
```

Проверь адрес удалённого репозитория:

```bash
git remote -v
```

Для SSH он должен выглядеть примерно так:

```text
origin  git@github.com:USERNAME/REPOSITORY.git (fetch)
origin  git@github.com:USERNAME/REPOSITORY.git (push)
```

Если используется HTTPS, можно заменить адрес на SSH:

```bash
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```

После этого:

```bash
git pull
```

и:

```bash
git push
```

Git будет использовать SSH-аутентификацию вместо ввода логина и пароля GitHub.

## 7. Проверка

Для окончательной проверки можно выполнить:

```bash
ssh -T git@github.com
git pull
git status
```

Если `git pull` выполняется без запроса логина и пароля, SSH-подключение настроено.