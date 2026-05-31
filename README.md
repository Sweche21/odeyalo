# Одеяло Backend

Backend-сервис для платформы психологической поддержки `одеяло.tech`. Проект развивался как учебный стартап в рамках ГПО при ТУСУР: серверная часть отвечает за пользователей, роли, заявки, психологические тесты, упражнения, дневник, отслеживание настроения, обучение, геймификацию, фоновые задачи и интеграцию с клиентским приложением.

## Что внутри

- REST API на FastAPI для клиентского приложения и административных сценариев.
- Авторизация, refresh/access tokens, роли пользователей и защищенные методы API.
- Модули для клиентов, менеджеров, психологов и администраторов.
- Заявки, отзывы, дневник, mood tracker, пользовательские задачи и ежедневные задания.
- Психологические тесты, упражнения, тренировочные упражнения и образовательные материалы.
- Геймификация, чат-бот, онтологический модуль и расчетные сервисы.
- PostgreSQL как основное хранилище, Redis для кэша, Celery для фоновых задач.
- Docker Compose окружение с backend, PostgreSQL, Redis, Celery worker/beat, Nginx и backup-сервисом.
- Автотесты: unit, integration и smoke-проверки API/сервисов.

## Стек

| Зона | Технологии |
| --- | --- |
| Backend | Python, FastAPI, Uvicorn |
| API и валидация | Pydantic, Swagger / OpenAPI |
| База данных | PostgreSQL, asyncpg, SQLAlchemy 2, Alembic |
| Auth | PyJWT, passlib, bcrypt |
| Кэш и фоновые задачи | Redis, fastapi-cache2, Celery |
| Инфраструктура | Docker, Docker Compose, Nginx |
| Тестирование | pytest, pytest-asyncio, httpx |
| Качество кода | Ruff, Black |

## Архитектура

Полная backend-часть организована по слоям:

```text
src/
  api/             # HTTP routes, dependencies, входные точки модулей
  services/        # бизнес-логика приложения
  repositories/    # доступ к данным и query-логика
  models/          # SQLAlchemy ORM-модели
  schemas/         # Pydantic-схемы запросов и ответов
  migrations/      # Alembic-миграции
  tasks/           # Celery-приложение и фоновые задачи
  connectors/      # внешние подключения, например Redis
  utils/           # общие утилиты
autotest/
  local/           # unit и локальные API-тесты
  integration/     # интеграционные проверки
  docker/smoke/    # smoke-тесты в контейнерном окружении
nginx/             # reverse proxy конфигурация
```

Основной принцип: `api` принимает запрос и валидирует входные данные, `services` выполняют бизнес-логику, `repositories` работают с БД, `models` описывают хранение, а `schemas` задают контракт API.

## Основные API-модули

- `auth` и `yandex_auth` - регистрация, вход, токены, восстановление доступа.
- `client`, `manager`, `psychologist`, `admin` - сценарии разных ролей.
- `application` и `review` - заявки и отзывы.
- `tests`, `exercise`, `training_exercise` - тесты и упражнения.
- `education` - образовательные темы, материалы и прогресс.
- `diary`, `mood_tracker` - дневник и отслеживание состояния.
- `daily_tasks`, `user_task`, `gamification` - задания и геймификация.
- `chat_bot`, `ontology` - дополнительные интеллектуальные сценарии.

В полной кодовой базе backend включает около 150 REST API routes.

## Быстрый запуск через Docker Compose

Создайте файл окружения:

```bash
cp .env-example .env
```

Для Windows PowerShell:

```powershell
Copy-Item .env-example .env
```

Запустите сервисы:

```bash
docker compose up --build
```

После запуска API будет доступен через Nginx:

```text
http://localhost:1802
```

Swagger UI:

```text
http://localhost:1802/docs
```

## Локальный запуск без Docker

Создайте виртуальное окружение и установите зависимости:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Для Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Подготовьте `.env`, примените миграции и запустите приложение:

```bash
alembic upgrade head
uvicorn src.main:app --reload
```

## Переменные окружения

Пример конфигурации хранится в `.env-example`. Для запуска нужны:

```env
MODE=LOCAL
DB_HOST=localhost
DB_PORT=4321
DB_USER=postgres
DB_PASS=postgres
DB_NAME=psycho2
REDIS_HOST=localhost
REDIS_PORT=6789
JWT_SECRET_KEY=change_me
JWT_REFRESH_SECRET_KEY=change_me
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=30
```

Секреты, пароли SMTP, JWT-ключи и production-настройки не должны храниться в репозитории. Используйте `.env`, секреты CI/CD или переменные окружения сервера.

## Миграции базы данных

Применить миграции:

```bash
alembic upgrade head
```

Создать новую миграцию:

```bash
alembic revision --autogenerate -m "describe changes"
```

Откатить последнюю миграцию:

```bash
alembic downgrade -1
```

## Тесты

Запуск всех доступных тестов:

```bash
pytest
```

Локальные unit/API тесты:

```bash
pytest autotest/local
```

Интеграционные тесты:

```bash
pytest autotest/integration
```

Smoke-проверки в контейнерном окружении:

```bash
docker compose -f autotest/docker/docker-compose.yml up --build --abort-on-container-exit
```

## Полезные команды

Запуск приложения:

```bash
uvicorn src.main:app --reload
```

Форматирование:

```bash
black src autotest
```

Линтинг:

```bash
ruff check src autotest
```

Просмотр логов контейнеров:

```bash
docker compose logs -f app
docker compose logs -f celery_worker
docker compose logs -f nginx
```

## Статус проекта

Проектная backend-часть использовалась как основа учебного стартапа `одеяло.tech` и демонстрировалась на мероприятиях / стартап-полигонах. README описывает backend-кодовую базу и способ ее запуска в development-окружении.

## Автор

Павел Пупенко  
Backend-разработчик  
Telegram: [@swechee](https://t.me/swechee)  
Email: [sweche21@mail.ru](mailto:sweche21@mail.ru)
