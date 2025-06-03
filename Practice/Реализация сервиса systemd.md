## Передварительные замечания
### Есть код, предназначенный для запуска в systemd и в нем запускается процесс **scheduler.start()**  

* При запуске скрипта вручную в терминале (со строки), процесс будет привязан к сессии терминала.
* Если прервать сессию (например, закрыть терминал или нажмёшь Ctrl+C), то по умолчанию процесс тоже завершится, потому что получает сигнал завершения при закрытии сессии (SIGHUP).
* Если же ты просто закрыть терминал, не делая Ctrl+C, процесс тоже, скорее всего, остановится — опять же из-за SIGHUP.

Если нужно, чтобы процесс **работал независимо от сессии**, нужно запускать его через:

* `nohup` (например, `nohup python script.py &`) — чтобы игнорировать SIGHUP, или
* запускать как systemd-сервис, тогда systemd сам будет управлять процессом и он не зависим от пользовательской сессии.

---

**Вывод:**
Если пустить скрипт терминале и просто закрыть терминал — процесс завершится. Если нужно, чтобы он продолжал работать — запускать надо через systemd или с помощью `nohup`/screen/tmux.

### BackgroundScheduler 
Это часть библиотеки **APScheduler** (Advanced Python Scheduler), которая используется для планирования задач.

#### Официальная документация и описание

