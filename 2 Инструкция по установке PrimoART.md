# Инструкция по установке ПО “Primo ART”

## Глоссарий

| Термин  | Описание                                                 |
|---------|----------------------------------------------------------|
| ПО      | Программное обеспечение                                  |
| Продукт | Продукт для мониторинга пользовательского опыта «Primo ART» |

## Введение

### Общие сведения
Настоящая инструкция содержит информацию, необходимую для установки платформы «Primo ART».

## Описание программного обеспечения

### Назначение платформы
Primo ART — это платформа для мониторинга цифрового опыта, объединяющая функции мониторинга реальных пользователей (RUM) и синтетических транзакций (STM). Она предназначена для повышения надежности и доступности критически важных бизнес-приложений, используя искусственный интеллект и машинное обучение для прогнозирования, анализа и оптимизации.

## Установка и настройка

### Системные требования

#### Аппаратные требования
- **Процессор:** Многоядерный, не менее 2.5 ГГц. Рекомендуется Intel Xeon или AMD EPYC.
- **Оперативная память:** Минимум 16 ГБ, рекомендуется от 32 ГБ для крупных развертываний.
- **Дисковое пространство:** Минимум 200 ГБ, рекомендуется SSD и RAID-массивы.
- **Сетевое подключение:** Высокоскоростное (не менее 1 Гбит/с).

#### Программные требования
- **Операционная система:**
  - Linux с поддержкой Docker и Docker Compose.
  - Windows Server 2016 и выше.
- **Дополнительные компоненты:** Docker и Docker Compose.

#### Клиентские требования
Современный веб-браузер: Google Chrome, Mozilla Firefox, Microsoft Edge, Safari.

#### Примечания
- Системные требования могут варьироваться в зависимости от нагрузки.
- Перед развертыванием рекомендуется тестирование в изолированной среде.

### Установка ПО

#### Установка Docker
На сервере с Ubuntu/Debian/Astra выполните:
```bash
sudo apt update
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates
echo "deb https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/repository-main/ 1.7_x86-64 main contrib non-free" | sudo tee -a "/etc/apt/sources.list"
echo "deb https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/repository-update/ 1.7_x86-64 main contrib non-free" | sudo tee -a "/etc/apt/sources.list"
echo "deb https://dl.astralinux.ru/astra/stable/1.7_x86-64/repository-extended/ 1.7_x86-64 main contrib non-free" | sudo tee -a "/etc/apt/sources.list"
sudo sed -i -E "s~^.*\s+cdrom:.*~#\0~" "/etc/apt/sources.list"
sudo apt update
sudo apt-get install -y docker docker-compose
```

#### Загрузка Docker images
Перейдите в директорию, где находятся образы Docker:
```bash
cd docker_images
```
Загрузите каждый Docker образ:
```bash
 sudo docker load < grafana_grafana-oss_latest.tar
 sudo docker load < postgres_13_3.tar
 sudo docker load < pa_core_latest.tar
 sudo docker load < pa_math_latest.tar
 sudo docker load < pa_synth_latest.tar
 ```
Проверьте, что образы были успешно загружены:
```bash
 sudo docker images
  ```
  
#### Установка через Docker Compose
Запустите все контейнеры с помощью Docker Compose, предварительно перейдите в директорию `docker-compose`:
```bash
 sudo docker compose up -d
 ```
Убедитесь, что все три контейнера запущены:
```bash
 sudo docker ps
 ```
 
#### Проверка доступности API микросервисов
Используйте команду `curl` или откройте браузер, чтобы проверить доступность каждого микросервиса по HTTP:
```bash
 curl http://localhost:8020/docs
 curl http://localhost:8030/docs
 curl http://localhost:8040/docs
  ```
  
#### Установка плагина Infinity для Grafana
Разархивируйте архив **yesoreyeram-infinity-datasource-2.10.0.darwin_amd64.zip** из каталога **grafana_plugins**

Скопируйте каталог **yesoreyeram-infinity-datasource** в docker-compose/grafana/data/plugins/

Выполните вход в контейнер
```bash
 sudo docker exec -it grafana-Primo ART bash
   ```
Перейдите в каталог с плагинами 
```bash
 cd /var/lib/grafana/plugins/
   ```
Добавьте права 
```bash
 chmod -R 755 yesoreyeram-infinity-datasource
   ```
Выйдите из контейнера 
```bash
 exit
   ```
Перезапустите контейнер
```bash
 sudo docker restart grafana-Primo ART
```
 
#### Установка плагина Business Forms для Grafana
Разархивируйте архив **volkovlabs-form-panel-4.7.0.zip** из каталога **grafana_plugins**
Скопируйте каталог **volkovlabs-form-panel-4.7.0.zip** в **docker-compose/grafana/data/plugins/**
Выполните вход в контейнер:
```bash
 sudo docker exec -it grafana-Primo ART bash
```
Перейдите в каталог с плагинами:
```bash
cd /var/lib/grafana/plugins/
```

Добавьте права:
```bash
 chmod -R 755 volkovlabs-form-panel
```
Выйдите из контейнера:
```bash
 exit
```

Перезапустите контейнер:
```bash
 sudo docker restart grafana-Primo ART
```
 
#### Настройка расписания Cron
Для избежания переполнения дискового пространства рекомендуется периодически очищать логи контейнера командой 
```bash
 truncate -s 0 /var/lib/docker/containers/*/*-json.log
```
Откройте crontab командой 
```bash
 crontab -e
```
Сделайте новую запись 
```bash
 0 3 * * 0 truncate -s 0 /var/lib/docker/containers/*/*-json.log
```
что означает запуск команды каждое воскресенье в 03:00. При необходимости измените расписание.
