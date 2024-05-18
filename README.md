# taski-docker
### Описание проекта 
taski - сервис для любителей котиков.

Что умеет проект:

- Добавлять, просматривать, редактировать и удалять котиков.
- Добавлять новые и присваивать уже существующие достижения. 
- Просматривать чужих котов и их достижения.

## Установка 

1. Клонируйте репозиторий на свой компьютер:

    ```bash
    git clone https://github.com/AwakeGit/taski-docker.git
    ```
    ```bash
    cd taski_final
    ```
2. Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.example.


### Создание Docker-образов

1.  Замените username на ваш логин на DockerHub:

    ```bash
    cd backend
    docker build -t awakepn/taski_backend:latest .
    cd frontend
    docker build -t awakepn/taski_frontend:latest .
    cd gateway
    docker build -t awakepn/taski_gateway:latest . 
    ```

2. Загрузите образы на DockerHub:

    ```bash
    docker push awakepn/taski_backend:latest
    docker push awakepn/taski_frontend:latest
    docker push awakepn/taski_gateway:latest
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```bash
    ssh -i /пуь имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию taski через терминал

    ```bash
    mkdir taski
    ```

3. Установка docker compose на сервер:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. В директорию taski/ скопируйте файлы docker-compose.production.yml и .env:

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/taski/docker-compose.production.yml
    * ath_to_SSH — путь к файлу с SSH-ключом;
    * SSH_name — имя файла с SSH-ключом (без расширения);
    * username — ваше имя пользователя на сервере;
    * server_ip — IP вашего сервера.g

5. Добавьте в файл .env переменные и их значения:
   ```
   bash
   POSTGRES_USER=django_user
   POSTGRES_PASSWORD=mysecretpassword
   POSTGRES_DB=django

   # Добавляем переменные для Django-проекта:
   DB_HOST=db
   DB_PORT=5432

   # БД
   POSTGRES_USER — имя пользователя БД (необязательная переменная, значение по умолчанию — postgres);
   POSTGRES_PASSWORD — пароль пользователя БД (обязательная переменная для создания БД в контейнере);
   POSTGRES_DB — название базы данных (необязательная переменная, по умолчанию совпадает с POSTGRES_USER).
   ```



6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```bash
    sudo docker compose -f docker-compose.production.yml pull
    sudo docker compose -f docker-compose.production.yml down
    sudo docker compose -f docker-compose.production.yml up -d
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collect_static/. /static_backend/static/
    sudo docker compose -f docker-compose.production.yml up
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