* Официальный сайт APScheduler:
  [https://apscheduler.readthedocs.io/en/latest/](https://apscheduler.readthedocs.io/en/latest/)

* Конкретно про `BackgroundScheduler` и метод `add_job`:
  [https://apscheduler.readthedocs.io/en/latest/modules/schedulers/background.html](https://apscheduler.readthedocs.io/en/latest/modules/schedulers/background.html)
  и
  [https://apscheduler.readthedocs.io/en/latest/modules/triggers/cron.html](https://apscheduler.readthedocs.io/en/latest/modules/triggers/cron.html)

---

**Про параметры `add_job(run_etl, 'cron', minute='*/5')`:**

* `run_etl` — функция, которую нужно запустить.
* `'cron'` — тип триггера, работает как расписание cron.
* `minute='*/5'` — означает «каждые 5 минут». Аналог записи в crontab `*/5 * * * *`.

Можно задать другие параметры для триггера `'cron'` — `hour`, `day`, `month`, `day_of_week` и т.п.

---

Когда запускаем код под Windows из VSCode в режиме отладки и остановливаем отладочную сессию, процесс **автоматически завершится**.

Почему?

* В режиме отладки VSCode запускает скрипт как дочерний процесс.
* Когда останавливаем отладку, VSCode посылает сигнал на завершение этого процесса.
* Следовательно, `scheduler.start()` (который запускает фоновые задачи) тоже прекратит работу, так как процесс, в котором он работает, завершится.

---

### Как всё работает в зависимости от способа запуска и нюансы

---

#### 1. Запуск из VSCode в режиме отладки (Debug)

* VSCode запускает скрипт как дочерний процесс.
* При остановке отладки VSCode **посылает сигнал завершения** процессу.
* Процесс завершается, и вместе с ним останавливается `BackgroundScheduler`.
* Никаких "зависших" процессов после остановки отладки не остаётся.

**Итого:** Всё чисто, вручную ничего не убивать не нужно.

---

#### 2. Запуск из терминала вручную

* Процесс запускается и привязан к сессии терминала.
* Если закрыть терминал, процесс по умолчанию получает сигнал SIGHUP и завершается.
* Но если запустить процесс в фоне с `nohup`, `&`, `screen` или `tmux`, он может продолжать работу вне сессии.
* Тогда придется либо вручную останавливать, либо настроить корректный выход в коде.

---

#### 3. Запуск как systemd-сервис (Linux)

* systemd управляет процессом.
* Процесс не зависит от пользовательской сессии и работает "вечно" пока не скажут остановиться.
* При перезагрузке сервиса или системы systemd автоматически запускает процесс заново (если настроено).
* Для остановки нужно использовать `systemctl stop`.

---

#### Касательно `BackgroundScheduler`

* Он запускает задачи в фоне в рамках **текущего процесса**.
* Если сам процесс завершается — задачи тоже останавливаются.
* Если нужно, чтобы scheduler работал «как сервис», лучше запускать процесс как службу (systemd, Windows Service) или с помощью менеджеров процессов (например, PM2, Supervisor, NSSM на Windows).

---

#### Как это работает `logging.basicConfig` и параметром `filename`

* В коде:

  ```python
  logging.basicConfig(
      filename="/var/log/etl_service.log",
    ...  )
  ```

* `filename="/var/log/etl_service.log"` — означает, что все сообщения уровня INFO и выше (INFO, WARNING, ERROR, CRITICAL) будут записываться в файл `/var/log/etl_service.log`.

* Можно указать **любой путь и имя файла**, куда есть права на запись. Например:

  ```python
  filename="etl.log"
  ```

  тогда лог будет создаваться в папке, откуда запускается скрипт.

* Если путь указать **абсолютный** (`/var/log/etl_service.log`), нужно, чтобы у пользователя, под которым запускается скрипт, были права на запись в эту папку.

---

**Кратко:**

* Можно указать любой путь к файлу.
* Логи пойдут в этот файл.
* Если файла нет, он будет создан (если есть права).
* Если нет прав — будет ошибка записи или лог может не записываться.

---

### Под какими правами VSCode создаёт файл лога?

* VSCode запускает скрипт от имени **пользователя**, под которым ты залогинен в Windows (или в Linux).
* Если ты просто нажал *Run* или запустил отладку, то процесс запускается с твоими правами.
* Значит, права на создание/запись файлов зависят от того, есть ли у твоего пользователя доступ к текущей папке, откуда запускается скрипт.

---

### Почему файл лога не создаётся?

1. **Путь к файлу:**

   * Если указать `filename="etl_service.log"`, лог создастся в **текущем рабочем каталоге процесса**.
   * Этот каталог может отличаться от папки, где лежит скрипт!
   * В VSCode по умолчанию рабочая директория — это обычно **корневая папка проекта**, или та, что указана в настройках запуска (`launch.json`) в параметре `cwd`.

2. **Права на запись:**
3. **Логирование не сработало из-за ошибки:**

---

### Как проверить и отладить?

1. **Вывести текущий рабочий каталог в скрипте:**

   ```python
   import os
   print("Current working directory:", os.getcwd())
   ```

2. **Явно указать абсолютный путь для лога, например:**

   ```python
   filename="C:\\Users\\ТвойПользователь\\etl_service.log"
   ```

   или на Linux:

   ```python
   filename="/tmp/etl_service.log"
   ```

3. **Проверить ошибки записи:**

   * Логгер по умолчанию не выбрасывает ошибки, если файл не создаётся.
   * Можно добавить обработчик ошибок или попробовать записать файл вручную:

   ```python
   try:
       with open("etl_service.log", "a") as f:
           f.write("test log\n")
   except Exception as e:
       print("Ошибка записи файла:", e)
   ```

4. **Проверить, нет ли других вызовов `logging.basicConfig` или конфигураций, которые мешают.**

---

### Как изменить рабочую директорию в VSCode?

1. Открыть файл `.vscode/launch.json` в проекте. Если его нет — создать.

2. Найти (или добавить) конфигурацию запуска, например:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Запуск скрипта",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "cwd": "${fileDirname}"
    }
  ]
}
```

3. Важный параметр — `"cwd"` — задаёт рабочую директорию.

* `${fileDirname}` — папка, где лежит запускаемый файл (скрипт).
* `${workspaceFolder}` — корень открытой папки проекта.

---

### Если запускаешь без `launch.json` (например, прямо из терминала VSCode)

Текущая директория будет та, в которой открыт терминал. Можно её проверить командой:

* `pwd` (Linux/macOS)
* `cd` без параметров (Windows)

---

### Как найти или создать launch.json?

1. Открыть проект в VSCode (через **File > Open Folder**), тогда будет корневая папка проекта.

2. В меню слева снизу найти значок **Run and Debug** (или нажать Ctrl+Shift+D).

3. Если нет настроек запуска — будет кнопка **create a launch.json file** (создать файл запуска). 

4. Выбрать шаблон для Python — VSCode создаст `.vscode/launch.json` в корне проекта.

---

### Триггер `'cron'`
В APScheduler триггер `'cron'` не поддерживает интервал с шагом в секунды — он работает с минутами и выше.

Чтобы запускать задачу **каждые 5 секунд**, надо использовать триггер `'interval'`, например так:

```python
scheduler.add_job(run_etl, 'interval', seconds=5)
```

Это запустит `run_etl` каждые 5 секунд.

---

Если очень хочется, можно с `'cron'` задавать по секундам (например, `second='*/5'`), но не во всех версиях APScheduler это поддерживается стабильно. Лучше `'interval'` для таких частых запусков.

---

### Как заставить logging писать в файл в UTF-8?

В `logging.basicConfig` есть параметр `encoding`, который позволяет задать кодировку файла.

Пример:

```python
logging.basicConfig(
    ...,
    encoding="utf-8"
)
```

---

### Важное замечание:

* Параметр `encoding` появился в Python 3.9 и новее (точно в 3.9+).

---

### Почему `run_etl` не вызывается и не пишет логи?

В режиме отладки, когда установлен брейкпоинт и останавливаешься на строке, основной поток "заморожен" — и задачи в фоне (в других потоках) не смогут нормально выполняться. Они ждут, пока основной поток продолжит работу.

Цикл с `time.sleep()` — правильный подход, чтобы процесс не завершался и фоновые задачи выполнялись.

---

## ✅ Где лежат unit-файлы systemd?

```bash
/etc/systemd/system/etl.service
```

Это правильное место для **пользовательских unit-файлов** (`systemd` будет искать их здесь при `systemctl start` и `enable`).

---

## ✅ Простой `etl.service` unit-файл (корректный шаблон)

Вот отредактированный, рабочий пример:

```ini
[Unit]
Description=Postgres to clickhouse integration service
After=network.target

