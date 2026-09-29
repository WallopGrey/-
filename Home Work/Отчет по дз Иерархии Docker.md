Составить иерархию 
Задача: развернуть несколько изолированных Docker-контейнеров, но открыть к ним доступ между собой для нескольких Есть основная база данных (это один контейнер), а есть его реплика (это второй контейнер). Между ними должна быть связь (не должны быть изолированы друг от друга), мастер пишет в реплику. (не нужно скачивать образы, просто используем названия и все). + несколько контейнеров дополнительно (Proxy (изолирован и доступен только FrontEnd'у), FrontEnd (доступен только BackEnd'у) и BackEnd (доступен только для Master Базы Данных (БД))). Схема в Draw.io (как ходят запросы и куда (как ходят запросы от клиента))

ВЫВОД: Построена и проверена иерархия из пяти Docker-контейнеров: Proxy, FrontEnd, BackEnd, Master DB и Replica DB. Для ограничения были созданы четыре отдельные bridge-сети, соединяющие только необходимые уровни архитектуры. FrontEnd, BackEnd и Master DB были подключены сразу к двум соседним сетям, благодаря чему они обеспечивают последовательное прохождение запросов между уровнями, но не предоставляют прямого доступа к удалённым контейнерам. С помощью `docker inspect` была проверена конфигурация сетей, а с помощью `docker exec ... getent hosts` подтверждены разрешённые и запрещённые соединения. Таким образом, была продемонстрирована сетевая изоляция контейнеров и реализация заданной иерархии взаимодействия.

ОТЧЕТ 

Необходимо было создать несколько изолированных контейнеров и разрешить между ними только необходимые соединения.

![[Pasted image 20260926144905.png]]

## Проверка Docker

Сначала была проверена установленная версия Docker:

```
docker --version
```

Получен результат:

```
Docker version 29.8.0
```

Для просмотра существующих контейнеров использовалась команда:

```
docker ps -a
```

Она показывает все контейнеры, включая остановленные.

Для просмотра локальных Docker-образов использовалась команда:

```
docker images
```

Было обнаружено несколько уже существующих образов, поэтому новые образы для лабораторной работы не скачивались.
## Создание Docker-сетей

Для разделения контейнеров были созданы четыре отдельные bridge-сети:

```
docker network create proxy-frontend
docker network create frontend-backend
docker network create backend-master
docker network create db-replication
```

Команда `docker network create` создаёт пользовательскую Docker-сеть.

Назначение сетей:

```
proxy-frontend
    Proxy ↔ FrontEnd

frontend-backend
    FrontEnd ↔ BackEnd

backend-master
    BackEnd ↔ Master DB

db-replication
    Master DB ↔ Replica DB
```

Наличие сетей проверялось командой:

```
docker network ls
```
## Создание контейнера Proxy

Контейнер Proxy был создан командой:

```
docker run -d \
  --name proxy \
  --network proxy-frontend \
  resume-backend:latest \
  tail -f /dev/null
```

Здесь:

- `docker run` — создаёт и запускает контейнер;
- `-d` — запускает контейнер в фоновом режиме;
- `--name proxy` — задаёт имя контейнера;
- `--network proxy-frontend` — подключает контейнер к нужной сети;
- `resume-backend:latest` — использованный локальный Docker-образ;
- `tail -f /dev/null` — оставляет контейнер запущенным для проведения сетевых тестов.
## Создание FrontEnd

FrontEnd был создан и подключён к первой сети:

```
docker run -d \
  --name frontend \
  --network proxy-frontend \
  resume-backend:latest \
  tail -f /dev/null
```

Затем он был подключён ко второй сети:

```
docker network connect frontend-backend frontend
```

В результате FrontEnd находится одновременно в двух сетях:

```
proxy-frontend
frontend-backend
```

Таким образом, он может связывать Proxy и BackEnd.
## Создание BackEnd

BackEnd был создан в сети `frontend-backend`:

```
docker run -d \
  --name backend \
  --network frontend-backend \
  resume-backend:latest \
  tail -f /dev/null
```

После этого была добавлена вторая сеть:

```
docker network connect backend-master backend
```

BackEnd в результате находится в:

```
frontend-backend
backend-master
```
## Создание Master DB

Контейнер основной базы был создан:

```
docker run -d \
  --name master-db \
  --network backend-master \
  resume-backend:latest \
  tail -f /dev/null
```

Затем Master DB был подключён к сети репликации:

```
docker network connect db-replication master-db
```

В результате Master DB находится в:

```
backend-master
db-replication
```
## Создание Replica DB

