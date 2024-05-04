# ImagePlace - место для размещение изображений.

## Описание проекта
Пользователи могут регистрироваться, загружать изображения с кратким описанием, смотреть изображения других пользователей, добавлять категории. 

## Технологии

 - Python
 - Django
 - DRF
 - Nginx
 - gunicorn
 - Docker
 - GitHub Actions
 - PostgreSQL


## Инструкция по запуску проекта

### Установка 

1. Клонируйте репозиторий на свой компьютер:

    ```bash
    git clone git@github.com:faqfriend/kittygram_final.git
    ```
    ```bash
    cd kittygram
    ```
2. Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.example.


### Создание Docker-образов

1.  Замените username на ваш логин на DockerHub:

    ```bash
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```bash
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию kittygram через терминал

    ```bash
    mkdir kittygram
    ```

3. Установка docker compose на сервер:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. В директорию kittygram/ скопируйте файлы docker-compose.production.yml и .env:

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    * ath_to_SSH — путь к файлу с SSH-ключом;
    * SSH_name — имя файла с SSH-ключом;
    * username — ваше имя пользователя на сервере;
    * server_ip — IP вашего сервера.
    ```

5. Запустите docker compose в режиме демона:

    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. На сервере в редакторе nano откройте конфиг Nginx:

    ```bash
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки location в секции server:

    ```bash
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте работоспособность конфига Nginx:

    ```bash
    sudo nginx -t
    ```
    Если ответ в терминале такой, значит, ошибок нет:
    ```bash
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

10. Перезапускаем Nginx
    ```bash
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Используйте уже готовый файл workflow. Он находится здесь:

    ```bash
    kittygram/.github/workflows/main.yml
    ```

2. Создайте секреты в GitHub Actions:

    ```bash
    DOCKER_USERNAME                # имя пользователя в DockerHub
    DOCKER_PASSWORD                # пароль пользователя в DockerHub
    HOST                           # ip_address сервера
    USER                           # имя пользователя
    SSH_KEY                        # приватный ssh-ключ (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE                 # кодовая фраза (пароль) для ssh-ключа

    TELEGRAM_TO                    # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN                 # токен бота (получить токен можно у @BotFather, /token, имя бота)
    ```


## Примеры запросов API:
* Создание нового пользователя:
  
  - api/users/
```
    {
        "email": "string",
        "username": "string",
        "password": "string"
    }

``` 
* Получение токена для аутентификации (Token): 

  - api/token/
```
    {
        "username": "string",
        "password": "string"
    }

``` 
* Получить список всех котов (GET): 

  - api/сats/

```
{
    "count": 10,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 24,
            "name": "Мурка",
            "color": "black",
            "birth_year": 1999,
            "achievements": [
                {
                    "id": 3,
                    "achievement_name": "Спит весь день"
                },
                {
                    "id": 1,
                    "achievement_name": "Считай голубей"
                }
            ],
            "owner": "Иван",
            "age": 24,
            "image": "http://127.0.0.1:8080/media/cats/images/temp.png",
            "image_url": "/media/cats/images/temp.png"
        },
  ....

```
* Разместить фото кота (POST): 

  - api/cats/
  - В поле image передавать строку с картинкой в формате base64 

```
        {
            "name": "Василий",
            "color": "#DCDCDC",
            "birth_year": 2020,
            "image": base64
            "achievements": [
                {"achievement_name": "разбил вазу"}
            ]
        }   

```
