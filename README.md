<h1 align="center">Приветствую вас, гитхаберы</a>
<img src="https://github.com/blackcater/blackcater/raw/main/images/Hi.gif" height="32"/></h1>

![e5c2aa](https://github.com/user-attachments/assets/8c342396-c844-489d-b956-875ed563053a)

Работа будет разделена на 3 пункта

Чтобы начать работу, нужно установить операционную систему на VirtualBox при этом нужно учесть некоторые пункты:

- Иметь образ Linux (https://drive.google.com/file/d/1dwdxyonlkTnOY452-JOaj8GkKKqASfBN/view?usp=sharing)
- Выделить 2+ ядер.
- Выделать 4096+ МБ оперативы.
- Не создавать в сетевых расположения, если они есть, т.к. виртуалка попросту не будет работать.
- При установки операционной системы, нужно будет выбрать английский язык.

Если проблем с установкой виртуальной машины не возникло, то приступаем к первому пункту.

# 1. Установка Grafana Stack с использованием Docker

В этом разделе описывается пошаговый процесс установки и настройки стека Grafana с использованием Docker и Docker Compose. Все команды приведены с пояснениями для лучшего понимания процесса и удобства выполнения.

Этот процесс включает:

Клонирование репозитория с настройками.
Создание необходимых директорий для хранения данных и конфигураций.
Настройку прав доступа для правильной работы сервисов.
Установку последней версии Docker Compose и запуск контейнеров.

Переходим к выполнению команд:

Клонирование репозитория Grafana Stack для Docker с GitHub.

    git clone https://github.com/skl256/grafana_stack_for_docker.git
![изображение](https://github.com/user-attachments/assets/2b007570-2348-4419-8747-52a88b5c687d)

Переход в каталог, куда был склонирован репозиторий.

    cd grafana_stack_for_docker
![изображение](https://github.com/user-attachments/assets/47722b1f-0c08-4065-90d3-389084ca0d33)
    
Создание каталога для конфигурации Grafana внутри общей директории в Docker Swarm.

    sudo mkdir -p /mnt/common_volume/swarm/grafana/config

Создание нескольких директорий для хранения данных конфигурации и информации для Grafana, Prometheus, Loki, Promtail.

    sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}
   
Изменение владельца этих директорий на текущего пользователя с помощью команд `id -u` и `id -g`, которые возвращают UID и GID текущего пользователя.   

    sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}
    
Создание пустого файла `grafana.ini` для дальнейшей конфигурации Grafana. 

    touch /mnt/common_volume/grafana/grafana-config/grafana.ini
    
Копирование всех файлов из локальной директории `config` в созданный каталог конфигурации.   

    cp config/* /mnt/common_volume/swarm/grafana/config/
 ![изображение](https://github.com/user-attachments/assets/66aef55c-dff9-4ef1-862e-cc4087b18c30)
* При выполнение команд от sudo mkdir до cp config/* в консоль не будет ответа, так что если вы ввели команды правильно, не стоит беспокоиться о том, что вы делаете что-то не правильно
   
Переименование файла `grafana.yaml` в `docker-compose.yaml`, который используется для запуска контейнеров Docker через Docker Compose. 

    mv grafana.yaml docker-compose.yaml
![изображение](https://github.com/user-attachments/assets/5d0e1152-5721-4a3d-a9bb-40dad32937b9)

Установка утилиты curl (на CentOS/RHEL) для выполнения HTTP-запросов.

    sudo yum install curl
![изображение](https://github.com/user-attachments/assets/0f22a97b-a242-464f-a992-788fe5f079a5)
    
Использование команды curl для получения последней версии Docker Compose с GitHub API. Фильтрация ответа для извлечения тега с версией.

    COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
![изображение](https://github.com/user-attachments/assets/ef150707-2018-438e-8849-a905474f7a4c)

Загрузка последней версии Docker Compose с официального репозитория GitHub и сохранение её в `/usr/bin/docker-compose`.     

    sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
![изображение](https://github.com/user-attachments/assets/f3bca54a-cebf-4c40-b6cf-63bdd55f2879)
    
Предоставление прав на выполнение файла `docker-compose`.  

    sudo chmod +x /usr/bin/docker-compose
![изображение](https://github.com/user-attachments/assets/8321e614-43e4-4819-b7b2-70ab6dbfcd95)
    
Проверка установленной версии Docker Compose.

    docker-compose --version
![изображение](https://github.com/user-attachments/assets/6dfc60d5-e8da-44db-9d4f-21b9e71f2490)
    
Установка wget для скачивания файлов.   

    sudo yum install wget
![изображение](https://github.com/user-attachments/assets/e5d6d6ff-dafe-4264-9f52-023f8b505449)
    
Скачивание файла репозитория Docker CE для CentOS и размещение его в директории `/etc/yum.repos.d/`.

    sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo
![изображение](https://github.com/user-attachments/assets/99a2fdda-2b2e-4a7e-8992-75be5a4ac0d3)
    
Установка Docker Engine (docker-ce), клиентских инструментов Docker (docker-ce-cli) и контейнерного рантайма containerd.

    sudo yum install docker-ce docker-ce-cli containerd.io
![изображение](https://github.com/user-attachments/assets/8c2c3277-fa72-44ce-86fc-879abf88aae7)
    
Включение и немедленный запуск службы Docker.

    systemctl enable docker --now
    
Запуск сервисов, описанных в `docker-compose.yaml`, в фоновом режиме.

    sudo docker compose up -d 
![изображение](https://github.com/user-attachments/assets/8505dc5f-14cf-4638-b0f2-48e469f589b6)

Самое важное, по пути: /grafana_stack_for_docker/ в файле docker-compose.yaml через vi нужно добавить вот эти строки, так это нужно будет для дальнейшей работы:

    node-exporter: 
    image: prom/node-exporter 
    volumes: 
      - /proc:/host/proc:ro 
      - /sys:/host/sys:ro 
      - /:/rootfs:ro 
    container_name: exporter 
    hostname: exporter 
    command: 
      - --path.procfs=/host/proc 
      - --path.sysfs=/host/sys 
      - --collector.filesystem.ignored-mount-points 
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/) 
    ports: 
      - 9100:9100 
    restart: unless-stopped 
    environment: 
      TZ: "Europe/Moscow" 
    networks: 
      - default
![изображение](https://github.com/user-attachments/assets/e1bba319-bd41-4c14-9b1c-8fe46bf6cc0d)

После всех действий и запуска докера, переходим к следующему пункту.

# 2. Установка Prometheus в Grafana

Prometheus и Grafana часто используются вместе для мониторинга и визуализации данных. Prometheus собирает метрики, а Grafana визуализирует их в виде удобных дашбордов. Ниже приведены основные шаги для интеграции Prometheus в Grafana:

1. Установите и настройте Prometheus
    Перед тем как интегрировать Prometheus в Grafana, необходимо убедиться, что Prometheus установлен и настроен для сбора данных. Если Prometheus уже настроен и работает, убедитесь, что он собирает метрики с нужных источников (экспортеров).
![изображение](https://github.com/user-attachments/assets/98614218-5a2a-404e-8df6-dbb8471d9365)

2. Запустите Grafana и откройте интерфейс
    После установки и запуска Grafana перейдите в веб-интерфейс по умолчанию (обычно это порт 3000 на локальном хосте, например: http://localhost:3000). Войдите в систему с помощью стандартных учетных данных (логин и пароль по умолчанию — admin/admin).

3. Добавьте источник данных Prometheus
    Чтобы Grafana могла отображать данные из Prometheus:
        В интерфейсе Grafana перейдите в раздел "Configuration" (Конфигурация) -> "Data Sources" (Источники данных).
        Нажмите кнопку "Add Data Source" (Добавить источник данных).
        В списке источников данных выберите Prometheus.
        В поле URL укажите адрес, по которому работает Prometheus (например: http://localhost:9090).
        Нажмите кнопку "Save & Test" (Сохранить и протестировать), чтобы проверить подключение.
![изображение](https://github.com/user-attachments/assets/a9ef47c4-f88e-45e7-a677-330645f6c5b8)

4. Создание дашбордов
    После успешного подключения Prometheus к Grafana вы можете создавать дашборды для визуализации метрик:
        Перейдите в раздел "Dashboards" (Дашборды) -> "New Dashboard" (Новый дашборд).
        Добавьте панель (panel) с графиками, выбрав Prometheus в качестве источника данных.
        В поле для запроса Prometheus введите нужные метрики (например, up, node_cpu_seconds_total, и другие), чтобы вывести соответствующую информацию.
![изображение](https://github.com/user-attachments/assets/135fc8ef-ef4c-4e05-b0ce-42a275fc80b5)

5. Импорт готовых дашбордов
    Чтобы ускорить процесс, можно импортировать уже готовые дашборды из сообщества Grafana:
        Перейдите в раздел "Dashboards" -> "Import" (Импорт).
        Введите ID дашборда из официального каталога (например, ID 1860 для популярных метрик по Prometheus).
        После импорта настройте панель для работы с вашим источником данных Prometheus
![изображение](https://github.com/user-attachments/assets/117be96f-a851-4e33-97b3-e13aafc812a9)

# 3. Настраивание VictoriaMetrics 

Первым делом нам нужно вернуться в папку grafana_stack_for_docker

    cd grafana_stack_for_docker

Открываем docker-compose, так как нам нужно будет ввести новые данные.

    sudo vi docker-compose.yaml

Вводим данные строки в наш файл, самое главное их нужно вставить после prometheus!

      vmagent:
    container_name: vmagent
    image: victoriametrics/vmagent:v1.105.0
    depends_on:
      - "victoriametrics"
    ports:
      - 8429:8429
    volumes:
      - vmagentdata:/vmagentdata
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - "--promscrape.config=/etc/prometheus/prometheus.yml"
      - "--remoteWrite.url=http://victoriametrics:8428/api/v1/write"
    restart: always
    # VictoriaMetrics instance, a single process responsible for
      # storing metrics and serve read requests.
      victoriametrics:
    container_name: victoriametrics
    image: victoriametrics/victoria-metrics:v1.105.0
    ports:
      - 8428:8428
      - 8089:8089
      - 8089:8089/udp
      - 2003:2003
      - 2003:2003/udp
      - 4242:4242
    volumes:
      - vmdata:/storage
    command:
      - "--storageDataPath=/storage"
      - "--graphiteListenAddr=:2003"
      - "--opentsdbListenAddr=:4242"
      - "--httpListenAddr=:8428"
      - "--influxListenAddr=:8089"
      - "--vmalert.proxyURL=http://vmalert:8880"
    restart: always

![изображение](https://github.com/user-attachments/assets/b5f66a5c-e73e-4d90-86cb-2c68f5bac401)

В кладке connection нужно будет найти Prometheus в "Data sources", кликнув по нему, вам нужно будет поменять "http://prometheus:9090" на "http://victoriametrics:8428" и переназвать имя на "Vika"

 ![изображение](https://github.com/user-attachments/assets/d8e7204c-a726-4d0f-8b37-48a44376cd2b)


 ![изображение](https://github.com/user-attachments/assets/0faccc35-0de8-416d-aab4-1db4fa2259a5)

После этого нам нужно вернуться в терминал и написать ещё пару команд.

     echo -e "# TYPE OILCOINT_metric1 gauge\nOILCOINT_metric1 0" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus

      curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric1'


