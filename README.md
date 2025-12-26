# Отчёт по практической работе: Дополнительные возможности Docker

## Цель работы

Изучение дополнительных возможностей Docker для управления жизненным циклом контейнеров, мониторинга их состояния и оптимизации использования системных ресурсов.

---

## Подготовка к работе

### Создание рабочей директории и файлов

Создал директорию для практической работы и необходимые файлы:

```bash
mkdir ~/docker-practice
cd ~/docker-practice
```

<img width="1139" height="121" alt="image" src="https://github.com/user-attachments/assets/aaf11c6b-5404-46cb-bf5b-3af8ab3a4e3f" />



### Создание файла entrypoint.sh

Создал скрипт `entrypoint.sh` со следующим содержимым:

```bash
#!/bin/bash

DURATION=${1:-60}

echo "====== Docker Practice Container ======"
echo "Start time: $(date)"
echo "Duration: ${DURATION} seconds"
echo "========================================"

echo "System Information:"
uname -a
echo "Available Memory: $(free -h | grep Mem)"
echo "CPU Info:"
nproc

counter=0
while [ $counter -lt $DURATION ]; do
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Iteration $((counter + 1))/$DURATION"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Memory usage: $(ps aux | awk '{sum+=$6} END {print sum/1024 " MB"}')"
    sleep 5
    counter=$((counter + 5))
done

echo "====== Container Shutdown ======"
echo "End time: $(date)"
echo "Container completed successfully"
```

### Создание Dockerfile

Создал Dockerfile:

```dockerfile
FROM alpine:3.18

RUN apk add --no-cache \
    bash \
    curl \
    htop \
    procps

RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /home/appuser

COPY --chown=appuser:appuser entrypoint.sh /home/appuser/

RUN chmod +x /home/appuser/entrypoint.sh

USER appuser

ENTRYPOINT ["/home/appuser/entrypoint.sh"]

CMD ["60"]
```

### Сборка образа

Выполнил сборку Docker-образа:

```bash
chmod +x entrypoint.sh
docker build -t practice-image:1.0 .
```

<img width="1277" height="524" alt="image" src="https://github.com/user-attachments/assets/d09cddd7-709b-4b0e-bdb2-0f6cdc995a6a" />




Проверил созданный образ:

```bash
docker images
```

<img width="1228" height="334" alt="image" src="https://github.com/user-attachments/assets/d91d074f-c49d-4a96-9f86-d4d7aaba4f81" />




---

## Задание 1: Вывод логов контейнера в файл

### Описание задания

Запуск контейнера и сохранение его логов в файл для последующего анализа.

### Выполнение

Запустил контейнер на 30 секунд:

```bash
docker run -d --name practice-container-1 practice-image:1.0 30
```

<img width="1482" height="50" alt="image" src="https://github.com/user-attachments/assets/468b4d84-576c-46bc-8582-bac0ebb336a9" />

Просмотрел логи контейнера в реальном времени:

```bash
docker logs -f practice-container-1
```

<img width="1426" height="550" alt="image" src="https://github.com/user-attachments/assets/f01be881-c1f9-4fb2-96e6-356cdb909fe3" />

Проверил статус завершения контейнера:

```bash
docker ps -a | grep practice-container-1
```

<img width="1280" height="49" alt="image" src="https://github.com/user-attachments/assets/e0839e7b-5b1f-4f4c-a170-6f9829368764" />




Сохранил логи в файл:

```bash
docker logs practice-container-1 > ~/docker-practice/container_logs.txt
cat ~/docker-practice/container_logs.txt
```
<img width="1350" height="108" alt="image" src="https://github.com/user-attachments/assets/df3998aa-a3d1-4716-b5ca-3e56f6921d3d" />


и Далее сохранил логи с временными метками и удалил контейнер

### Результат

✅ Успешно сохранены логи контейнера в файлы для дальнейшего анализа

---

## Задание 2: Проверка docker-stats

### Описание задания

Мониторинг потребления системных ресурсов контейнером в реальном времени.

### Выполнение

Запустил контейнер на 45 секунд:

```bash
docker run -d --name practice-container-2 practice-image:1.0 45
```

Просмотрел статистику в реальном времени:

```bash
docker stats practice-container-2
```


Сделал снимок статистики:

