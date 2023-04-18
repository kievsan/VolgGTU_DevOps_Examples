# Лабораторная №? "Знакомство с работой облачных сервисов Prometheus и Grafana"

Итак, в данной работе приводится пример базовой настройки мониторинга состояния операционной системы. Мониторинг настраивается с помощью следующих модулей:

- Node Exporter
- Prometheus
- Grafana

Сейчас рассмотрим каждый модуль подробнее.
Node Exporter - небольшое приложение, собирающее метрики операционной системы и предоставляющее к ним доступ по HTTP.
Prometheus - СУБД, которое собирает данные с одного или нескольких экземпляров Node Exporter.
Grafana - графическая утилита, отображает данные из Prometheus в виде графиков и диаграмм, организованных в дашборды.

## Этап 1. Установка утилит

Итак, для базовых задач не нужно настраивать каждый модуль т.к. есть готовые. В данном случае воспользуемся Docker. Система контейнеров позволяет работать с готовыми модулями и быстро настроить систему мониторинга.

Чтобы использовать модули, клонируем репозиторий:

```
git clone https://github.com/digitalstudium/grafana-docker-stack.git
```

Т.к. используется Docker, скачаем программу:

```
apt update && apt install docker.io
```

Инициализируем Docker Swarm
```
docker swarm init
```

## Этап 2. Базовая настройка сервисов

Запустим файл из скачанного репозитория (файл приложен к л.р.):

```
docker stack deploy -c grafana-docker-stack/docker-compose.yml monitoring
```


Далее открываем браузер и перейдём по адресу - <your ip address>:3000 (именно на этом порту запущена Grafana). 
Дефолтный пароль/логин - admin/admin 
После первого входа система потребует изменить пароль
  
  Вот и стандратный UI Grafana:
  
  ![image](https://user-images.githubusercontent.com/66121979/232743040-c4cfb12c-56a1-46ca-9e11-257b9353efff.png)
 
Чтобы добавить Prometheus, нужно зайти в Configuration->Data sources, в строке Url ввести адрес - http://prometheus:9090
  
Итак, БД для хранения метрик добавлена.
  

## Этап 3. Приложение для мониторинга

Чтобы добавить Node Explorer, на сайте https://grafana.com/grafana/dashboards/1860-node-exporter-full/ скачиваем JSON-файл, открываем его и копируем содержимое.
  
В Grafana переходим в Dashboards->Manage и нажимаем Import. Далее в окно "Import via panel json" вставляем скопированное содержимое файла.
  
  ![image](https://user-images.githubusercontent.com/66121979/232746076-5d0a2e1d-6553-43f3-8b71-04b7351edd4c.png)

Вот теперь есть дашборд с метрикам состояния системыб но данные пока  что не поступают. 
  
Для получения данных из Node Explorer нужно изменить файл prometheus.yml, добавив туда несколько строк:
  
 ```
  static_configs 
    - targets ['localhost:9090'] // после этих строк добавить привязку к Node Explorer
  
  
  - job_name: 'node-explorer'
  
  static_configs 
    - targets ['node-explorer:9100]
  
  ```
  
  Последние штрихи:
  
  ```
  docker ps | grep prometheus // команда на поиск id контейнера prometheus
  
  docker kill -s SIGHUP <id_process> // команда на перезапуск контейнера
  
  ```
  
  Вот и рабочий дашборд с метриками операционной системы
  
  
  ![image](https://user-images.githubusercontent.com/66121979/232749754-e582ad9a-4b60-40ba-b4ca-9d7ea9dfd3bd.png)
