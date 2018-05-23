# spinor72_microservices
[![Build Status](https://travis-ci.org/Otus-DevOps-2018-02/spinor72_microservices.svg?branch=master)](https://travis-ci.org/Otus-DevOps-2018-02/spinor72_microservices)

Репозиторий для ДЗ по теме "microservices"

## ДЗ №13 Технология контейнеризации. Введение в Docker.

 - [x] Основное ДЗ
 - [x] Задание со *
 
### В процессе сделано:
Установлены пакеты для работы с контейнерами:
```
docker version 18.03.0-ce
docker-compose version 1.16.1,
docker-machine version 0.14.0
```
Изучены основные команды для `docker`:   
 `inof`, `run [-i|-d][-t]`, `ps`, `images`, `create`, `start`, `attach`, `exec`, `commit`, `inspect`, `kill`, `system df`, `rm`, `rmi` 

Записан вывод команды `docker images` в файл `docker-monolith/docker-1.log` , в который для задания * добавлено описание отличий вывода команды  `docker inspect` для образа и контейнера

### Как запустить проект:
не применимо

### Как проверить работоспособность:
не применимо

## ДЗ №14 Docker-контейнеры.

 - [x] Основное ДЗ
 - [x] Задание со *
 
### В процессе сделано:

 - Создан новый проект в GCP с названием docker;
 - С помощью пакета `docker-machine` в GCE развернут хост для работы с docker-контейнерами
 - Проверена работа изоляции ресурсов, например, опция запуска `--pid host` отключает изоляцию на уровне pid, таким образом контейнер видит все процессы системы, в отличие от запуска по-умолчанию, без такой опции
 - Создан `Dockerfile` для генерации образа с тестовым приложением.
 - Образ контейнера с тестовым приложение мсгенерирован и размещен в реестре Docker Hub под именем `spinor72/otus-reddit`
 - Проверена процедура развертывания созданного образа на docker-host из размещенного в реестре образа

Для задания со *
 - Созданы шаблоны packer для генерации образа с установленным Docker
 - Настроено создание заданного количества хостов в GCE с помощью  терраформа 
 - Созданы плейбуки ansible для установки Docker и развертывания контейнера  с тестовым приложением (используется динамическое инвентори из terraform state в бакете)

### Как запустить проект:

 - Создать контейнер

    В папке docker-monolith создать образ  командой вида, где `docker_hub_login` - ваш логин в Docker Hub
    ```
    docker build -t docker_hub_login/otus-reddit:1.0 .
    ```
    и, авторизовавшись командой `docker login` загрузить его в свой реестр на Docker Hub,
    ```
    docker push docker_hub_login/otus-reddit:1.0
    ```
    Запустить контейнер командой `docker run --name reddit -d -p 9292:9292 docker_hub_login/otus-reddit:1.0`


 - Развернуть инфраструктуру
    - перейти в `docker-monolith/infra/terraform`
    - Создать бакет для хранения стейта терраформ командой `terraform apply` 
    - Перейти в `docker-monolith/infra/terraform/prod` На базе example-файла с перемеными создать свой файл, задать количество создаваемых инстансов в переменой `docker_count` создать инстансы командой `terraform apply` 
    - перейти в `docker-monolith/infra/ansible` и выполнить установку ролей: `ansible-galaxy install -r environments/prod/requirements.yml`
    - выполнить развертывание приложения командой 
    `ansible-playbook playbooks/site.yml` , при необходимости подставив название своего контейнера в переменную `docker_container_image`

 - Создать образ с установленным Docker  
    перейти в `docker-monolith/infra` и выполнить `packer build -var-file=packer/variables.json packer/docker.json` предварительно создав файл `docker-monolith/infra/packer/variables.json` с нужными значениями переменных по образцу *.example


### Как проверить работоспособность:

Перейти по ссылке `http://<host_ip>:9292`  , где host_ip это localhost, при локальном запуске контейнера, ip-адрес хоста созданного docker-machine или ip-адрес, любого из созданных терраформом инстансов 

_После проверки удалить ненужные инстансы командой `terraform destroy` из папки `docker-monolith/infra/terraform/prod`_


## ДЗ №15 Docker-образа. Микросервисы.

 - [x] Основное ДЗ
 - [x] Задание со *
 - [x] Задание с ** ?
 
### В процессе сделано:

 - в репозиторий добавлены исходные файлы для развертывания микросервисов
 - созданы `Dockerfile` для запуска тестового приложения в виде набора из 3-х микросервисов `post-py`, `comment`, `ui`
 - для базы данных используется докер с MongoDB (`mongo:latest`), для постоянного хранения данных к которому добавлен volume (`docker volume create reddit_db`)
 - собраны образы для микросервисов, проверена работа приложения после запуска 4-х контейнеров.
 - проверена передача переменных среды при запуске контейнера, для переопределения прописанных в `Dockerfile` значений
 - рассмотрены методы оптимизации размера образов, на прмиере образа `ui` 
      - переход на образ `ubuntu`, размер уменьшается до около 400Мб
      - \* при переходе на  `alpine` образ выходит в размере около 200Мб
      - ** далее образ из `alpine` оптимизирован путем удалениея build-пакетов  до 35.7Мб , можно попробовтаь еще уменьшить размер, использовав multi stage сборку на базе scratch-образа, скопировав все, кроме, например таких системных утилит как apk и т.п, которые не нужны для работы ruby. 
      фрагмент таблицы с размерами образов:
      ```
      REPOSITORY          TAG         SIZE
      spinor72/comment    1.0         739MB
      spinor72/post       1.0         102MB
      spinor72/ui         1.0         746MB  # ruby 
      spinor72/ui         2.0         399MB  # ubuntu
      spinor72/ui         2.1         35.7MB # alpine 
      ```
 - Для проверки докер-файлов использован линтер (некоторые проверки отключены в инлайн комментарии докер-файла)
     ```
     docker run --rm -i hadolint/hadolint < ui/Dockerfile
     docker run --rm -i hadolint/hadolint < post-py/Dockerfile
     docker run --rm -i hadolint/hadolint < comment/Dockerfile
     ```

### Как запустить проект:

 - Перейти в папку `/src`
 - Подготовить контейнеы
     ```
    docker pull mongo:latest
    docker build -t <your-dockerhub-login>/post:1.0 ./post-py
    docker build -t <your-dockerhub-login>/comment:1.0 ./comment
    docker build -t <your-dockerhub-login>/ui:2.1 ./ui
    ```

 - Создать том для базы данных и сеть
    ```
    docker volume create reddit_db
    docker network create reddit
    ```

 - Запустить контейнеры (приведен пример с переопределением сетевых алиасов, как для задания * )
    ```
    docker run -d --network=reddit --network-alias=post_db_new --network-alias=comment_db_new mongo:latest
    docker run -d --network=reddit --network-alias=post_new --env POST_DATABASE_HOST=post_db_new <your-dockerhub-login>/post:1.0
    docker run -d --network=reddit --network-alias=comment_new --env COMMENT_DATABASE_HOST=comment_db_new <your-dockerhub-login>/comment:1.0
    docker run -d --network=reddit -p 9292:9292 --env POST_SERVICE_HOST=post_new --env COMMENT_SERVICE_HOST=comment_new <your-dockerhub-login>/ui:2.1
    ```

### Как проверить работоспособность:

Перейти по ссылке `http://<host_ip>:9292`  , где host_ip это localhost, при локальном запуске контейнеров или ip-адрес хоста созданного docker-machine
Сделать один или несколько постов. Убедится, что при убивании  командой `docker kill $(docker ps -q)` и повторном запуске контейнеров  данные не пропадают.


## ДЗ №16 Docker: сети, docker-compose.

 - [x] Основное ДЗ
 - [x] Задание со *
 
### В процессе сделано:

Работа с сетями в Docker
 - изучены сети типов `none`, `host`, `bridge`. 
 - Для задания со * использованы команды
    -  `ip netns`  и  `ip netns exec <namespace> <command>`  для просмотра неймспейсов и запуска команд в нужном неймспейсе
    -  `brctl show` для отображения veth-интерфейсов, подключенных к bridge-интерфейсом докера 
    -  `iptables -nL -t -v` для просмотра правил iptables, обеспечивабщих работу сетей Docker  
    -  `ps ax | grep docker-proxy` для отображения прокси-процесса, используемого дял направления траффика в контейнер
 - Изучен вопрос подключения контейнера к нескольким бридж-сетям и присвоения сетевых алиасов для сетевого взаимодействия конетйнеров 

Работа с docker-compose
 - создан файл `docker-compose.yml` для запуска тестового приложения из микросервисов
 - файл параметризован  с помощью переменных окружения, значения которых загружаются из `.env` файла, образец заполения см. `/src/.env.example`
 - задано базовое имя проекта, с помощью переменной COMPOSE_PROJECT_NAME (или использовать ключ -p при запуске docker-compose)
 - файл docker-compose.yml доработан для использования нескольких сетей и сетевых алиасов
 - для задания * сделан файл `docker-compose.override.yml` в котором изменяются параметры docker-compose, для примера в репозиторий помещен файл   `/src/docker-compose.override.yml.example`, с помощью которого: 
    - опцией bind подключется папка с исходным кодом для микросервисов
    - изменяются параметры запуска сервера puma


### Как запустить проект:

 - Для использования удленного хоста, запустить его с помощью docker-machine и настроить докер на работу с ним, командой
`eval $(docker-machine env docker-host)`

 - Перейти в папку `/src`
 - Скопировать файл `.env.example` в `.env` и задать нужные значения переменных
 - Запустить командой `docker-compose up -d`

Для переопределения параметров контейнеров:

 - скопировать файл `docker-compose.override.yml.example` в `docker-compose.override.yml` , 
 - перенести на `docker-host` код приложений , например, командой `docker-machine scp -r . docker-host:src` или склонировав репозитоирий.
 - задать переменную `SRC_PATH` в файле `.env` (при локлаьном использовании Docker из папки `/src` можно не задавать) 
 - снова запустить `docker-compose up -d`. 
 - В дальнейшем при обновлениия кода приложения сделать перезапуск контейнеров `docker-compose restart`


### Как проверить работоспособность:

Перейти по ссылке `http://<host_ip>:9292`  , где host_ip это localhost, при локальном запуске контейнеров или ip-адрес хоста созданного docker-machine

Должен открыться интерфейс тестового приложения в котором можно делать посты и комментарии


## ДЗ №17 Устройство Gitlab CI. Построение процесса непрерывной интеграции.

 - [x] Основное ДЗ
 - [x] Задание со *
 
### В процессе сделано:

 - Для работы с Gitlab CE поднят инстанс gitlab-host с помощью  docker-machine
    ```
    docker-machine create --driver google --google-disk-size 100 \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west4-b \
    gitlab-host
    ```
 - На созданном хосте, используя docker-compose развернут контейнер с сервисом Gitlab в  версии Omnnibus Gitlab
 - Выполнена первичная настройка gitlab
 - Созданы тестовые группа `homework` и проект `example`
 - В  "microservices"-репозиторий добавлен файл описания CI/CD Pipeline `.gitlab-ci.yml` 
 - Добавлен раннер в виде докер-сервиса и зарегистрирован для использования в CI/CD pipline тестового проекта на Gitlab
 - В приложение reddit добавлены тесты и необходимые библиотеки для тестирования
 - "microservices"-репозиторий синхронизирован в тестовый проект на gitlab
 - проверено что изменение кода в папке приложения reddit привозит к запуску pipeline и оно корректно срабатывает 

Для задания со * 

 - Для автоматического масштабирования раннеров можно использовать возможность autoscaling раннеров https://docs.gitlab.com/runner/configuration/autoscale.html 
 - Можно генерировать инфраструктуру раннеров с помощью terrafrom, образ с "запеченным" раннером и докером делатся пакером, который использует `ansible` для провижининга. Регистрация раннеров происходит при провижининге терраформом в `remote-exec`. Пример приведен в папке /gitlab-runner . Но если предполагается изменение инфрастурктуры то, видимо,нужно еще синхронизировать наличие раннеров с gitlab, например удалять отсутствующие. Для этого можно использовать API Gitlab и соответственно написать скрипт.
 
 - Настроена интеграция Pipeline с тестовым  Slack-чатом 


### Как запустить проект:

 - Установить или использовать готовую утсановку Gitlab пусть это бует адрес  `http://<gitlab-ip>`
 - Создать группу и проект /homework/example
 - Добавить к тееущему "microservices"-репозиторию удаленный репозиторий на gitlab
    `git remote add gitlab http://<gitlab-ip>/homework/example.git `
    `git push gitlab gitlab-ci-1`
 -  создать и зарегистрировать раннер , например, с помощью пакер и терраформ из папки gitlab-runner 
     - Задать свои значения переменных в `/gitlab-runner/packer/variables.json`, для образца использовать example-файл
     - Перейти в папку `/gitlab-runner` и создать образ пакером `packer build -var-file=packer/variables.json packer/gitlab-runner.json`
     - В папке `/gitlab-runner/terraform`  создать файл `terraform.tfvars`со своими значеними переменных (для образца использовать example-файл и кроме параметров GCP, необходимо указать адрес http://<gitlab-ip> и токен для регистрации gitlab  а также количество раннеров в переменной `count`) и запустить создание инфраструктуры раннеров терраформом (`terraform init` , `terraform apply`)
    - в settings/ci_cd проекта должны появитьcя зарегистрированные раннеры
 

### Как проверить работоспособность:

 - Должен автоматически стартовать pipeline при коммите  в репозиторий (после отправки изменений в gitlab). Также можно запустить pipeline вручную
 - В slack-канал https://devops-team-otus.slack.com/messages/C9NT0JMSA должны приходить уведомления от gitlab


## ДЗ №18 Устройство Gitlab CI. Непрерывная поставка.

 - [x] Основное ДЗ
 - [x] Задание со *
 
### В процессе сделано:
 
 - Созданы тестовый проект `example2` и в "microservices"-репозиторий добавлен  `remote` на него 
 - зарегистрирован раннер для использования в этом проекте
 - Добавлено Dev окружение в `.gitlab-ci.yml`
 - Добавлены  два новых этапа: `stage` и `production` , к которым добавлены ограничения  для срабатывания
 - Определены динамические окружения для каждой ветки в репозитории, кроме ветки master
 - Проверена работа pipeline в разных ситуациях 

Для задания со *  

 - Включил registry в omnibus установку Gitlab для хранения образов конетйнеров
 - В шаг build добавил сборку контейнера с приложением reddit и сохранение его в gitlab registry
 - Добавил создание сервера для каждого пуша новой ветки и разворачивания на сервере контейнера с приложением и удаление сервера кнопкой

В этом задании используется отдельный shell-раннер который с помощью docker-machine создает и удаляет инстансы для развертывания приложения. Описание инфраструктуры раннеров находится в папке `/gitlab-runner` 


### Как запустить проект:

 - Установить или использовать готовую утсановку Gitlab пусть это бует адрес  `http://<gitlab-ip>`
 - Создать группу и проект, например `/homework/example2`
 - Добавить к текущему "microservices"-репозиторию удаленный репозиторий на gitlab
    `git remote add gitlab http://<gitlab-ip>/homework/example2.git `
    `git push gitlab2 gitlab-ci-2`
 -  создать и зарегистрировать раннеры с помощью пакер и терраформ из папки gitlab-runner 
     - Задать свои значения переменных в `/gitlab-runner/packer/variables.json`, для образца использовать example-файл
     - Перейти в папку `/gitlab-runner` и создать образ пакером `packer build -var-file=packer/variables.json packer/gitlab-runner-machine.json`
     - В папке `/gitlab-runner/terraform`  создать файл `terraform.tfvars`со своими значеними переменных (для образца использовать example-файл и кроме параметров GCP, необходимо указать адрес http://<gitlab-ip> и токен для регистрации gitlab,  а также количество раннеров в переменной `count` и данные gitlab registry) и запустить создание инфраструктуры раннеров терраформом (`terraform init` , `terraform apply`)
     - в settings/ci_cd проекта должны появитьcя зарегистрированные раннеры
     - Добавить переменные в Secret variables  settings/ci_cd нужные для работы docker-machine и билда контейнеров 
        ```
        GITLAB_REGISTRY: 'gitlab-host:4567'
        GCP_PROJECT: 'project-name'
        GCP_ZONE: 'europe-west4-b'
        ```


### Как проверить работоспособность:

 - Должен автоматически стартовать pipeline при коммите  в репозиторий (после отправки изменений в gitlab2).
 - Для тегированных версией коммитов доступны стадии stage и production
 - Должны автоматически появляться новые серверы, на которых развернут код приложения из запушенной ветки
 - Серверы удаляются кнопкой
