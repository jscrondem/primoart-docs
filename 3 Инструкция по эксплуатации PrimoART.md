# Подготовка сервера к установке

## Проверка конфигурации primoart-api

`PARobot`в данной инструкции поднимается на том же сервере, что и `primoart-api`. Подниматься `pa-robot` будет также в системе контейнеризации Docker.

Поэтому, следует подключить контейнер с агентом в ту же `docker network,` что и контейнеры `primoart-api`. Для этого узнаем имя `docker network `для `primoart-api`

```
sudo docker network list
```

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-5.png =x131)

Интересует сеть `primoart-network`. Далее будет использоваться это имя

---

## Скачивание исходников

Склонировать репозиторий (или получить архив с ним из поставки)

```
git clone git@github.com:jscrondem/primoart-stm-agent.git
```

Переключиться на ветку `main`

```
git checkput main
```

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-1.png =x173)

Директория с проектом должна иметь следующий вид:

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie.png =x537)

---

## Настройка конфигурации

Перейти в директорию `primoart-stm-agent/parobot`

Скопировать файл `config.example.json `рядом в директории с именем `config.json`

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-2.png =x156)

Открыть скопированный файл `config.json` в любом удобном редакторе, например `nano`

```
nano config.json
```

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-3.png =x302)

*Описание полей:*

**api_url** - URL `pa-synth`, эндопоинта `primoart-api` , с которым будет происходить взаимодействие

**robot_uuid** - Сгенерированный UUID робота

**robot_name** - Имя робота

**update_shedule** - Расписание получения новых или обновления старых описаний транзакций от `primoart-api`

**db_path** - Путь до локальной базы данных. Путь указан для разворачивания сервиса в Docker контейнере

**save_trace** - Булевый параметр, если установлен в true, то по пути `trace_folder` будут сохраняться трейсы Playwright

**trace_folder** - Путь до директории, где будут лежат сохранные трейсы Playwright. Путь указан для разворачивания сервиса в Docker контейнере

---

## Подготовка к сборке Docker контейнера pa-robot

Перейти в директорию `deploy/`

Открыть в любом удобном редакторе, например `nano`файл `docker-compose.yml`

Поменять во всем файле имя `docker network` с `app-network` на имя сети, в которой работают контейнеры `primoart-api` (`primoart-network`)

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-8.png =x600)

Перейти в директорию `deploy/pa_arobot/`и добавить права на выполнение файла `init.sh`

```
chmod +x init.sh
```

---

# Установка Docker контейнера с PARobot

Перейти в директорию `deploy` и поднять docker контейнер с pa-robot

```
sudo docker-compose up -d
```

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-9.png =x403)Проверить, что контейнер успешно поднят

```
sudo docker ps -a
```

# ![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-10.png =x342)

Остановить контейнер на время (`id` контейнера взять из вывода предыдущей команды)

```
sudo docker stop da4578148021
```

---

# Проверка взаимодействия PARobot и Primoart API

Задача - загрузить транзакцию со следующим кодом в систему, чтобы она выполнялась каждую 1 минуты на всех агентах `PARobot`

```
from pasdk import *

with step(step_name="Create tasks"):
    page.goto("https://demo.playwright.dev/todomvc/")
    page.goto("https://demo.playwright.dev/todomvc/#/")
    page.get_by_placeholder("What needs to be done?").click()
    page.get_by_placeholder("What needs to be done?").fill("task1")
    page.get_by_placeholder("What needs to be done?").press("Enter")
    page.get_by_placeholder("What needs to be done?").fill("task2")
    page.get_by_placeholder("What needs to be done?").press("Enter")
    page.get_by_placeholder("What needs to be done?").fill("task3")
    page.get_by_placeholder("What needs to be done?").press("Enter")

with step(step_name="Check tasks"):
    page.locator("li").filter(has_text="task2").get_by_label("Toggle Todo").check()
    page.locator("li").filter(has_text="task3").get_by_label("Toggle Todo").check()
    page.get_by_role("link", name="Completed").click()
    expect(page.get_by_text("task3")).to_be_visible()
    #expect(page.get_by_text("task3")).not_to_be_visible()
    expect(page.get_by_text("task2")).to_be_visible()

with step(step_name="Clear"):
    page.get_by_role("button", name="Clear completed").click()
    page.get_by_role("link", name="Active").click()
    expect(page.get_by_text("All Active Completed")).to_be_visible()
```

