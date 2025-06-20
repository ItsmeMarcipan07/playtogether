# Sports Activities API

## 1. Бърз старт

### Локално с Docker Compose

```bash
docker compose up --build
```

Docker Compose ще:

* изгради образ **backend**, базиран на `Dockerfile`, инсталира зависимостите и стартира FastAPI сървър на порт **8000**.  Командата `uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload` се изпълнява след автоматично прилагане на миграциите чрез Alembic.
* изтегли официален образ **postgres:13** и ще създаде база данни на порт **5432** с персистентен том `postgres_data`.

### Основни команди

* **Създаване/презаписване на контейнерите**
  `docker compose up --build`
* **Спиране**
  `docker compose down`
* **Достъп до базата**
  `docker compose exec db psql -U $POSTGRES_USER $POSTGRES_DB`

## 2. Структура на проекта

```
.
├── app
│   ├── main.py         # FastAPI маршрути
│   ├── models.py       # SQLAlchemy модели
│   ├── schemas.py      # Pydantic схеми (DTO)
│   ├── crud.py         # Бизнес-логика/заявки към БД
│   └── database.py     # Инициализация на връзката към Postgres
├── alembic/            # Скриптове за миграции (генерират се автоматично)
├── Dockerfile
├── docker-compose.yml  # Оркестрация на услугите
└── README.md
```

## 3. Описание на компонентите

| Компонент                     | Роля                            | Ключови особености                                                        |
| ----------------------------- | ------------------------------- | ------------------------------------------------------------------------- |
| **FastAPI** (`app/main.py`)   | REST API слой                   | Определя CRUD крайни точки за потребители, активности, резервации и др.   |
| **CRUD слой** (`app/crud.py`) | Инкапсулира заявки към БД       | Хешира пароли с *passlib*, използва транзакции и връща ORM обекти.        |
| **Модели** (`app/models.py`)  | Описва структурата на таблиците | Използва SQLAlchemy + Enums за статуси и типове.                          |
| **Схеми** (`app/schemas.py`)  | Валидира вход/изход             | Pydantic модели, които се използват от FastAPI за сериализация.           |
| **База данни** (PostgreSQL)   | Персистенция                    | Достъпвана през SQLAlchemy, миграции с Alembic.                           |
| **Alembic**                   | Миграции                        | Стартира автоматично при `docker compose up` чрез `alembic upgrade head`. |

## 4. Комуникация между услугите

* Контейнерът **backend** и **db** споделят дефинирана от Docker мрежа (bridge).
* FastAPI достъпва Postgres чрез `DATABASE_URL=postgresql://<user>:<pass>@db:5432/<db>` – **`db`** е DNS името на услугата, декларирано в *docker-compose.yml*.
* Вътрешният пул на SQLAlchemy се конфигурира в `app/database.py` и се използва във всички депенденции (`Depends(get_db)`).

## 5. Променливи на средата

Създайте **.env** файл на кореново ниво:

```dotenv
# Postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=sportsmate
DATABASE_URL=postgresql://postgres:postgres@db:5432/sportsmate

# Alembic
DB_SCHEMA=public
```

## 6. Тестване на API

След стартиране отворете [http://localhost:8000/docs](http://localhost:8000/docs) за Swagger UI или изпратете примерна заявка:

```bash
curl -X POST http://localhost:8000/users/ \
     -H "Content-Type: application/json" \
     -d '{"username":"demo","email":"demo@demo.bg","password":"secret"}'
```
