## 📚 **Книги и гайды для продвинутого уровня**

### 1. **"Effective Pandas" – Matt Harrison** *(продвинутые техники pandas)*

* Фокус на производительности, группировках, трансформациях и pipeline-подходах.
* Хорошо подходит для DataFrame-heavy проектов и оптимизации кода.

### 2. **"Data Science at Scale with Python and Dask" – Jesse Daniel**

* Про Dask — библиотеку для обработки данных большего объёма, чем помещается в память.
* Практические примеры обработки данных в кластере.

### 3. **"High Performance Python" – Micha Gorelick**

* Оптимизация кода, работа с памятью, ускорение расчётов, профилирование.

### 4. **"Learning Spark" (O’Reilly)**

* Подробное руководство по Apache Spark, включая PySpark.

---

## 💻 **Онлайн-курсы (продвинутые)**

### Coursera / edX / Udemy:

1. **[Big Data Analysis with Scala and Spark](https://www.coursera.org/learn/scala-spark-big-data)** — но можно адаптировать идеи под PySpark.
2. **[Advanced Machine Learning Specialization – HSE / Yandex](https://www.coursera.org/specializations/aml)** (на Python, сильный ML + Big Data контекст).
3. **[Data Engineering with Google Cloud](https://www.coursera.org/professional-certificates/gcp-data-engineering)** — изучаешь BigQuery, Dataflow, Apache Beam (есть Python SDK).
4. **[PySpark Essentials for Data Scientists](https://www.udemy.com/course/pyspark-essentials-for-data-scientists/)** — практический проектный подход к Spark с Python.

---

## 🛠️ **Инструменты и библиотеки для Big Data / Data Eng.**

| Библиотека                    | Что делает                                                   |
| ----------------------------- | ------------------------------------------------------------ |
| `pyspark`                     | Работа с распределёнными данными на кластерах (Apache Spark) |
| `dask`                        | Аналог pandas для распределённой/параллельной обработки      |
| `luigi`, `airflow`, `prefect` | Data pipeline'ы, DAG-и, ETL                                  |
| `pyarrow`, `fastparquet`      | Быстрое хранение и чтение big data (Parquet, Arrow)          |
| `duckdb`, `clickhouse`        | In-memory базы для аналитики с Python API                    |
| `polars`                      | Альтернатива pandas — быстрее и эффективнее (в Rust)         |

---

## 📁 **Проекты на GitHub для вдохновения и практики**

* 🔬 [nytimes/covid-19-data](https://github.com/nytimes/covid-19-data) — кейс по сбору и анализу публичных данных.
* 🧠 [ageron/handson-ml3](https://github.com/ageron/handson-ml3) — репозиторий к книге "Hands-on ML", включая обработку больших объёмов.
* 🚀 [dask/dask-examples](https://github.com/dask/dask-examples) — демонстрация распределённой обработки данных.
* 🧮 [Polars Examples](https://github.com/pola-rs/polars/tree/main/examples/python) — как обрабатывать большие таблицы с использованием Polars.

---

## 🎓 **Что попробовать в проектах уровня "продвинутый"**

* Построить ETL pipeline с использованием `Airflow` + `Spark` + `PostgreSQL`.
* Реализовать кластерную обработку логов с `Dask` или `Spark`, сохраняя результат в `Parquet`.
* Построить ML pipeline (scikit-learn или PyTorch) и обучать модели на данных, загружаемых частями из `BigQuery` или `S3`.
* Анализировать миллионные датасеты с помощью `DuckDB` или `Polars` (вместо pandas).

---

## Туториалы, с которых стоит начать
Отлично, тогда начнём с **проверенных, хорошо структурированных туториалов**, которые дадут тебе прочную практическую базу для обработки больших данных и использования Python в серьёзных проектах.

---

## 🔥 **ТОП-5 туториалов и учебных проектов для среднего / продвинутого уровня**

---

### 1. **Dask для работы с большими данными**

📍 **Официальный туториал:**
🔗 [https://examples.dask.org/](https://examples.dask.org/)

📌 Что изучишь:

* Работа с большими CSV/Parquet файлами, превышающими объем RAM
* Распределённые вычисления с DataFrame, Array и ML-модулями
* Интеграция с pandas и NumPy

🛠 Подходит, если ты уже уверенно работаешь с `pandas`, и хочешь выйти за пределы памяти одного компьютера.

---

### 2. **PySpark: обработка Big Data в Apache Spark**

📍 **Хороший вводный курс с практикой:**
🔗 [https://github.com/databricks/learning-spark-examples](https://github.com/databricks/learning-spark-examples)
или
🔗 [PySpark Tutorial for Beginners (Analytics Vidhya)](https://www.analyticsvidhya.com/blog/2021/05/a-beginners-guide-to-pyspark/)

📌 Что изучишь:

* Spark DataFrame API, SQL и трансформации
* Обработка логов, больших текстов, распределённые расчёты
* Настройка локального кластера Spark + Python

🛠 Рекомендуется, если хочешь освоить индустриальный инструмент Big Data.

---

### 3. **Polars: быстрая альтернатива pandas**

📍 **Официальные примеры и документация:**
🔗 [https://pola-rs.github.io/polars-book/](https://pola-rs.github.io/polars-book/)

📌 Что изучишь:

* Как быстро обрабатывать таблицы на миллионы строк
* Сравнение по скорости и памяти с pandas
* Векторизованные вычисления и ленивые трансформации

🛠 Подходит, если тебе нужна скорость и эффективность при анализе больших таблиц.

---

### 4. **DuckDB + Python: SQL по файлам в стиле pandas**

📍 **Хороший интро-пост:**
🔗 [https://duckdb.org/docs/guides/python/overview.html](https://duckdb.org/docs/guides/python/overview.html)
📍 **Проектные примеры:**
🔗 [https://github.com/duckdb/duckdb/tree/main/tools/python/examples](https://github.com/duckdb/duckdb/tree/main/tools/python/examples)

📌 Что изучишь:

* Как выполнять SQL-запросы прямо по CSV, Parquet, без загрузки в RAM
* Комбинирование `duckdb` и `pandas` / `polars`
* Использование с `Jupyter` или `Streamlit`

🛠 если интересен SQL + Python подход без тяжелых баз данных.

---

### 5. **Airflow + Python: построение data pipeline**

📍 **Официальный туториал Apache Airflow:**
🔗 [https://airflow.apache.org/docs/apache-airflow/stable/tutorial/index.html](https://airflow.apache.org/docs/apache-airflow/stable/tutorial/index.html)

📌 Что изучишь:

* Как писать DAG-пайплайны для ежедневной ETL-обработки
* Создание тасков на Python, bash и SQL
* Планирование и управление зависимостями

🛠 Must-have, если планируешь работать с системами автоматической обработки данных.

---

## 👨‍💻 Советы по практике

* Запускай всё локально сначала, потом можно перейти на кластер или облако (например, Google Colab или AWS).
* Используй Jupyter Notebooks или VS Code с интерактивным режимом.
* Пробуй не просто копировать код, а модифицировать под свои данные или задачи.
* Сохраняй свои тетрадки / пайплайны в GitHub, это поможет собирать портфолио.