---

## Загрузка транзакции в Primoart API

Загрузим тестовую транзакцию в `primoart-api`

Для этого следует открыть swagger эндпоинта `pa_core` по адресу http://127.0.0.1:8030/docs

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-11.png =x600)

Открыть раздел `/api/v1/transactions/` и нажать ***Try it out.*** Открывается поле для ввода описания транзакции. Вставляем туда следующий код:

```
[
  {
    "uuid": "123e5327-e89b-12d3-a456-426614179800",
    "name": "transaction-test0",
    "cron": "*/1 * * * *",
    "code": "from pasdk import *\n\nwith step(step_name=\"Create tasks\"):\n    page.goto(\"https://demo.playwright.dev/todomvc/\")\n    page.goto(\"https://demo.playwright.dev/todomvc/#/\")\n    page.get_by_placeholder(\"What needs to be done?\").click()\n    page.get_by_placeholder(\"What needs to be done?\").fill(\"task1\")\n    page.get_by_placeholder(\"What needs to be done?\").press(\"Enter\")\n    page.get_by_placeholder(\"What needs to be done?\").fill(\"task2\")\n    page.get_by_placeholder(\"What needs to be done?\").press(\"Enter\")\n    page.get_by_placeholder(\"What needs to be done?\").fill(\"task3\")\n    page.get_by_placeholder(\"What needs to be done?\").press(\"Enter\")\n\nwith step(step_name=\"Check tasks\"):\n    page.locator(\"li\").filter(has_text=\"task2\").get_by_label(\"Toggle Todo\").check()\n    page.locator(\"li\").filter(has_text=\"task3\").get_by_label(\"Toggle Todo\").check()\n    page.get_by_role(\"link\", name=\"Completed\").click()\n    expect(page.get_by_text(\"task3\")).to_be_visible()\n    #expect(page.get_by_text(\"task3\")).not_to_be_visible()\n    expect(page.get_by_text(\"task2\")).to_be_visible()\n\nwith step(step_name=\"Clear\"):\n    page.get_by_role(\"button\", name=\"Clear completed\").click()\n    page.get_by_role(\"link\", name=\"Active\").click()\n    expect(page.get_by_text(\"All Active Completed\")).to_be_visible()",
    "target_duration": 1.0,
    "metadata": {
      "location": "location01"
    },
    "description": "Test transaction"
  }
]
```

*Описание полей:*

**uuid** - Сгенерированный UUID транзакции

**name** - Имя транзакции (`transaction-test0`)

**cron** - Строка в формате cron, указывающая расписание запуска данной транзакции

**code** - Отформатированный код транзакции, где перенос строки заменен на `“\n“`

**target_duration** - Желаемое время выполнение транзакции, с.

**metadata** - Некая метадата для выполнения данной транзакции,

**description** - Описание транзакции

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-12.png =x600)

Нажимаем `Execute` , проверяем что в поле Response body `status: ”success”`

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-13.png =x600)

Зайти в Grafana http://127.0.0.1:3000/dashboards (логин и пароль по умолчанию - admin/admin)

Открыть дашборд `Список транзакций DEV`

Проверить, что добавленная нами транзакция появилась в списке загруженных транзакций![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-14.png =x600)![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-15.png =x600)

Нажать на имя данной транзакции - откроется новая страница с информацией о запусках данной транзакции на агентах. Позже сюда надо будет вернуться.

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-16.png =x600)

---

## Проверка выполнений транзакции

Вернуться в терминал и поднять контейнер снова (id созданного контейнера берется из предыдущих шагов)

```
sudo docker start da4578148021
```

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-18.png =x412)Подождать пару минут и проверить логи контейнера

```
 sudo docker logs da4578148021
```

Вывод команды должен быть похож на то, что указано ниже

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-21.png =x363)Желтым видны логи, что агент получил новую транзакцию, красным выделены логи о выполнении транзакции.

Вернуться на страницу в Grafana  с транзакцией. Должно выглядеть примерно следующим образом

![изображение.png](/homepage/primoart/instrukcii/nastrojjka-robota-playwright/.files/izobrazhenie-20.png =x600)

Это показывает, что `PARobot`успешно запустился, получил транзакции от `primoart-api`, выполняет их по расписанию и загружает результаты обратно в `primoart-api`.