[Service]
Type=simple
User=admin
WorkingDirectory=/home/admin/systemd/integration/
#ExecStart=/usr/bin/python3 simple_etl.py
ExecStart=/home/admin/systemd/integration/pgchintegration/bin/python simple_etl.py
EnvironmentFile=/home/admin/systemd/integration/.env
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Что нужно обязательно проверить:

1. **`User=your_user`** — заменить на актуального пользователя.
2. **`WorkingDirectory=`** — путь к папке, где лежит скрипт.
3. **`ExecStart=`** — полный путь к Python и скрипту.

Проверить, что:

```bash
which python3
```

выдаёт путь, совпадающий с `/usr/bin/python3`

---

## ✅ Как протестировать?

### 1. Создать и сохранить юнит:

```bash
sudo nano /etc/systemd/system/pgchintegration.service
```

### 2. Перезагрузить systemd, чтобы он увидел новый юнит:

```bash
sudo systemctl daemon-reexec
# или менее жесткий вариант
sudo systemctl daemon-reload
```

### 3. Запустить:

```bash
sudo systemctl start pgchintegration.service
```

### 4. Проверить статус:

```bash
sudo systemctl status pgchintegration.service
```

Если всё работает - статус `active (running)` и логи.

### 5. Посмотреть логи сервиса:

```bash
journalctl -u pgchintegration.service -f
```

(можно просто `-xe` без `-f` для детального просмотра)

### 6. Остановить:

```bash
sudo systemctl stop pgchintegration.service
```

### 7. Добавить в автозагрузку:

```bash
sudo systemctl enable pgchintegration.service
```

---

## ⚠️ Важно

* Если скрипт пишет лог в файл — убедиться, что путь доступен пользователю `User=...`.
* systemd будет логировать `stdout` и `stderr` в `journalctl`, если указано `StandardOutput=journal`.

---

## 👉 Запуск в виртуальном окружении (venv)

## 🔧 Где должны лежать systemd unit-файлы?

### 📁 `/etc/systemd/system/`

Это **правильное место для системных юнитов**, созданных вручную:

* Юниты здесь доступны **всей системе** (всем пользователям).
* Службы запускаются **от имени root** или указанного пользователя (`User=...`).
* Это то место, куда systemd ожидает пользовательские системные службы (например, `pgchintegration.service`).

> ✅ Это **рекомендуемое место**, если нужно, чтобы служба стартовала при загрузке или была частью системных служб (`systemctl start`, `enable`, `status` и т.д.)

---

### 📁 `/etc/systemd/user/`

Это — **для user units**, т.е. служб, которые запускаются **в рамках сессии конкретного пользователя**, а не на уровне системы.

* Запускаются не через обычный `systemctl`, а через `systemctl --user`.
* Работают **только при активной сессии пользователя** (если не настроено иначе).
* Не стартуют автоматически при загрузке системы, если не включён user lingering.

> ⚠️ Это **не то, что нужно**, если необъодимо, чтобы скрипт работал в фоновом режиме на уровне системы.

---

## 📦 Где хранятся юниты системных сервисов (PostgreSQL, ClickHouse и т.п.)?

### 📁 `/usr/lib/systemd/system/`

* Сюда ставятся **юниты, установленные через пакеты** (`rpm`, `deb` и т.д.).
* Например:

  * PostgreSQL: `/usr/lib/systemd/system/postgresql.service`
  * ClickHouse: `/usr/lib/systemd/system/clickhouse-server.service`

> 📌 **Не редактируй их прямо здесь**, если хочешь изменить — лучше копировать в `/etc/systemd/system/` и там переопределять.

---

## 💡 Итого: `pgchintegration.service` должен лежать?

**✅ Рекомендуемое:**

```bash
/etc/systemd/system/etl.service
```

* Доступен всей системе.
* Запускается как системный сервис.
* Позволяет использовать `sudo systemctl start|stop|enable etl.service`.

---

### How to

1. 🔍 Посмотреть, где лежат systemd-юниты.
2. 🛠 Переопределить системный юнит (без переписывания).
3. 📋 Отобразить все активные или установленные юниты.

---

## 🔍 1. Где лежит конкретный юнит?

Например, найти юнит `postgresql.service`:

```bash
systemctl status postgresql
```

В строке `Loaded:` будет написано, откуда загружен юнит:

```text
Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
```

---

## 🛠 2. Как переопределить системный юнит (без прямого редактирования)

Если нужно изменить юнит PostgreSQL (или любой другой из `/usr/lib/systemd/system/`) — не редактируй его прямо.

Вместо этого:

```bash
sudo systemctl edit postgresql
```

Это создаст (или откроет) файл-оверлей:

