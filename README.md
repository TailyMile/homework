<h1> Часть 1: Docker-Compose </h1>

Реализовано web-приложение HelloWorld на языке Python c использванием фреймворка Flask.
Flask выбран из-за удобства использования модуля prometheus_flask_exporter для наглядного мониторинга вэб-приложения на Python.

В том числе реализована связка Prometheus+Grafana с дэшбордами мониоринга хостового узла, вэб-приложения, и основных метрик Prometheus и Grafana.

Сборка непосредственно приложения для запуска его в виде Docker-контейнера описана в Docker-файле в каталоге ./hw
Используется деректива HEALTHCHECK для проверки состояния работы приложения через регулярный запрос вэб-страницы с помощью утилиты curl

Запуск полностью готового приложения с мониторингом осуществляется с помощью команды <code>docker-compose build && docker-compose up</code> в корневом каталоге репозитория и описан в файле docker-compose.yaml
<br>Выполнение команды приводит к следующим автоматическим шагам:
1. Происходит сборка вэб-приложения с параметрами описаными выше
2. Происходит загрузка и запуск образов со следующими параметрами:
    - Для Prometheus монтируется файл конфигурации ./prometheus.yaml с указанием хостов для сбора метрик
    - Для Grafana монтируются каталоги с файлами шаблонов дэшбордов и источника данных (http://prometheus:5000)
3. Запускается образ Node Exporter для мониторинга состояния хостовой системы

Для доступа из браузера используются следующие ендпоинты:
- Web-приложение HelloWorld - http://localhost:5000
- Prometheus - http://localhost:9090 (логин/пароль не треубется)
- Grafana - http://localhost:3000 (RO - вход без пароля; RW - admin/admin)

<h1> Часть 2: k8s </h1>

Подготовлены деплоймент файлы, для автоматического развертывания приложения вместе с мониторингом.
Деплой осуществляется с помощью заранее подготовленных образов Docker загруженых на Docker Hub.
<br>Ресурсы:
- ./confs -- Файлы конфигурации для работы Prometheus и Grafana (конфигурации источников данных, дэшборды)
- ./Deployments -- инструкции запуска. На каждый ресурс создан свой деплоймент описывающий создание пода с healh и read-check, определением потребляемых ресурсов и полтики презапуска + создание сервиса для сетевого взаимодействия между сервисами.

<br>Запуск:
1. Запуск деплоя осуществляется командой <code>kubectl create -f ./Deployment/</code> из каталога k8s текущего репозитория.
    - Для корректной работы в minikube требуется запустить его с параметром <code>minikube --mount-string ./confs/:/confs</code>
    <br><i>Это необходимо так как файлы конфигурации монтируются внутри контейнеров через дерективу spec.volumes, где источником указан каталог /confs </i></br>
2. Для доступа к Grafana: <code> kubectl port-forward deployments/grafana 3000:3000 </code>. После чего мониторинг станет доступен по адресу: http://localhost:3000
Примечание: Похожим образом можно получить доступ ко всем приложениям использовав необходимое имя депйломента и соответвующий порт:
    - Grafana: deployments/grafana 3000:3000
    - Prometheus: deployments/prometheus 9090:9090
    - helloworld: deployments/helloworld 5000:5000