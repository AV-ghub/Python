## Простой пример перекладки данных из PostgreSQL в ClickHouse на Python
> без всяких ETL-фреймворков — просто с использованием `psycopg2` и `clickhouse-connect` (или `clickhouse-driver`

---

## 📦 Установи библиотеки (если ещё не установлены)

```bash
pip install psycopg2-binary clickhouse-connect
```

---

## ✅ Простой рабочий пример

```python
import psycopg2
import clickhouse_connect

# --- Подключение к PostgreSQL
pg_conn = psycopg2.connect(
    host="localhost",
    port=5432,
    database="source_db",
    user="pg_user",
    password="pg_pass"
)
pg_cursor = pg_conn.cursor()

# --- Подключение к ClickHouse
ch_client = clickhouse_connect.get_client(
    host='localhost',
    port=8123,
    username='default',
    password=''
)

# --- Запрос из PostgreSQL
pg_cursor.execute("SELECT id, name, value FROM source_table")
rows = pg_cursor.fetchall()  # Получили список кортежей [(...), (...), ...]

# --- Вставка в ClickHouse
ch_client.insert(
    table='target_table',
    data=rows,
    column_names=['id', 'name', 'value']
)

print("✅ Данные успешно скопированы.")

# --- Закрытие соединений
pg_cursor.close()
pg_conn.close()
```

---

## 🧱 Минимальные таблицы

#### PostgreSQL:

```sql
CREATE TABLE source_table (
    id INT,
    name TEXT,
    value NUMERIC
);
```

#### ClickHouse:

```sql
CREATE TABLE target_table (
    id UInt32,
    name String,
    value Float64
) ENGINE = MergeTree() ORDER BY id;
```

---

## ⚠️ Учитывай:

* **Типы данных должны быть совместимы** (например, `NUMERIC → Float64`)
* ClickHouse **не поддерживает NULL** — замени `None` на значение по умолчанию (если нужно)
* Это **вставка одним пакетом** — без проверки, очистки, дедупликации и т.д.

---

Если нужно сделать:

* **батчи** (частями),
* **предобработку** (например, округлить, заменить `None`),
* **логгирование или отладку** — скажи, и я добавлю