```bash
/etc/systemd/system/postgresql.service.d/override.conf
```

Можно переопределить только нужные поля. Например:

```ini
[Service]
Environment="PGDATA=/custom/postgres/data"
```

После редактирования — обязательно:

```bash
sudo systemctl daemon-reload
sudo systemctl restart postgresql
```

---

## 📋 3. Как посмотреть все юниты

### Все юниты, которые systemd знает:

```bash
systemctl list-unit-files
```

### Только активные сервисы:

```bash
systemctl list-units --type=service
```

### Найти конкретный юнит:

```bash
systemctl list-units | grep etl
systemctl list-unit-files | grep postgres
```

---

## 📍 Где systemd ищет юниты (в порядке приоритета)

```bash
/etc/systemd/system/               # пользовательские системные юниты
/run/systemd/system/              # временные (напр. сгенерированные)
/usr/lib/systemd/system/          # установленные пакетом
```

И переопределения:

```bash
/etc/systemd/system/<service>.service.d/override.conf
```

---


## Запуск через виртуальное окружение (venv)

---

### ✅ Шаг 1: Создать виртуальное окружение и установить зависимости

Допустим, проект находится по пути:

```bash
/home/admin/systemd/integration
```

### Создаем виртуальное окружение:

```bash
cd /home/admin/systemd/integration
python3 -m venv pgchintegration
```

### Активируем и ставим нужные пакеты:

```bash
source pgchintegration/bin/activate
pip install apscheduler psycopg2-binary clickhouse-connect
# и всё, что ещё нужно
```

После установки убедиться:

```bash
(pgchintegration) $ which python
# ~/systemd/integration/pgchintegration/bin/python
```

---

### ✅ Шаг 2: Обновить unit-файл etl.service

```bash
sudo nano /etc/systemd/system/etl.service
```

И изменяем так:

