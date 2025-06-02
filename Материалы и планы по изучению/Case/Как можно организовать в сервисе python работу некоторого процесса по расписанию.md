## ✅ **4 рабочих способа запуска задач по расписанию в Python**

---

### 🔹 1. **Библиотека `schedule`** — простой и понятный способ

📦 Установка:

```bash
pip install schedule
```

📘 Пример:

```python
import schedule
import time

def my_job():
    print("Выполняю задачу...")

# Запуск каждый день в 10:30
schedule.every().day.at("10:30").do(my_job)

while True:
    schedule.run_pending()
    time.sleep(1)
```

🧩 Особенности:

* Простая реализация для **локального или фонового запуска**
* Хорошо подходит для простых сценариев (обработка файла, отправка почты и т.д.)
* Требует, чтобы процесс работал непрерывно (например, как `daemon` или служба)

---

### 🔹 2. **Модуль `APScheduler`** — более гибкий планировщик

📦 Установка:

```bash
pip install apscheduler
```

📘 Пример (запуск каждую минуту):

```python
from apscheduler.schedulers.blocking import BlockingScheduler

def job_function():
    print("Выполняю задачу!")

scheduler = BlockingScheduler()
scheduler.add_job(job_function, 'interval', minutes=1)
scheduler.start()
```

🧩 Поддерживает:

* Планирование по времени (раз в минуту, каждый день, по cron)
* Варианты запуска: в фоне, в отдельном процессе, с базой данных
* Подходит для **сервисов и веб-приложений** (можно интегрировать с `Flask`, `FastAPI`, `Django`)

