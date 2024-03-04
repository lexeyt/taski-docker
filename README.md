# taski-docker
### Описание
Сервис для планирования задач. Позволяет добавлять, изменять статус и удалять задачи. В рамках данного проекта полностью автоматизирован CI/CD на прод сервер с применением GitHub Actions.

### Технологии
[![Python](https://img.shields.io/badge/-Python-464646?style=flat&logo=Python&logoColor=56C0C0&color=cd5c5c)](https://www.python.org/)
[![Django](https://img.shields.io/badge/-Django-464646?style=flat&logo=Django&logoColor=56C0C0&color=0095b6)](https://www.djangoproject.com/)
[![Django REST Framework](https://img.shields.io/badge/-Django%20REST%20Framework-464646?style=flat&logo=Django%20REST%20Framework&logoColor=56C0C0&color=cd5c5c)](https://www.django-rest-framework.org/)
[![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-464646?style=flat&logo=PostgreSQL&logoColor=56C0C0&color=0095b6)](https://www.postgresql.org/)
[![Nginx](https://img.shields.io/badge/-NGINX-464646?style=flat&logo=NGINX&logoColor=56C0C0&color=cd5c5c)](https://nginx.org/ru/)
[![gunicorn](https://img.shields.io/badge/-gunicorn-464646?style=flat&logo=gunicorn&logoColor=56C0C0&color=0095b6)](https://gunicorn.org/)
[![Docker](https://img.shields.io/badge/-Docker-464646?style=flat&logo=Docker&logoColor=56C0C0&color=cd5c5c)](https://www.docker.com/)
[![Docker-compose](https://img.shields.io/badge/-Docker%20compose-464646?style=flat&logo=Docker&logoColor=56C0C0&color=0095b6)](https://www.docker.com/)
[![Docker Hub](https://img.shields.io/badge/-Docker%20Hub-464646?style=flat&logo=Docker&logoColor=56C0C0&color=cd5c5c)](https://www.docker.com/products/docker-hub)
[![GitHub%20Actions](https://img.shields.io/badge/-GitHub%20Actions-464646?style=flat&logo=GitHub%20actions&logoColor=56C0C0&color=0095b6)](https://github.com/features/actions)

### Инструкция по CI/CD
Инструкции автоматического тестирования и развертки на сервере приведены в файле:
```
.github/workflows/main.yml
```
Файлы инструкций docker-compose располагаются в соответствующих директориях. 

### Инструкция ручному запуску сервиса
Подключитесь к удалённому серверу и клонируйте проект:
```
git clone git@github.com:lexeyt/taski-docker.git
```
Переходим в директорию backend-приложения проекта
```
cd taski-docker/backend/
```
Создаём виртуальное окружение
```
python -m venv venv
```
Активируем виртуальное окружение
```
source venv/bin/activate
```
Обновляем pip в виртуальном окружении
```
pip install --upgrade pip
```
Устанавливаем зависимости
```
pip install -r requirements.txt
```
Из директории, в которой находится файл manage.py применяем миграции
```
python manage.py migrate
```
Создаём суперпользователя
```
python manage.py createsuperuser
```
Собираем статику бэкенда
```
python manage.py collectstatic
```
Из корня проекта скопируем статику бэкенда в системную директорию
```
sudo cp -r /home/yc-user/taski-docker/backend/static_backend/ /var/www/taski/
```
Запускаем веб-сервер разработки Django
```
python manage.py runserver
```
В файле settings.py xxx.xxx.xxx.xxx укажите IP вашего сервера
```
ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost']
```
В другом окне терминала установите зависимости для фронтенд-приложения. Перейдите в директорию taski-docker/frontend/ и выполните команду
```
npm i
```
Запустите приложение командой
```
npm run start
```
Проверте тестовый запуск в браузере по адресу
http://внешний_ip_адрес_сервера:3000

### Установка и запуск Gunicorn
На удалённом сервере при активированном виртуальном окружении проекта
```
pip install gunicorn==20.1.0
```
Из директории с файлом manage.py
```
gunicorn --bind 0.0.0.0:8000 backend.wsgi
```
Проверим на админке - должна работать без статики
```
http://ваш_публичный_IP:8000/admin/
```
Остановим и запустим для непрерывной работы.

В директории /etc/systemd/system/ создайте файл gunicorn.service и откройте его в Nano
```
sudo nano /etc/systemd/system/gunicorn.service
```
Подставьте в код из листинга свои данные, добавьте этот код без комментариев в файл gunicorn.service и сохраните изменения
```
[Unit]
# Это текстовое описание юнита, пояснение для разработчика.
Description=gunicorn daemon 

# Условие: при старте операционной системы запускать процесс только после того, 
# как операционная система загрузится и настроит подключение к сети.
# Ссылка на документацию с возможными вариантами значений 
# https://systemd.io/NETWORK_ONLINE/
After=network.target 

[Service]
# От чьего имени будет происходить запуск:
# укажите имя, под которым вы подключались к серверу.
User=yc-user 

# Путь к директории проекта:
# /home/<имя-пользователя-в-системе>/
# <директория-с-проектом>/<директория-с-файлом-manage.py>/.
# Например:
WorkingDirectory=/home/yc-user/taski-docker/backend/

# Команду, которую вы запускали руками, теперь будет запускать systemd:
# /home/<имя-пользователя-в-системе>/
# <директория-с-проектом>/<путь-до-gunicorn-в-виртуальном-окружении> --bind 0.0.0.0:8000 backend.wsgi
ExecStart=/home/yc-user/taski-docker/backend/venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi

[Install]
# В этом параметре указывается вариант запуска процесса.
# Значение <multi-user.target> указывают, чтобы systemd запустил процесс,
# доступный всем пользователям и без графического интерфейса.
WantedBy=multi-user.target
```
Чтобы точно узнать путь до Gunicorn, активируйте виртуальное окружение и воспользуйтесь командой
```
which gunicorn
```
Заново запустите процесс gunicorn.service
```
sudo systemctl start gunicorn 
```
Добавьте процесс Gunicorn в список автозапуска операционной системы на удалённом сервере
```
sudo systemctl enable gunicorn 
```
### Установка Nginx
Находясь на удалённом сервере, из любой директории
```
sudo apt install nginx -y
sudo systemctl start nginx
```
Укажите файрволу, какие порты должны остаться открытыми
```
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
```
Включите файрвол
```
sudo ufw enable
```
Проверьте внесённые изменения
```
sudo ufw status
```
Запустите сборку фронтенд-приложения из директории tasking-django-react/frontend/
```
npm run build
```
Скопируйте в системную директорию Nginx (которую он использует по умолчанию для доступа к статическим файлам — /var/www/) содержимое папки .../frontend/build/
```
sudo cp -r /home/yc-user/tasking-django-react/frontend/build/. /var/www/taski/
```
Через редактор Nano откройте файл конфигурации веб-сервера
```
sudo nano /etc/nginx/sites-enabled/default
```
Удалите все настройки из файла, запишите и сохраните новые
```
server {

    listen 80;
    server_name публичный_ip_вашего_удалённого_сервера;
    
    location /api/ {
        # Эта команда определяет, куда нужно перенаправить запрос.
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location / {
        root   /var/www/taski;
        index  index.html index.htm;
        try_files $uri /index.html;
    }

}
```
Проверьте файл конфигурации на ошибки
```
sudo nginx -t
```
Перезагрузите конфигурацию Nginx
```
sudo systemctl reload nginx
```
В адресную строку браузера введите внешний IP вашего удалённого сервера без указания порта.

Команда для просмотра лога последних запросов
```
sudo tail /var/log/nginx/access.log
```
### Автор
[Тачеев Алексей](https://github.com/lexeyt/)