```ini
[Unit]
Description=Postgres to clickhouse integration service
After=network.target

[Service]
Type=simple
User=admin
WorkingDirectory=/home/admin/systemd/integration/
#ExecStart=/usr/bin/python3 simple_etl.py
ExecStart=/home/admin/systemd/integration/pgchintegration/bin/python simple_etl.py
EnvironmentFile=/home/admin/systemd/integration/.env
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

🔧 Ключевое изменение:
**`ExecStart`** теперь запускает Python из виртуального окружения.

---

### ✅ Шаг 3: Перезапускаем и проверяем

```bash
sudo systemctl daemon-reload
sudo systemctl restart pgchintegration.service
sudo systemctl status pgchintegration.service
```

---

### 👁 Как убедиться, что используется venv?

1. **Можно добавить в `simple_etl.py` запись пути Python:**

```python
import sys
import logging
logging.info(f"Python path: {sys.executable}")
```

После запуска сервиса, в логе будет:

```
Python path: /systemd/integration/pgchintegration/bin/python
```

2. **Или, если добавить `print(sys.executable)` — можно увидеть в journalctl:**

```bash
journalctl -u pgchintegration.service -f
```

3. **Проверить, что нужные пакеты работают (например, `apscheduler`) — если бы он не был установлен в venv, сервис бы упал.**

---

## 🧼 Дополнительно (опционально):

Если нужно передавать переменные окружения (например, `PYTHONPATH`, `ENV`, API-ключи и т.д.), можно добавить:

```ini
Environment="ENV=production"
Environment="PYTHONUNBUFFERED=1"
```

Либо прописать `.env` файл и загрузить через `EnvironmentFile=`.

---

## Ошибка `Permission denied` при запуске Python из виртуального окружения

Очень распространённая, особенно при использовании systemd и нестандартных путей.

---

### 🔎 Ошибка:

```
Failed to locate executable .../bin/python: Permission denied
```

Означает, что **systemd не может выполнить файл**, несмотря на то, что он существует.

---

### 🧭 Что нужно проверить:

#### 1. ✅ Права на сам файл `/home/admin/systemd/integration/pgchintegration/bin/python`

Убедись, что файл:

* **существует**
* **исполняемый**
* **читаем для пользователя**, под которым запускается сервис (`User=...`)

Проверить:

```bash
ls -l /home/admin/systemd/integration/pgchintegration/bin/python
```

Ожидаем:

```text
-rwxr-xr-x 1 admin admin ... python
```

Если нет `x` — добавить:

```bash
chmod +x /home/admin/systemd/integration/pgchintegration/bin/python
```

---

#### 2. ✅ Права на все родительские директории

Systemd не может войти в директорию, если одна из родительских папок **не исполняемая (без `x`)**.

Проверить:

```bash
ls -ld /home
ls -ld /home/admin
ls -ld /home/admin/systemd
ls -ld /home/admin/systemd/integration
```

Они **все** должны быть `drwxr-xr-x` или хотя бы `--x` для нужного пользователя.

Если `User=admin` в юните — нормально. Но если юнит запускается от `root`, и одна из папок не исполняемая для `root`, будет `Permission denied`.

---

#### 3. ✅ Селинукс (если включён)

Если в системе включён **SELinux**, он может блокировать выполнение файлов из «неразрешённых» директорий, например: `/home/admin/...`

Проверить статус:

```bash
getenforce
```

Если ответ `Enforcing` — это почти наверняка причина.

🔧 Варианты решения:

* Временно отключить (для теста):

  ```bash
  sudo setenforce 0
  ```

* Или **использовать стандартные каталоги** (например, `/opt/pgchintegration`), которые уже разрешены для systemd.

---

#### 4. ✅ Исправить `ExecStart` для отладки

Пока отлаживаешь, можешь временно заменить `ExecStart` на скрипт-обёртку:

Создай файл `/home/admin/systemd/integration/start.sh`:

```bash
#!/bin/bash
echo "Запускается из $(which python)"
/home/admin/systemd/integration/pgchintegration/bin/python simple_etl.py
```

Сделать его исполняемым:

```bash
chmod +x start.sh
```

И в `etl.service`:

```ini
ExecStart=/home/admin/systemd/integration/start.sh
```

Это поможет убедиться, что проблема именно в доступе к интерпретатору.

---

### ✅ Подводим итог

* `Permission denied` чаще всего:

* Неправильные права на исполняемый файл или директории.
* SELinux блокирует запуск.
* Убедиться, что все каталоги до `python` доступны (`+x`).
* Для SELinux — временно отключить для теста или переместить окружение в безопасный путь (`/opt/...`).
* Скрипт-обёртка (`start.sh`) — хорошее средство отладки.

---


### Выяснили причину — да, **SELinux** довольно часто мешает запуску сервисов в пользовательских директориях.

---

## ⚠️ Варианты отключения SELinux

**Временно** или **постоянно** отключить SELinux:

---

### 🔧 Вариант 1: Временно (до перезагрузки)

Для отладки и проверки:

```bash
sudo setenforce 0
```

Проверить:

```bash
getenforce
# → Permissive
```

> В режиме `Permissive` SELinux **не блокирует**, а только **логирует** нарушения.

Чтобы снова включить:

```bash
sudo setenforce 1
```

---

### 🧨 Вариант 2: Полностью отключить SELinux (на постоянку)

> ⚠️ Делать это можно только если понимаешь, что **теряешь уровень защиты системы**. Лучше — настроить нужные политики (см. ниже), но если отключать, то так:

#### Шаг 1: Открыть конфиг

```bash
sudo nano /etc/selinux/config
```

#### Шаг 2: Найти строку:

```ini
SELINUX=enforcing
```

#### Шаг 3: Заменить на:

```ini
SELINUX=disabled
```

#### Шаг 4: Перезагрузить систему:

```bash
sudo reboot
```

---

## ✅ Альтернатива — оставить SELinux, но разрешить запуск

Если хочешь **не отключать SELinux**, а просто разрешить запуск Python в нужной директории, можно сделать так:

```bash
sudo chcon -Rt bin_t /home/admin/systemd/integration/pgchintegration/bin
```

Или назначить безопасный контекст:

```bash
sudo chcon -t bin_t /home/admin/systemd/integration/pgchintegration/bin/python
```

Или переместить виртуальное окружение в `/opt/`:

```bash
/opt/pgchintegration/
```

Папки в `/opt` или `/usr/local` уже имеют правильные контексты SELinux.

---

## ✅ Цель: дать systemd разрешение запускать Python из venv, расположенного, например, в `/home/admin/...`

Настроим SELinux **правильно**, чтобы сервис работал без отключения безопасности всей системы. Это считается **наилучшей практикой** на проде.

---

### 🔎 Шаг 1: Проверим текущие ошибки SELinux

В SELinux все блокировки логируются. Самый простой способ — использовать `ausearch` или `journalctl`.

Если установлен `setroubleshoot-server`, можно использовать:

```bash
sudo sealert -a /var/log/audit/audit.log
```

Либо через `ausearch`:

```bash
sudo ausearch -m avc -ts recent
```

> Там будут строки вроде:
> `type=AVC msg=audit(…): avc: denied { execute } for pid=… comm="(python)" ...`

---

### 🛠 Шаг 2: Дать временное разрешение через `chcon`

Это временное изменение, **до перезагрузки**.

```bash
sudo chcon -t bin_t /home/admin/systemd/integration/pgchintegration/bin/python
```

Если нужно разрешить все файлы внутри:

```bash
sudo chcon -R -t bin_t /home/admin/systemd/integration/pgchintegration/bin/
```

---

### 💾 Шаг 3: Сделать метки постоянными (через `semanage`)

> Для этого нужно установить утилиту `policycoreutils-python-utils`:

```bash
sudo dnf install policycoreutils-python-utils
```

Теперь назначим **постоянную метку**:

```bash
sudo semanage fcontext -a -t bin_t "/home/admin/systemd/integration/pgchintegration/bin(/.*)?"
sudo restorecon -Rv /home/admin/systemd/integration/pgchintegration/bin/
```

---

### ✅ Проверим:

1. Перезапусти systemd:

```bash
sudo systemctl daemon-reload
sudo systemctl restart pgchintegration.service
```

2. Убедись, что работает:

```bash
sudo systemctl status pgchintegration.service
```

3. И что SELinux не жалуется:

```bash
sudo ausearch -m avc -ts recent
```

---

## 🧠 Альтернатива: положить venv в "белую зону"

Вместо `/home/admin/...`, переместить виртуальное окружение в `/opt`, например:

```bash
sudo mv /home/admin/systemd/integration/pgchintegration /opt/pgchintegration
```

И прописать в юнит-файле:

```ini
ExecStart=/opt/pgchintegration/bin/python simple_etl.py
```

У `/opt` уже корректные контексты SELinux для запуска программ.

---

## 📌 Как создать собственную SELinux-политику**

Чтобы системный сервис имел разрешение на выполнение Python из виртуального окружения или выполнение любой другой ограниченной операции — **без отключения SELinux и без взлома системы метками.**

Создаём свою политику SELinux, которая:

* **разрешает конкретному сервису** (например, `pgchintegration.service`)
* **выполнять Python** из указанного каталога (например, `/home/admin/systemd/integration/...`)
* и **делает это безопасно** — без снятия SELinux, `chcon`, `disable`, и т.п.

---

## 🧰 Что нужно:

Установим нужные инструменты:

```bash
sudo dnf install policycoreutils-devel checkpolicy selinux-policy-devel -y
```

(на CentOS/Alma/RHEL — это всё есть в стандартных репах)

---

## ✅ Шаг 1: Включим логирование нарушения (если не включено)

Запустим сервис, чтобы SELinux **залогировал отказ**:

```bash
sudo systemctl restart pgchintegration.service
```

---

## ✅ Шаг 2: Найдем последние блокировки SELinux

```bash
sudo ausearch -m avc -ts recent
```

Или:

```bash
sudo journalctl -t setroubleshoot
```

---

## ✅ Шаг 3: Создадим кастомную SELinux-политику автоматически

Есть утилита `audit2allow`, которая по логам **сама сгенерирует нужную политику**:

```bash
sudo ausearch -c 'pgchintegration' --raw | audit2allow -M pgchintegration_policy
```

> Здесь `'pgchintegration'` — это `comm=` из журнала SELinux (обычно имя команды или бинарника).

---

## ✅ Шаг 4: Установим сгенерированную политику

```bash
sudo semodule -i pgchintegration_policy.pp
```

Теперь SELinux будет **разрешать именно эту операцию**, только для этой программы/пути.

---

## 🛡 Пример вывода команды `audit2allow`

Вот как может выглядеть автосгенерированная политика:

```bash
module pgchintegration_policy 1.0;

