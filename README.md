# SOSAitu Deployment Guide

## Структура папок

```
devops/
├── backend/
│   ├── Dockerfile              # Dockerfile для бэкенда
│   └── v1-0.0.1-SNAPSHOT.jar   # JAR файл приложения
├── frontend/
│   └── dist/                   # Собранный фронтенд (Vite)
│       ├── index.html
│       └── assets/
├── docker-compose.yml          # Основная конфигурация Docker
└── nginx.conf                  # Конфигурация Nginx
```

## Быстрый старт

### 1. Сборка и деплой

```bash
# Из корня проекта
./build-and-deploy.sh
```

Этот скрипт:
- Собирает фронтенд (`npm run build`)
- Копирует `dist/` в `devops/frontend/`
- Собирает бэкенд (`./mvnw clean package`)
- Копирует JAR в `devops/backend/`

### 2. Запуск Docker Compose

```bash
cd devops
docker-compose up -d --build
```

### 3. Проверка статуса

```bash
docker-compose ps
docker-compose logs -f
```

## Сервисы

| Сервис     | Порт      | Описание                          |
|------------|-----------|-----------------------------------|
| postgres   | 5433:5432 | PostgreSQL база данных            |
| backend    | 8080      | Spring Boot приложение (внутренний) |
| nginx      | 8080:8080 | Nginx веб-сервер (публичный)      |
| ollama     | 11434     | AI модель Qwen2.5:3b              |

## Переменные окружения

### Backend
```bash
SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/sosa_db
SPRING_DATASOURCE_USERNAME=whyy0u
SPRING_DATASOURCE_PASSWORD=whyy0u_super_passwdlzo
OLLAMA_BASE_URL=http://ollama:11434
SPRING_PROFILES_ACTIVE=docker
```

### Database
```bash
POSTGRES_DB=sosa_db
POSTGRES_USER=whyy0u
POSTGRES_PASSWORD=whyy0u_super_passwdlzo
```

## Миграции БД

После первого запуска примените миграции:

```bash
# Подключиться к БД
docker exec -it sosa_postgres psql -U whyy0u -d sosa_db

# Применить миграции
docker cp backend/set_role_owner.sql sosa_postgres:/tmp/
docker exec -it sosa_postgres psql -U whyy0u -d sosa_db -f /tmp/set_role_owner.sql
```

## Логи

```bash
# Все логи
docker-compose logs -f

# Только бэкенд
docker-compose logs -f backend

# Только nginx
docker-compose logs -f nginx
```

## Остановка

```bash
# Остановить все сервисы
docker-compose down

# Остановить с удалением volumes (данные будут удалены!)
docker-compose down -v
```

## Обновление

```bash
# 1. Собрать новую версию
./build-and-deploy.sh

# 2. Пересобрать и перезапустить контейнеры
cd devops
docker-compose up -d --build
```

## Ресурсы

| Сервис   | Лимит памяти | Резерв памяти |
|----------|--------------|---------------|
| postgres | 1 GB         | 512 MB        |
| backend  | 1 GB         | 512 MB        |
| nginx    | 256 MB       | 128 MB        |
| ollama   | 6 GB         | 3 GB          |

## Troubleshooting

### Бэкенд не запускается
```bash
docker-compose logs backend
# Проверить подключение к БД и Ollama
```

### Фронтенд не отображается
```bash
docker-compose logs nginx
# Проверить наличие dist/ в devops/frontend/
```

### Ollama не загружает модель
```bash
docker-compose logs ollama
# Убедитесь что есть 3GB+ свободной памяти
```

## Безопасность

⚠️ **Важно:** Перед деплоем на продакшен:
1. Смените пароли в `docker-compose.yml`
2. Используйте HTTPS для Nginx
3. Ограничьте доступ к порту PostgreSQL
4. Используйте secrets для чувствительных данных
