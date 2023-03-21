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
