# Визуализация данных с помощью Loki
## Общая информация
В этом проекте содержатся файлы, необходимые для развертывания docker и следующих контейнеров:
* Jenkins
* Opentelemetry Collector (OTelCol)
* Loki Grafana
* Grafana
* Jaeger
## Принцип работы
OTelCol принимает данные из двух источников: 
* Jenkins pipeline по протоколу GRPC 
* файлы в локальной директории

OTelCol затем отсылает эти данные для визуализации:
* информации типа traces в Jaeger
* информации типа logs в Loki

Подробнее о том, что входит в эти типы: https://opentelemetry.io/docs/concepts/signals/

## Источники
Код для примера взят из следующих примеров:
* https://github.com/ichernysheff/jenkins-basic
* https://grafana.com/docs/opentelemetry/collector/send-logs-to-loki/

## Запуск проекта
1. В главной директории проекта запустить следующие команды из https://github.com/ichernysheff/jenkins-basic/blob/master/README.md
```
docker build -t jenkins-basic .
docker run --name jenkins-basic --restart=always --detach --publish 8080:8080 --volume jenkins-basic:/home/jenkins/data/ jenkins-basic
```
После этого по адресу http://localhost:8080 можно будет открыть Jenkins UI

2. Зайти в директорию `send_logs_to_grafana_loki_exmp` и выполнить команду
```
docker-compose up -d
```
Создадутся контейнеры с Jaeger, Loki и Grafana. Порты можно узнать командой:
```
docker container ls
```

3. Контейнер OTelCol в данном примере запускается из image. Для начала распакуем `otelcol-contrib_0.63.1_linux_amd64.tar.gz`:
```
gunzip -c otelcol-contrib_0.63.1_linux_amd64.tar.gz | tar xopf -
```
Затем запустим с использованием файла конфигурации:
```
./otelcol-contrib --config ./otel-config.yaml
```

4. Файл с тестами `test_capitalize.py` необходимо запустить в Jenkins. Одним из способов сделать это является перенос файла в рабочую директорию пайплайна.
Зайти в рабочее пространство Jenkins можно командой:
```
docker exec -it <container_id> bash
```
Узнать путь до рабочей директории можно в Jenkins UI. 

5. Для запуска тестов создадим новый Pipeline в Jenkins и запустим следующий код:
```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                sh"""#!/bin/bash
                                        shopt -s expand_aliases
                                        source /etc/profile
                                        source /etc/bashrc
                                        source ~/.bashrc
                                        source /fs/GE/default/common/jenkins_settings.sh
                                        
                                        source pytest/bin/activate
                                        dpkg -l | grep pytest-opentelemetry
                                        export OTEL_EXPORTER_OTLP_ENDPOINT=http://172.18.0.1:4317
                                        pytest --export-traces
                """
            }
        }
    }
}

```