Реплика была создана только в сети `db-replication`:

```
docker run -d \
  --name replica-db \
  --network db-replication \
  resume-backend:latest \
  tail -f /dev/null
```

Таким образом, Replica DB не имеет сетей, связывающих её напрямую с Proxy, FrontEnd или BackEnd.
## Проверка сетевой конфигурации

Для проверки сетей контейнера использовалась команда:

```
docker inspect frontend --format '{{json .NetworkSettings.Networks}}'
```

Аналогично проверялись:

```
docker inspect backend --format '{{json .NetworkSettings.Networks}}'
docker inspect master-db --format '{{json .NetworkSettings.Networks}}'
docker inspect replica-db --format '{{json .NetworkSettings.Networks}}'
```

Команда `docker inspect` позволяет получить подробную информацию о контейнере.

Параметр:

```
.NetworkSettings.Networks
```

показывает сети, к которым подключён контейнер, а также его IP-адреса.
## Проверка разрешённых соединений

Для проверки взаимодействия использовалась команда:

```
docker exec proxy getent hosts frontend
```

`docker exec` выполняет указанную команду внутри работающего контейнера.

`getent hosts` проверяет разрешение имени контейнера через Docker DNS.

Были проверены следующие соединения:

```
docker exec proxy getent hosts frontend
docker exec frontend getent hosts backend
docker exec backend getent hosts master-db
docker exec master-db getent hosts replica-db
```

Все четыре соединения успешно разрешались:

```
Proxy → FrontEnd       ✅
FrontEnd → BackEnd     ✅
BackEnd → Master DB    ✅
Master DB → Replica DB ✅
```
## Проверка изоляции

Затем были проверены соединения, которые по архитектуре должны быть запрещены:

```
docker exec proxy getent hosts backend
docker exec frontend getent hosts master-db
docker exec backend getent hosts replica-db
```

Все команды вернули пустой результат.

Это означает, что:

```
Proxy → BackEnd        ❌
FrontEnd → Master DB   ❌
BackEnd → Replica DB   ❌
```

Контейнеры не имеют общей пользовательской Docker-сети, поэтому Docker DNS не предоставляет им прямого разрешения имён друг друга.


# Набор команд - Docker

### Проверка Docker

```
docker --version
```

Проверили установленную версию Docker.

```
docker ps -a
```

Посмотрели все контейнеры.

```
docker images
```

Посмотрели локальные образы.

---

### Создание сетей

```
docker network create proxy-frontend
```

Создали сеть между Proxy и FrontEnd.

```
docker network create frontend-backend
```

Создали сеть между FrontEnd и BackEnd.

```
docker network create backend-master
```

Создали сеть между BackEnd и Master DB.

```
docker network create db-replication
```

Создали сеть между Master DB и Replica DB.

```
docker network ls
```

Проверили созданные сети.

---

### Создание и подключение контейнеров

Использовали:

```
docker run ...
```

для создания контейнеров:

```
proxy
frontend
backend
master-db
replica-db
```

А для подключения уже существующего контейнера к дополнительной сети:

```
docker network connect frontend-backend frontend
```

FrontEnd получил вторую сеть.

```
docker network connect backend-master backend
```

BackEnd получил вторую сеть.

```
docker network connect db-replication master-db
```

Master DB получил вторую сеть.

---

### Проверка конфигурации

```
docker inspect frontend --format '{{json .NetworkSettings.Networks}}'
```

Посмотрели, к каким сетям подключён FrontEnd и какой у него IP.

То же самое сделали для:

```
docker inspect backend --format '{{json .NetworkSettings.Networks}}'
docker inspect master-db --format '{{json .NetworkSettings.Networks}}'
docker inspect replica-db --format '{{json .NetworkSettings.Networks}}'
```

---

### Проверка разрешённых соединений

```
docker exec proxy getent hosts frontend
```

Проверили:

```
Proxy → FrontEnd
```

```
docker exec frontend getent hosts backend
```

Проверили:

```
FrontEnd → BackEnd
```

```
docker exec backend getent hosts master-db
```

Проверили:

```
BackEnd → Master DB
```

```
docker exec master-db getent hosts replica-db
```

Проверили:

```
Master DB → Replica DB
```

Во всех случаях Docker вернул IP-адрес.

---

### Проверка изоляции

```
docker exec proxy getent hosts backend
```

```
docker exec frontend getent hosts master-db
```

```
docker exec backend getent hosts replica-db
```

Результат был пустым.

То есть прямой связи через Docker DNS между этими уровнями нет.