```bash
docker stats --no-stream practice-container-2
```

<img width="1434" height="50" alt="image" src="https://github.com/user-attachments/assets/07cff6a6-bedd-45e0-b722-06e333bf5d97" />


Сохранил статистику в файл:

```bash
docker stats --no-stream practice-container-2 > ~/docker-practice/stats.txt
cat ~/docker-practice/stats.txt
```

<img width="1282" height="48" alt="image" src="https://github.com/user-attachments/assets/c49d7ad9-5abe-4c9f-89cc-17d630c7a220" />




Удалил контейнер после завершения и сделал анализ метрик

### Анализ метрик

Наблюдаемые метрики:
- **CPU %**: Процент использования процессорного времени
- **MEM USAGE / LIMIT**: Использование оперативной памяти
- **NET I/O**: Сетевой ввод/вывод
- **BLOCK I/O**: Дисковые операции чтения/записи
- **PIDS**: Количество процессов в контейнере

### Результат

✅ Успешно выполнен мониторинг ресурсов контейнера и сохранены метрики

---

## Задание 3: Ограничение контейнера по CPU и Memory

### Описание задания

Установка ограничений на использование процессора и памяти контейнером.

### Выполнение

Запустил контейнер с ограничениями (256MB памяти, 0.5 CPU):

```bash
docker run -d --name practice-limited \
  --memory=256m \
  --cpus=0.5 \
  practice-image:1.0 60
```

<img width="1143" height="148" alt="image" src="https://github.com/user-attachments/assets/f40b4a2d-5d70-4625-b26c-06152d75871a" />




Проверил статистику с учетом лимитов:

```bash
docker stats --no-stream practice-limited
```

Обновил лимиты на работающем контейнере:

```bash
docker update --memory=512m --cpus=1.0 practice-limited
```


Проверил, остановил и удалил контейнер:

```bash
docker stop practice-limited
docker rm practice-limited
```

<img width="1412" height="59" alt="image" src="https://github.com/user-attachments/assets/10c878b8-b8dc-405e-bc92-4eb26cf6f763" />


### Результат

✅ Успешно применены и изменены ограничения ресурсов контейнера

---

## Задание 4: Сохранение docker-контейнера в tar

### Описание задания

Экспорт файловой системы контейнера в tar-архив для миграции или архивирования.

### Выполнение

Запустил контейнер:

```bash
docker run -d --name practice-export practice-image:1.0 30
```


Проверил статус контейнера:

```bash
docker ps -a | grep practice-export
```

Экспортировал контейнер в tar-архив:

```bash
docker export practice-export > ~/docker-practice/container_export.tar
```

<img width="928" height="25" alt="image" src="https://github.com/user-attachments/assets/c68fbc2e-cf8d-4bc1-9ec8-c5bed46ad03e" />


Проверил размер архива:

```bash
ls -lh ~/docker-practice/container_export.tar
```

Просмотрел содержимое архива:

```bash
tar -tf ~/docker-practice/container_export.tar | head -20
```

<img width="1144" height="514" alt="image" src="https://github.com/user-attachments/assets/4e4dd9a7-0e0b-41db-a8f3-5c8c1c1d2eeb" />




Создал сжатый архив и удалил контейнер

### Результат

✅ Успешно экспортирован контейнер в tar-архив (обычный и сжатый)

---

## Задание 5: Загрузка контейнера из tar

### Описание задания

Импорт образа из tar-архива и запуск контейнера из восстановленного образа.

### Выполнение

Импортировал образ из tar-архива:

```bash
docker import ~/docker-practice/container_export.tar restored-practice:1.0
```

<img width="1245" height="49" alt="image" src="https://github.com/user-attachments/assets/aaa5f062-e8ed-4b53-81e4-098bfed0beac" />




Проверил созданный образ и запустил контейнер из восстановленного образа:

```bash
docker run -d --name restored-from-tar restored-practice:1.0 /home/appuser/entrypoint.sh 30
```

<img width="1245" height="49" alt="image" src="https://github.com/user-attachments/assets/b3cd19d7-656b-46d4-bf03-7552c2f163c7" />



Проверил работу контейнера, просмотрел логи восстановленного контейнера:

```bash
sleep 35
docker logs restored-from-tar
```