require {
    type unconfined_t;
    type admin_home_t;
    class file { execute open read };
}

# Allow access
allow unconfined_t admin_home_t:file { execute open read };
```

Её можно вручную подправить и собрать:

```bash
checkmodule -M -m -o pgchintegration_policy.mod pgchintegration_policy.te
semodule_package -o pgchintegration_policy.pp -m pgchintegration_policy.mod
semodule -i pgchintegration_policy.pp
```

---

## 📋 Как проверить, что политика работает?

1. Перезапустить сервис:

```bash
sudo systemctl restart pgchintegration.service
```

2. Убедиться, что ошибок SELinux больше нет:

```bash
sudo ausearch -m avc -ts recent
```

3. Проверка установленных политик:

```bash
semodule -l | grep pgchintegration
```

---

## ❗️Важно

* Такие политики действуют **только для конкретных операций** — SELinux всё ещё работает и защищает систему.
* Политика привязывается к типу (`type`) и пользователю — можно написать более узкую политику для одного сервиса.
* Это **чистый, безопасный** путь — как делают в продакшене в банках, телекомах и госсекторах.

---

## 🎯 Создадим ручную SELinux-политику (.te) — максимально чисто и контролируемо

Это дает гибкость и понимание, **как разрешать только то, что действительно нужно**, не ломая остальную защиту системы.

Цель:

Разрешить сервису (например, `pgchintegration.service`) запуск Python-интерпретатора, расположенного в `/home/admin/systemd/integration/pgchintegration/bin/python`, **без отключения SELinux**, но и **без избыточных полномочий**.

---

## ✅ План:

1. Соберём `.te`-файл — текстовое описание политики.
2. Скомпилируем её в `.pp`.
3. Установим её через `semodule`.

---

## 🛠 Пример .te-файла

Создай файл `pgchintegration.te`:

```bash
nano pgchintegration.te
```

Пример содержимого:

```te
module pgchintegration 1.0;

require {
    type unconfined_t;
    type user_home_t;
    class file { execute open read };
}

