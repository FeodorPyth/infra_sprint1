# Kittygram на удаленном сервере
## Описание проекта:
Kittygram — социальная сеть для обмена фотографиями любимых питомцев. 
Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

## СТЕК технологий:
* Python == 3.10.12
* Django == 3.2.3
* Django Rest Framework == 3.12.4
* Gunicorn == 20.1.0
* Nginx

## Особенности проекта:
- [x] Получено доменное имя через онлайн-сервис No-IP
- [x] На удаленный сервер был установлен Git  
- [x] Был связан аккаунт на GitHub с удаленным сервером
- [x] Настроен WSGI-сервер Gunicorn для работы с бэкенд-приложением
- [x] Настроен веб-сервер Nginx для перенаправления запросов и работы со статикой проекта
- [x] Подключено шифрование запросов по протоколу HTTPS
- [x] Пройдены автотесты

## Как запустить проект локально:
Клонировать репозиторий и перейти в него в командной строке:

```sh
git clone git@github.com:FeodorPyth/infra_sprint1.git
```

```sh
cd infra_sprint1
```

Cоздать и активировать виртуальное окружение:

```sh
python3 -m venv venv
```

```sh
source venv/bin/activate
```

Установить зависимости из файла requirements.txt:

```sh
python -m pip install --upgrade pip
```

```sh
pip install -r requirements.txt
```

Выполнить миграции:

```sh
python manage.py migrate
```

Запустить проект:

```sh
python manage.py runserver
```

# Деплой проекта на удаленный сервер.
## Подключение GitHub к удаленному серверу:
Проверка наличия установленного Git на удаленном сервере:

```sh
sudo apt update
git --version
```

Установка Git в ОС на удаленный сервер (при необходимости):

```sh
sudo apt install git
```

Сгенирировать пару SSH-ключей:

```sh
ssh-keygen
```

Сохранить открытый SSH-ключ в настройках GitHub:

```sh
cat .ssh/id_rsa.pub
```

Склонировать проект на удаленный сервер:

```sh
git clone git@github.com:FeodorPyth/infra_sprint1.git
```

## Настройка бэкенда проекта:
Установить на сервер пакетный менеджер и утилиту для создания виртуального окружения:

```sh
sudo apt install python3-pip python3-venv -y
```

Перейти в директорию backend-приложения проекта:

```sh
cd infra_sprint1/backend/
```

Создать виртуальное окружение:

```sh
python3 -m venv venv
```

Активировать виртуальное окружение:

```sh
source venv/bin/activate
```

Обновить pip в виртуальном окружении:

```sh
pip install --upgrade pip
```

Установить зависимости:

```sh
pip install -r requirements.txt
```

Применить миграции:

```sh
python manage.py migrate
```

Создать суперпользователя:

```sh
python manage.py createsuperuser
```

Перейти в директорию с файлом settings.py и выполнить команду:

```sh
nano settings.py
```

Отредактировать этот файл — в список ALLOWED_HOSTS добавить:

```sh
ALLOWED_HOSTS = ['IP вашего сервера', '127.0.0.1', 'localhost']  
```

## Настройка фронтенд проекта:
Установить на сервер пакетный менеджер npm:

```sh
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt install -y nodejs 
```

Установить зависимости для фронтенд-приложения (нужно перейти в директорию infra_sprint1/frontend/):

```sh
npm i  
```

## Установка и запуск Gunicorn:
На удалённом сервере при активированном виртуальном окружении проекта установитm пакет:

```sh
pip install gunicorn==20.1.0   
```

В файле settings.py настроить константу DEBUG:

```sh
DEBUG = False 
```

Создать юнит для Gunicorn.
В директории /etc/systemd/system/ создать файл gunicorn_kittygram.service и открыть его в Nano:

```sh
sudo nano /etc/systemd/system/gunicorn.service
```

Добавить этот код без комментариев в файл gunicorn_kittygram.service и сохранить изменения:

```sh
[Unit]
Description=gunicorn daemon
After=network.target
[Service]
User=<имя_пользователя_в_системе>
WorkingDirectory=/home/<имя_пользователя_в_системе>/infra_sprint1/backend
ExecStart=/home/<имя_пользователя_в_системе>/infra_sprint1/backend/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
[Install]
WantedBy=multi-user.target
```

