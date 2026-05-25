## Лабораторная работа по работе с docker
Работа посвящена изучению технологии работы с контейнерами

## Задачи

- [ ] 1. Ознакомиться со ссылками учебного материала
- [ ] 2. Выполнить инструкцию учебного материала
- [ ] 3. Составить отчет и отправить ссылку преподавателю 

## Задание лабораторной работы

```bash
$ export GITHUB_USERNAME=<имя_пользователя>
$ export GIST_TOKEN=<сохраненный_токен>
$ alias edit=<nano|vi|vim|subl>
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab06 projects/lab_docker
$ cd projects/lab_docker
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab_docker
```

```sh
# Debian
$ sudo apt-get update
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```sh
$ cat >> main.py <<EOF
print("Hello, Docker!")
EOF
```

```sh
$ cat >> requirements.txt <<EOF
flask
requests
EOF
```

```sh
$ cat >> Dockerfile <<EOF
FROM python:3.9-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential 

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "main.py"]
EOF
```

```sh
$ docker build -t lab-docker .
$ docker run --rm -it lab-docker
```

### Docker compose

```sh
$ cat >> docker-compose.yml <<EOF
version: '3.8'

services:
  app:
    build: . 
    container_name: lab_docker
    depends_on:
      db:
        condition: service_healthy
    environment:
      - DB_HOST=$DB_HOST
      - DB_USER=$DB_USER
      - DB_PASSWORD=$DB_PASSWORD
      - DB_NAME=$DB_NAME

  # Сервис базы данных MySQL
  db:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: $DB_ROOT_PASSWORD
      MYSQL_DATABASE: $DB_NAME
      MYSQL_USER: $DB_USER
      MYSQL_PASSWORD: $DB_PASSWORD
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
EOF
```

```sh
$ docker compose up --build
```

## Ссылки

### Docker compose

- [Install the Docker Compose plugin](https://docs.docker.com/compose/install/linux/)

### Dockerfile

- [Как запаковать простое приложение в Docker: на пальцах](https://habr.com/ru/companies/slurm/articles/930822/)

## Домашнее задание

В репозитории приведен код web-приложения, которое сохраняет в БД введенную информацию о задаче - ее имя.

## Часть I. Docker

1. Добавьте в код Dockerfile, который позволит запустить web-приложение с исходным кодом в каталоге app/ через docker.

```bash
cat > Dockerfile << 'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ .
CMD ["python", "app.py"]
```

Файл Dockerfile создан в корне проекта. Содержимое соответствует требованиям.

2. Выполните запуск контейнера с этим приложением.

```bash
docker build -t lab-docker .
docker run -d --name lab-docker-container -p 5000:5000 lab-docker
```

Вывод сборки:

```bash
Successfully built e3b0c44298fc
Successfully tagged lab-docker:latest
```

Контейнер запущен, веб-приложение доступно по адресу http://localhost:5000.
   
4. Скопируйте из консоли в каталог /home/ контейнера файл README.md.

```bash
docker cp README.md lab-docker-container:/home/README.md
```

Файл скопирован без ошибок.

6. Подключитесь к терминалу контейнера с приложением в интерактивном режиме. Проверьте, что скопированный файл находится в нужном каталоге.

```bash
docker exec -it lab-docker-container /bin/bash
```

Внутри контейнера выполнены команды:

```bash
ls -la /home/
cat /home/README.md
```

Вывод:
```bash
total 16
drwxr-xr-x 1 root root 4096 May 23 18:27 .
drwxr-xr-x 1 root root 4096 May 23 18:27 ..
-rw-rw-r-- 1 1000 1001 4442 May 23 18:22 README.md
## Лабораторная работа по работе с docker
...
```

Файл README.md присутствует в /home/, содержимое соответствует оригиналу.

8. Выйдите из интерактивного режима.

```bash
exit
```

Возврат в командную строку хост-системы.

10. Остановите контейнер с приложением.

```bash
docker stop lab-docker-container
```

Контейнер остановлен. Проверка: docker ps не показывает работающий контейнер.


## Часть II. Docker compose
1. Создайте файл docker-compose.yml таким образом, чтобы совместно с описанным в части 1 контейнером работала бы база данных mysql. Файл инициализации БД в каталоге db/init.sql. Также пропишите порт подключения к приложению. Например 5000.


Команда для .env (секреты, в репозиторий не включён):
```bash
cat > .env << 'EOF'
MYSQL_ROOT_PASSWORD=root_password_123
MYSQL_DATABASE=todo_db
MYSQL_USER=todo_user
MYSQL_PASSWORD=todo_password_456
DB_HOST=db
DB_USER=todo_user
DB_PASSWORD=todo_password_456
DB_NAME=todo_db
EOF
```

Команда для docker-compose.yml:

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  app:
    build: .
    container_name: lab_docker_app
    ports:
      - "5000:5000"
    depends_on:
      db:
        condition: service_healthy
    environment:
      - DB_HOST=${DB_HOST}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_NAME=${DB_NAME}
    restart: unless-stopped

  db:
    image: mysql:8.0
    container_name: lab_docker_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
EOF
```

Содержимое db/init.sql:

```bash
CREATE TABLE IF NOT EXISTS tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

2. Запустите связку web-приложение - БД.

```bash
docker compose up --build
```

Вывод (сокращённо):

```bash
[+] Building 1.0s (12/12) FINISHED
...
lab_docker_db  | 2026-05-23 18:44:23+00:00 [Note] [Entrypoint]: Database files initialized
lab_docker_db  | ... ready for connections. Version: '8.0.46'  port: 3306
lab_docker_app | * Running on http://0.0.0.0:5000
```

Оба контейнера запущены, сервис db прошёл healthcheck, приложение слушает порт 5000.

3. Проверьте подключение к приложению через браузер. Сделайте снимок экрана.

Действие:
Открыт браузер, в адресной строке введено http://localhost:5000.

Результат:
Появилась веб-страница с формой добавления задач и списком.

Скриншот:

<img width="740" height="559" alt="изображение" src="https://github.com/user-attachments/assets/ecdb2a73-9f15-4016-a4de-d80701ee7e88" />


4. Проверьте работу приложения через браузер.

Действие:
Добавлены задачи: «kupit moloko», «kupit hleb», «kupit ananas». После добавления они отобразились в списке. Страница обновлена — задачи сохранились.

Результат:
Приложение успешно читает и записывает данные в MySQL. Связка работает корректно.

Полный вывод команд:
[`dockervivod.txt`](https://github.com/dmitriysd9202/lab_docker/blob/main/dockervivod.txt).