🔗 Документация: [https://apscheduler.readthedocs.io/](https://apscheduler.readthedocs.io/)

---

### 🔹 3. **Обычный cron + Python скрипт (на Linux)**

📘 Добавление задачи:

```bash
crontab -e
```

📘 Пример: запуск Python-скрипта каждый час

```bash
0 * * * * /usr/bin/python3 /home/user/scripts/my_job.py >> /home/user/logs/my_job.log 2>&1
```

🧩 Просто и стабильно, но cron есть не везде (не работает в Windows без WSL). Для Windows есть аналог — **Task Scheduler**.

---

### 🔹 4. **Инструменты промышленного уровня (как SQL Server Agent)**

#### ✅ Apache Airflow (для продвинутых пайплайнов)

* Написан на Python, используется в data engineering
* Поддерживает DAG-и, расписание, мониторинг, повторные попытки
* Идеален, если задач много и они взаимосвязаны

🔗 [https://airflow.apache.org/](https://airflow.apache.org/)

---

## 🧰 Что выбрать?

| Цель                       | Подход                  |
| -------------------------- | ----------------------- |
| Простые скрипты по времени | `schedule` или `cron`   |
| Гибкое планирование        | `apscheduler`           |
| Много задач и зависимостей | Apache Airflow, Prefect |
| Windows/GUI планирование   | Windows Task Scheduler  |

## ✅ Пример: `APScheduler` как фоновая служба для ETL-пайплайна

---

### 📦 Шаг 1. Установи APScheduler

```bash
pip install apscheduler
```

---

### 🛠️ Шаг 2. Пример скрипта `etl_scheduler.py`

```python
from apscheduler.schedulers.background import BackgroundScheduler
from datetime import datetime
import time

# Твой ETL-процесс
def run_etl():
    print(f"[{datetime.now()}] → Запуск ETL процесса...")
    # Пример ETL:
    # 1. Загрузка данных
    # 2. Преобразование
    # 3. Сохранение
    # Здесь ты можешь подключать pandas, SQL, API и т.п.
    print("ETL завершён.\n")

# Создаём фоновый планировщик
scheduler = BackgroundScheduler()
scheduler.add_job(run_etl, 'interval', minutes=5)  # Каждые 5 минут
scheduler.start()

print("Фоновый ETL-сервис запущен. Нажми Ctrl+C для остановки.")

# Держим процесс живым
try:
    while True:
        time.sleep(1)
except (KeyboardInterrupt, SystemExit):
    scheduler.shutdown()
    print("Остановлено.")
```

---

### 📘 Что происходит:

* `BackgroundScheduler` запускает задачу в фоне
* `run_etl()` выполняется каждые 5 минут
* Скрипт можно запустить в консоли, как обычный демон

---

### 📂 Как развернуть как **службу** или **фоновый процесс**:

#### 🔸 На Linux:

```bash
nohup python etl_scheduler.py > etl.log 2>&1 &
```

#### 🔸 На Windows:

* Запусти через **Task Scheduler**
* Или создай **.bat-файл**, который будет запускать Python-скрипт

---

### 📌 Можно добавить логирование:

```python
import logging

logging.basicConfig(filename='etl.log', level=logging.INFO)

def run_etl():
    logging.info(f"{datetime.now()} - запуск ETL")
    ...
```

---

## ✅ Общий план действий (в виде шагов пайплайна)

---

### 🧩 **1. Извлечение данных из PostgreSQL**

* Читаем только *свежие* записи — через `timestamp`, `id`, флаг, или логическую таблицу
* Используем `psycopg2` или `sqlalchemy`

### 🔁 **2. Возможные трансформации в Python**

* `pandas`, `polars`, `pyarrow` — в зависимости от сложности
* Валидация, фильтрация, преобразование типов, обогащение

### 📤 **3. Загрузка в ClickHouse (в “триггерную” таблицу)**

* Через HTTP API, `clickhouse-connect` или `clickhouse-driver`
* После вставки можно вызвать `OPTIMIZE TABLE` или другой maintenance

### ⏳ **4. Ожидание завершения внутренних трансформаций**

* Мониторим таблицу-флаг или статус в ClickHouse
* Простой `while` с паузами (или timeout)

### 🧹 **5. Постобработка**

* Удаление обработанных данных, архивация, создание индексов и т.п.
* Выполняем SQL-обслуживание на стороне Postgres и ClickHouse

---

## 🔧 Пример структуры `etl_scheduler.py` (упрощённо)

```python
from apscheduler.schedulers.background import BackgroundScheduler
from datetime import datetime
import time
import psycopg2
from clickhouse_driver import Client
import pandas as pd

# Настройки подключения (вынеси в .env или config.yaml)
PG_CONN = {...}
CLICK_CONN = {...}

def run_etl():
    print(f"[{datetime.now()}] Запуск ETL...")

    # 1. Подключение к PostgreSQL
    with psycopg2.connect(**PG_CONN) as pg_conn:
        cursor = pg_conn.cursor()
        cursor.execute("SELECT * FROM source_table WHERE processed = false")
        rows = cursor.fetchall()

    # 2. Преобразование в pandas (если нужно)
    df = pd.DataFrame(rows, columns=['id', 'name', 'timestamp'])

    # (опционально) трансформация данных
    df['name'] = df['name'].str.upper()

    # 3. Вставка в ClickHouse
    ch = Client(**CLICK_CONN)
    insert_data = df.to_dict('records')
    ch.execute("INSERT INTO trigger_table (id, name, timestamp) VALUES", insert_data)

    # 4. Ждём окончания внутренней обработки ClickHouse
    while not check_clickhouse_ready(ch):
        print("Ждём завершения обработки...")
        time.sleep(5)

    # 5. Post-processing
    with psycopg2.connect(**PG_CONN) as pg_conn:
        cursor = pg_conn.cursor()
        cursor.execute("UPDATE source_table SET processed = true WHERE processed = false")
        pg_conn.commit()

    print(f"[{datetime.now()}] ETL завершён.")

def check_clickhouse_ready(ch):
    # Простая проверка: смотрим наличие флага или изменения состояния
    result = ch.execute("SELECT count(*) FROM clickhouse_status_table WHERE ready = 0")
    return result[0][0] == 0

# Планировщик
scheduler = BackgroundScheduler()
scheduler.add_job(run_etl, 'cron', minute='*/10')  # каждые 10 минут
scheduler.start()

print("ETL-служба запущена.")
try:
    while True:
        time.sleep(1)
except (KeyboardInterrupt, SystemExit):
    scheduler.shutdown()
    print("Остановлено.")
```

---

## 🧰 Что тебе ещё может пригодиться:

| Что нужно             | Решение / библиотека                         |
| --------------------- | -------------------------------------------- |
| Чтение из PostgreSQL  | `psycopg2` или `sqlalchemy`                  |
| Загрузка в ClickHouse | `clickhouse-driver` или `clickhouse-connect` |
| Трансформации         | `pandas`, `polars`, `pyarrow`                |
| Конфигурации          | `dotenv`, `pydantic`, `yaml`                 |
| Отладка/логи          | `logging`, `loguru`                          |

---

---

## ✅ **Цель: Запустить Python-ETL как systemd-сервис**

---

### 🧩 **1. Особенности Python-кода для запуска под systemd**

Тебе не нужно делать ничего сверхособенного, **но желательно соблюдать**:

#### ✅ Рекомендации:

* Не использовать `input()` или любую форму интерактива.
* Использовать **логирование в файл**, а не `print()`.
* Всегда ловить `KeyboardInterrupt` и завершать корректно (`scheduler.shutdown()`).
* Убедиться, что рабочие пути (logs, config) заданы **абсолютно**.
* Использовать `venv` (или poetry/pipenv), если требуется изоляция.

---

### 📦 **2. Пример кода с логированием**

```python
import time
import logging
from datetime import datetime
from apscheduler.schedulers.background import BackgroundScheduler

# Настройка логирования
logging.basicConfig(
    filename="/var/log/etl_service.log",
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)

def run_etl():
    logging.info("Запуск ETL-процесса.")
    # Здесь твой ETL: выгрузка, преобразование, загрузка
    logging.info("ETL завершён.")

scheduler = BackgroundScheduler()
scheduler.add_job(run_etl, 'cron', minute='*/5')
scheduler.start()

logging.info("ETL-служба запущена.")

try:
    while True:
        time.sleep(1)
except (KeyboardInterrupt, SystemExit):
    scheduler.shutdown()
    logging.info("ETL-служба остановлена.")
```

---

### ⚙️ **3. Юнит-файл systemd (`etl.service`)**

[Systemd](https://github.com/AV-ghub/Linux/blob/main/008%20Administration/Systemd.md)

Создай файл:

```bash
sudo nano /etc/systemd/system/etl.service
```

Пример юнит-файла:

```ini
[Unit]
Description=ETL Background Service
After=network.target

[Service]
Type=simple
User=your_user
WorkingDirectory=/home/your_user/projects/etl
ExecStart=/usr/bin/python3 etl_scheduler.py
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

✅ **Пояснение**:

* `ExecStart` — путь до скрипта или `venv`-обёртки (`ExecStart=/home/your_user/venv/bin/python etl_scheduler.py`)
* `Restart=always` — перезапустит сервис при сбое
* `User` — под каким пользователем запускать

---

### 🚀 **4. Управление службой**

```bash
sudo systemctl daemon-reexec         # Если systemd перезагружался
sudo systemctl daemon-reload         # После создания/изменения etl.service
sudo systemctl start etl.service     # Запустить
sudo systemctl enable etl.service    # Включить автозапуск
sudo systemctl status etl.service    # Проверить статус
sudo journalctl -u etl.service       # Логи
```

---

## ⚠️ Возможные подводные камни:

| Проблема                   | Что делать                                                                                   |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| Нет доступа к логам/файлам | Убедись, что `User=` имеет права                                                             |
| Ошибки импорта             | Убедись, что `venv` правильно указан                                                         |
| `ExecStart` не срабатывает | Указывай **полный путь к Python**                                                            |
| Права доступа к БД         | Убедись, что переменные окружения передаются (можно через `.env` или systemd `Environment=`) |

---

---

## ✅ Как правильно запускать Python-сервис с `venv` и `.env` через systemd

---

### 🧩 Структура проекта (пример)

```
etl_project/
├── venv/                      ← виртуальное окружение
├── .env                       ← переменные окружения (логин/пароль БД)
├── etl_scheduler.py          ← основной скрипт
└── run_etl.sh                ← обёртка для systemd
```

---

### 🔹 1. Пример `.env` файла

```dotenv
PG_HOST=localhost
PG_PORT=5432
PG_USER=etl_user
PG_PASSWORD=secret
PG_DATABASE=source_db

CLICK_USER=default
CLICK_PASSWORD=
CLICK_HOST=localhost
CLICK_PORT=9000
```

> ⚠️ Не коммить `.env` в Git!

---

### 🔹 2. Подгрузка `.env` в Python (через `python-dotenv`)

📦 Установка:

```bash
source venv/bin/activate
pip install python-dotenv
```

📘 В `etl_scheduler.py`:

```python
from dotenv import load_dotenv
import os

load_dotenv(dotenv_path=".env")

pg_host = os.getenv("PG_HOST")
pg_user = os.getenv("PG_USER")
click_host = os.getenv("CLICK_HOST")
# и так далее
```

---

### 🔹 3. Скрипт-обёртка `run_etl.sh` для systemd

```bash
#!/bin/bash
cd /home/your_user/etl_project
source venv/bin/activate
exec python etl_scheduler.py
```

> 📌 Сделай его исполняемым:

```bash
chmod +x run_etl.sh
```

---

### 🔹 4. Юнит-файл systemd (`etl.service`)

```ini
[Unit]
Description=ETL Service with venv and .env
After=network.target

[Service]
Type=simple
User=your_user
WorkingDirectory=/home/your_user/etl_project
ExecStart=/home/your_user/etl_project/run_etl.sh
Restart=always
RestartSec=5
EnvironmentFile=/home/your_user/etl_project/.env

[Install]
WantedBy=multi-user.target
```

> 🔐 `EnvironmentFile` — опционально, если хочешь передавать `.env` ещё и как переменные окружения (можно использовать вместе с `python-dotenv`, особенно если какие-то переменные нужны вне Python)

---

### 🔧 5. Запуск и отладка

```bash
sudo systemctl daemon-reload
sudo systemctl enable etl.service
sudo systemctl start etl.service
sudo journalctl -u etl.service -f  # Логи в реальном времени
```

---

## 🚨 Подводные камни

| Ошибка                | Причина                                               |
| --------------------- | ----------------------------------------------------- |
| `ModuleNotFoundError` | systemd запускает **не тот Python**, не из `venv`     |
| `.env не читается`    | Забыт `load_dotenv()` или не указан `dotenv_path`     |
| `Permission denied`   | `run_etl.sh` не имеет `+x` или `.env` закрыт          |
| Логов нет             | `StandardOutput=journal` и `journalctl` — смотри логи |

---

## ✅ Резюме: что использовать

| Файл            | Назначение                             |
| --------------- | -------------------------------------- |
| `.env`          | Хранит конфигурацию (БД, ключи и т.п.) |
| `python-dotenv` | Подгружает `.env` в Python             |
| `venv`          | Изолирует зависимости                  |
| `run_etl.sh`    | Гарантирует запуск правильной среды    |
| `etl.service`   | Управляет процессом на уровне ОС       |

---

---

### 📦 Скопируй и выполни этот код на своей машине (Python 3.8+):

```python
import zipfile
from pathlib import Path

project_dir = Path("etl_project")
project_dir.mkdir(exist_ok=True)

env_content = """PG_HOST=localhost
PG_PORT=5432
PG_USER=etl_user
PG_PASSWORD=secret
PG_DATABASE=source_db

CLICK_USER=default
CLICK_PASSWORD=
CLICK_HOST=localhost
CLICK_PORT=9000
"""

run_etl_sh = """#!/bin/bash
cd "$(dirname "$0")"
source venv/bin/activate
exec python etl_scheduler.py
"""

etl_scheduler_py = '''import os
import time
import logging
from datetime import datetime
from dotenv import load_dotenv
from apscheduler.schedulers.background import BackgroundScheduler

# Загрузка .env
load_dotenv(dotenv_path=".env")

# Настройка логирования
logging.basicConfig(
    filename="etl.log",
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)

# Пример ETL задачи
def run_etl():
    logging.info("Запуск ETL процесса")
    # Здесь: подключение к БД, извлечение, обработка, загрузка
    logging.info("ETL завершён")

scheduler = BackgroundScheduler()
scheduler.add_job(run_etl, 'cron', minute='*/5')
scheduler.start()

logging.info("ETL служба запущена")
try:
    while True:
        time.sleep(1)
except (KeyboardInterrupt, SystemExit):
    scheduler.shutdown()
    logging.info("ETL служба остановлена")
'''

etl_service = """[Unit]
Description=ETL Service with venv and .env
After=network.target

[Service]
Type=simple
User=your_user
WorkingDirectory=/home/your_user/etl_project
ExecStart=/home/your_user/etl_project/run_etl.sh
Restart=always
RestartSec=5
EnvironmentFile=/home/your_user/etl_project/.env

[Install]
WantedBy=multi-user.target
"""

# Создание файлов
(project_dir / ".env").write_text(env_content)
(project_dir / "run_etl.sh").write_text(run_etl_sh)
(project_dir / "etl_scheduler.py").write_text(etl_scheduler_py)
(project_dir / "etl.service").write_text(etl_service)
(project_dir / "run_etl.sh").chmod(0o755)

# Архивирование
zip_path = Path("etl_project_template.zip")
with zipfile.ZipFile(zip_path, "w") as zipf:
    for file_path in project_dir.iterdir():
        zipf.write(file_path, arcname=file_path.name)

print("Готово: etl_project_template.zip")
```