Запустите процесс gunicorn_kittygram.service:

```sh
sudo systemctl start gunicorn_kittygram
```

Добавить процесс Gunicorn в список автозапуска ОС на удалённом сервере:

```sh
sudo systemctl enable gunicorn_kittygram
```

Проверить работоспособность запущенного демона:

```sh
sudo systemctl status gunicorn_kittygram
```

## Установка и настройка Nginx:
Находясь на удалённом сервере, из любой директории выполнить команду:

```sh
sudo apt install nginx -y
```

Указать файрволу, какие порты должны остаться открытыми:

```sh
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
```

Включить файрвол:

```sh
sudo ufw enable
```

Собрать статику фронтенд-приложения.
Перейти в директорию infra_sprint1/frontend и выполнить команду:

```sh
npm run build 
```

Скопировать в системную директорию веб-сервера содержимое папки .../frontend/build/:

```sh
sudo cp -r /home/<имя_пользователя_в_системе>/infra_sprint1/frontend/build/. /var/www/kittygram/ 
```

Через редактор Nano открыть файл settings.py, указать новые значения для констант STATIC_URL и STATIC_ROOT:

```sh
STATIC_URL = '/static_backend/'
STATIC_ROOT = BASE_DIR / 'static_backend' 
```

При активированном виртуальном окружении перейти в директорию с файлом manage.py и выполнить команду:

```sh
python manage.py collectstatic
```

Перейти в корень проекта Kittygram и скопировать директорию static_backend/ в системную директорию:

```sh
sudo cp -r /home/<имя_пользователя_в_системе>/infra_sprint1/backend/static_backend/ /var/www/kittygram/
```

Перезапустить Gunicorn:

```sh
sudo systemctl restart gunicorn_kittygram
```

Через редактор Nano открыть файл конфигурации веб-сервера:

```sh
sudo nano /etc/nginx/sites-enabled/default
```

Удалить все настройки из файла, записать и сохранить новые:

```sh
  server {

      listen 80;
      server_name публичный_ip_удаленного_сервера;

      location /api/ {
          proxy_pass http://127.0.0.1:8080;
      }
      
      location /admin/ {
  	    proxy_pass http://127.0.0.1:8000;
  		}
  	
      location / {
          root   /var/www/kittygram;
          index  index.html index.htm;
          try_files $uri /index.html;
      }

 }
```

## Добавление доменного имени в настройки Django:
Перейти в директорию /infra_sprint1/backend/kittygram_backend, с помощью Nano открыть файл settings.py и добавить в список ALLOWED_HOSTS полученное доменное имя:

```sh
ALLOWED_HOSTS = ['публичный_ip_удаленного_сервера', '127.0.0.1', 'localhost', 'ваш-домен'] 
```

Сохранить изменения и перезапустить Gunicorn:

```sh
sudo systemctl restart gunicorn_kittygram
```

Изменить конфигурационный файл Nginx:

```sh
sudo nano /etc/nginx/sites-enabled/default
```

Найти в файле строку, которая начинается с server_name:

```sh
server {
...
    server_name <публичный_ip_удаленного_сервера> <ваш-домен>;
...
}
```

После этого проверить конфигурацию и перезагрузить её командой:

```sh
sudo nginx -t
sudo systemctl reload nginx
```

## Получение и настройка SSL-сертификата:
Установить пакетный менеджер snap:

```sh
sudo apt install snapd
sudo snap install core; sudo snap refresh core
```

Установить certbot на удалённый сервер:

```sh
sudo snap install --classic certbot
```

Создать ссылку на certbot в системной директории:

```sh
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Запустить certbot и получить SSL-сертификат:

```sh
sudo certbot --nginx
```

После ответа на вопросы certbot отправит ваши данные на сервер Let's Encrypt, и там будет выпущен сертификат, который автоматически сохранится на вашем сервере в системной директории /etc/ssl/. Также будет автоматически изменена конфигурация Nginx: в файл /etc/nginx/sites-enabled/default добавятся новые настройки и будут прописаны пути к сертификату.

Перезагрузить конфигурацию Nginx:

```sh
sudo systemctl reload nginx
```

## Автор:
[FeodorPyth](https://github.com/FeodorPyth)