# Разрешить выполнение python из каталога в home
allow unconfined_t user_home_t:file { execute open read };
```

> 💡 Здесь:
>
> * `unconfined_t` — это тип, под которым чаще всего работает `systemd` или пользователь.
> * `user_home_t` — тип файлов в `/home/...`, назначаемый SELinux автоматически.

---

## ✅ Компиляция и установка

1. Скомпилируй модуль:

```bash
checkmodule -M -m -o pgchintegration.mod pgchintegration.te
```

2. Упакуй его в модуль:

```bash
semodule_package -o pgchintegration.pp -m pgchintegration.mod
```

3. Установи:

```bash
sudo semodule -i pgchintegration.pp
```

---

## 📋 Проверка

* Убедись, что политика установлена:

```bash
semodule -l | grep pgchintegration
```

* Перезапусти сервис:

```bash
sudo systemctl restart pgchintegration.service
```


* Убедись, что ошибок SELinux нет:

```bash
sudo ausearch -m avc -ts recent
```

---

## 🧠 Как определить нужные `type` и `class`?

Если хочешь создать более специфичную политику (например, разрешить запуск **только этого скрипта**, только этому сервису), используй:

```bash
ls -Z /home/admin/systemd/integration/pgchintegration/bin/python
```

Пример вывода:

```
-rwxr-xr-x. admin admin unconfined_u:object_r:user_home_t:s0 python
```

Здесь:

* `user_home_t` — это и есть нужный `type`
* Если будет `admin_home_t`, то его укажешь в `.te`

---

## 🚀 Для продакшена (точечная политика)

Хочешь жёсткую политику? Например: разрешить запуск `python` только **этому конкретному systemd-сервису**?

1. Сначала выясни контекст:

```bash
ps -eZ | grep pgchintegration
```

или

```bash
sudo systemctl show pgchintegration.service --property=ExecContext
```

2. Напиши `.te`, где `allow YOUR_SERVICE_CONTEXT` — это может быть `systemd_service_t`, `init_t` или даже кастомный, если сам назначишь через `semanage`.

---


Отлично, ты дал именно то, что нужно — теперь можно **точно и безопасно разрешить SELinux запуск `python` из каталога `/home/admin/...` для твоего systemd-сервиса.**

---

## 📊 Разбор:

### 🔹 Контекст Python-процесса:

```
system_u:system_r:pcp_pmcd_t:s0
```

Это `pcp_pmcd_t` — относится к Performance Co-Pilot, **не к твоему сервису.**

> Нам нужно разрешить выполнение Python **от systemd**, под контекстом `system_u:system_r:init_t:s0` или `systemd_t`, если твой сервис запускается напрямую через `systemctl`.

---

### 🔹 Контекст файлов:

```
unconfined_u:object_r:user_home_t:s0
```

Это значит, твой `python`, `simple_etl.py` и `pgchintegration` лежат в **домашнем каталоге**, с типом `user_home_t`, который **по умолчанию запрещён к запуску от systemd**.

---

## ✅ Что делаем: создаём SELinux-модуль, разрешающий выполнение файлов с типом `user_home_t` для сервисов с типом `init_t` (типичный systemd).

---

## 🛠 Шаг 1: Создай `.te`-файл

```bash
nano allow_pgchintegration.te
```

Содержимое:

```te
module allow_pgchintegration 1.0;

require {
    type user_home_t;
    type init_t;
    class file { read open execute execute_no_trans };
}