<img width="1259" height="511" alt="image" src="https://github.com/user-attachments/assets/02921eb8-05c6-4850-b301-48f81c6a13e2" />




### Результат

✅ Успешно импортирован образ из tar-архива и запущен контейнер

---


## Ответы на контрольные вопросы

### 1. Какие потоки данных перехватывает Docker при логировании и почему это важно для диагностики?

Docker перехватывает стандартные потоки вывода **stdout** (стандартный вывод) и **stderr** (поток ошибок) контейнеризированного приложения. Это важно для диагностики, так как позволяет:
- Восстановить цепь событий при сбоях
- Провести постмортем-анализ проблем
- Отследить все сообщения и ошибки приложения
- Интегрировать логи с внешними системами мониторинга

### 2. Каким образом контрольные группы (cgroups) Linux обеспечивают ограничение ресурсов в Docker?

Контрольные группы (cgroups) работают на уровне ядра Linux и позволяют:
- Отслеживать потребление ресурсов отдельными процессами
- Устанавливать жесткие лимиты на память (hard limit)
- Применять throttling для процессора при превышении лимитов
- Завершать процессы через OOM-killer при исчерпании памяти
- Обеспечивать изоляцию и справедливое распределение ресурсов

### 3. Чем отличается экспорт контейнера от сохранения образа, и в каких сценариях применяется каждый подход?

**Экспорт контейнера (docker export)**:
- Сохраняет файловую систему контейнера в текущем состоянии
- Теряется история слоев образа
- Не сохраняются метаданные (ENTRYPOINT, CMD)
- Используется для миграции настроенных контейнеров

**Сохранение образа (docker save)**:
- Сохраняет образ со всеми слоями и историей
- Сохраняются все метаданные
- Используется для распространения образов

### 4. Почему важно устанавливать ограничения памяти для контейнеров в многоконтейнерной среде?

Ограничения памяти важны для:
- **Предотвращения "noisy neighbor"**: один контейнер не может монополизировать ресурсы
- **Защиты от DoS**: предотвращение полного исчерпания памяти хоста
- **Справедливого распределения**: обеспечение QoS для всех контейнеров
- **Предсказуемости**: гарантированное поведение приложений
- **Безопасности**: изоляция приложений друг от друга

### 5. Какие метрики предоставляет docker stats и как их интерпретировать для оптимизации производительности?

**Метрики docker stats**:
- **CPU %**: использование процессорного времени (высокие значения = нужно больше CPU или оптимизация кода)
- **MEM USAGE / LIMIT**: потребление памяти (рост со временем = утечка памяти)
- **NET I/O**: сетевой трафик (высокие значения = много сетевых операций)
- **BLOCK I/O**: дисковые операции (высокие значения = оптимизировать работу с диском)
- **PIDS**: количество процессов (много процессов = возможно избыточное распараллеливание)

### 6. Как восстановить контейнер из tar-архива и какие ограничения существуют при этом процессе?

**Процесс восстановления**:
```bash
docker import archive.tar image-name:tag
docker run -d --name container image-name command
```

**Ограничения**:
- Теряется история слоев образа
- Не сохраняются ENTRYPOINT и CMD
- Нужно явно указывать команду запуска
- Нет информации о порте и томах из оригинального образа

### 7. Каким образом можно использовать экспорт контейнеров для миграции приложений между хостами?

**Процесс миграции**:
1. Экспорт на исходном хосте: `docker export container > app.tar`
2. Передача архива на целевой хост (scp, rsync)
3. Импорт на целевом хосте: `docker import app.tar app:1.0`
4. Запуск контейнера с нужными параметрами

**Преимущества**:
- Портативность между различными хостами и облаками
- Простота переноса настроенных окружений
- Возможность создания резервных копий

---

## Выводы

В ходе практической работы были изучены и успешно применены следующие возможности Docker:

1. ✅ **Логирование**: Освоены методы захвата и сохранения логов контейнеров для диагностики и аудита
2. ✅ **Мониторинг**: Использована команда `docker stats` для анализа потребления ресурсов
3. ✅ **Ограничение ресурсов**: Применены лимиты CPU и памяти для обеспечения справедливого распределения ресурсов
4. ✅ **Экспорт/Импорт**: Выполнена миграция контейнеров через tar-архивы