# Разрешить systemd (init_t) выполнять python и скрипты из каталога с типом user_home_t
allow init_t user_home_t:file { read open execute execute_no_trans };
```

---

## 🛠 Шаг 2: Собери и установи модуль

```bash
checkmodule -M -m -o allow_pgchintegration.mod allow_pgchintegration.te
semodule_package -o allow_pgchintegration.pp -m allow_pgchintegration.mod
sudo semodule -i allow_pgchintegration.pp
```

---

## ✅ Шаг 3: Перезапусти сервис и проверь

```bash
sudo systemctl restart pgchintegration.service
sudo systemctl status pgchintegration.service
sudo ausearch -m avc -ts recent
```

Если всё сделано правильно — **ошибок больше не будет**, а SELinux будет спокойно пропускать запуск скрипта из твоей директории.

---

## 📋 Пояснение по `execute_no_trans`

* `execute` — разрешает выполнить файл.
* `execute_no_trans` — запрещает SELinux **переводить в другой контекст**, т.е. оставляет процесс под тем же, что и был (важно для systemd).

---

Можно **ещё ужесточить**: разрешить только конкретный файл или только конкретному юзеру.

---

## ✅ Перенос проекта в `/opt/`

Это правильный, безопасный и "по-админски" чистый путь при использовании SELinux, systemd и виртуальных окружений.

Почему `/opt/` — лучше:

* SELinux **разрешает исполнение** из `/opt` (типы вроде `usr_t` или `bin_t`)
* Systemd **без проблем** запускает процессы из `/opt`
* `/opt/` предназначен для размещения локальных приложений, в отличие от домашнего каталога

---

## 🛠 План действий:

1. 📁 Перенос проекта в `/opt/`
2. 🐍 Обновление путей к виртуальному окружению
3. 🧾 Правка `etl.service`
4. 🚀 Перезапуск systemd и проверка

---

### 📁 1. Перенеси проект

```bash
sudo mkdir -p /opt/pgchintegration
sudo cp -r /home/admin/systemd/integration/* /opt/pgchintegration/
sudo chown -R admin:admin /opt/pgchintegration
```

💡 Убедись, что структура такая:

```
/opt/pgchintegration/
├── simple_etl.py
├── pgchintegration/       <-- venv
│   └── bin/python
├── run_pgchintegration.sh (если используешь)
```

---

### 🐍 2. Убедись, что `venv` рабочий

```bash
cd /opt/pgchintegration
source pgchintegration/bin/activate
python simple_etl.py  # Проверь, работает ли вручную
deactivate
```

---

### 🧾 3. Обнови `pgchintegration.service`

Открой:

```bash
sudo nano /etc/systemd/system/pgchintegration.service
```

Пример конфигурации:

```ini
[Unit]
Description=ETL Background Service from /opt
After=network.target

[Service]
Type=simple
User=admin
WorkingDirectory=/opt/pgchintegration
ExecStart=/opt/pgchintegration/pgchintegration/bin/python simple_etl.py
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

---

### 🔄 4. Перезапусти сервис

```bash
sudo systemctl daemon-reload
sudo systemctl restart pgchintegration.service
sudo systemctl status pgchintegration.service
```

---

### 📋 5. Проверка

* Проверить логи сервиса:

  ```bash
  journalctl -u pgchintegration.service -f
  ```

* Проверить SELinux:

  ```bash
  getenforce         # Должно быть Enforcing
  sudo ausearch -m avc -ts recent
  ```

**Если всё молчит — значит SELinux доволен** и `python` запускается из безопасного каталога.

---

### 🧠 Совет

Можешь создать `/opt/pgchintegration/venv` и там держать окружение, а код в `/opt/pgchintegration/app` — но это уже на вкус и стиль.

---

## эГотовый bash-скрипт**

Который автоматизирует:

1. Создание каталога `/opt/pgchintegration`
2. Копирование проекта из `~/systemd/integration/`
3. Настройку прав
4. Создание виртуального окружения `venv`
5. Установку зависимостей (если нужно)
6. Создание `systemd`-юнита
7. Перезапуск systemd и запуск сервиса

---

## 📄 Скрипт: `setup_pgchintegration.sh`

Сохрани его как `setup_pgchintegration.sh`, сделай исполняемым и запусти от `sudo`.

```bash
#!/bin/bash

set -e

APP_NAME="pgchintegration"
APP_USER="admin"
APP_DIR="/opt/$APP_NAME"
SRC_DIR="/home/$APP_USER/systemd/integration"
VENV_DIR="$APP_DIR/venv"
SERVICE_FILE="/etc/systemd/system/$APP_NAME.service"
PYTHON_BIN="/usr/bin/python3"

echo "📁 Создаём каталог $APP_DIR..."
mkdir -p "$APP_DIR"
cp -r "$SRC_DIR/"* "$APP_DIR/"
chown -R "$APP_USER:$APP_USER" "$APP_DIR"

echo "🐍 Создаём виртуальное окружение в $VENV_DIR..."
$PYTHON_BIN -m venv "$VENV_DIR"

echo "📦 Устанавливаем зависимости (если есть requirements.txt)..."
if [[ -f "$APP_DIR/requirements.txt" ]]; then
    "$VENV_DIR/bin/pip" install --upgrade pip
    "$VENV_DIR/bin/pip" install -r "$APP_DIR/requirements.txt"
fi

echo "🧾 Создаём systemd unit: $SERVICE_FILE"
cat <<EOF > "$SERVICE_FILE"
[Unit]
Description=ETL Background Service from /opt
After=network.target

[Service]
Type=simple
User=$APP_USER
WorkingDirectory=$APP_DIR
ExecStart=$VENV_DIR/bin/python simple_etl.py
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

echo "🔄 Перезагружаем systemd и включаем сервис..."
systemctl daemon-reload
systemctl enable "$APP_NAME.service"
systemctl restart "$APP_NAME.service"

echo "✅ Готово! Сервис $APP_NAME запущен."
systemctl status "$APP_NAME.service" --no-pager
```

---

## 🛠 Как использовать

1. Скопируй скрипт:

```bash
nano setup_pgchintegration.sh
```

2. Вставь содержимое, сохрани и сделай исполняемым:

```bash
chmod +x setup_pgchintegration.sh
```

3. Запусти:

```bash
sudo ./setup_pgchintegration.sh
```

---

## ✅ Результат

После запуска:

* Код и `venv` лежат в `/opt/pgchintegration/`
* Сервис называется `pgchintegration.service`
* Лог можно смотреть через:

```bash
journalctl -u pgchintegration.service -f
```

---
