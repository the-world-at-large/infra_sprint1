# Kittygram
***
## Kittygram - это проект для обмена фотографиями - https://kittttygrammm.hopto.org
***
## Технологии:
* Python 3.10
* Django3
* Nginx
* Gunicorn
* React
* djangorestframework
* Certbot
***
## Установка на локальном сервере
1. Клонируйте проект из репозитория:
```git clone git@github.com:the-world-at-large/infra_sprint1.git```
2. Создайте виртуальное окружение: ```python3 -m venv venv```
3. Установите зависимости: ```pip install -r requirements.txt```
4. Создайте файл ***.env*** в корневой папке (в папке с manage.py), добавьте SECRET_KEY
***
# Развёртывание проекта на удалённом сервере
***
 ### Подключение к GitHub
Установите Git: 

```sudo apt install git```

Создайте пару SSH-ключей: 

```ssh-keygen```

Сохраните открытый ключ в учётной записи GitHub: 

```cat .ssh/id_rsa.pub```

Клонируйте проект на удалённый сервер: 

```git clone git@github.com:Your_account/Project_name.git```  
***
### Запуск бэкенда
Установите менедер пакетов и утилиту для виртуального окружения на сервере:  
```sudo apt install python3-pip python3-venv -y```  
Создайте виртуальное окружение:  
```python3 -m venv venv ```  
Активируйте виртуальное окружение:  
```source venv/bin/activate```
Установите зависимости:  
```pip install -r requirements.txt```  
Выполните миграции:   
```python manage.py migrate```  
Создайте супер-пользователя:  
```python manage.py createsuperuser```
В файле settings.py в ALLOWED_HOSTS добавьте IP-адрес сервера, а также '127.0.0.1', 'localhost' и доменное имя.  
***
### Запуск фронтенда
Установите менджер пакетов npm на сервере:  
```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\```  
```sudo apt-get install -y nodejs```
Установите зависимости фронтенда из директории ***infra_sprint1/frontend***:  
``npm i``  
***
### Установка и запуск Gunicorn
```pip install gunicorn==20.1.0```
Создайте юнит для Gunicorn:  
```sudo nano /etc/systemd/system/gunicorn.service ```  
В файле gunicorn.service опишите конфигурацию процесса:
```html
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=yc-user 
WorkingDirectory=/home/yc-user/taski/backend/
ExecStart=/home/yc-user/taski/backend/venv/bin/gunicorn --bind 0.0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target  
```
Мой пример конфигурации Gunicorn доступен [Здесь](https://github.com/the-world-at-large/infra_sprint1/blob/main/infra/gunicorn_kittygram.service)

Запустите процесс gunicorn.service:  
```sudo systemctl start gunicorn```  
Добавьте процесс в автозапуск:  
```sudo systemctl enable gunicorn```  
***
### Установка nginx
```sudo apt install nginx -y```  
Запустите Nginx:  
```sudo systemctl start nginx```  
Укажите брандмауэру порты, которые должны оставаться открытыми:
```sudo ufw allow 'Nginx Full'```  
```sudo ufw allow OpenSSH```  
Включите брандмауэр:  
```sudo ufw enable```
***

### Сбор статики фронтенда
Из директории ***infra_sprint1/frontend*** запустите:  
```npm run build```
```sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/```  
Опишите настройки обработки статики фронтенда:   
``` sudo nano /etc/nginx/sites-enabled/default```  
Удалите все настройки из файла и добавьте:  
```html
server {
    server_name YOUR_IP YOUR DOMAIN;
    client_max_body_size 32m;
    server_tokens off;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }
    location / {
        root /var/www/kittygram;
        index index.html index.html index.htm;
        try_files $uri /index.html;
    }
```
Мой пример конфигурации Nginx доступен [здесь](https://github.com/the-world-at-large/infra_sprint1/blob/main/infra/default)

***
### Настройка работы с бэкендом
В файле ***settings.py*** внесите изменения (добавьте):  
```STATIC_URL = 'static_backend'```
```STATIC_ROOT = BASE_DIR / 'static_backend'```  
С активированным виртуальным окружением выполните команду:  
```python manage.py collectstatic```
Перейдите в корневую папку проекта и выполните команду:  
```sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/```  
***
### Настройка шифрования
Установите ***certbot***:  
```sudo apt install snapd```

```sudo snap install core; sudo snap refresh core```  

```sudo snap install --classic certbot```

```sudo ln -s /snap/bin/certbot /usr/bin/certbot```

Давайте запустим certbot и получим SSL-сертификат:

```sudo certbot --nginx```

Перезагрузите конфигурацию Nginx:

```sudo systemctl reload nginx```  

Настройте автоматическое обновление SSL-сертификата:

```sudo certbot certificates```  

```sudo certbot renew --dry-run```  

### Настройка мониторинга и сбора ошибок

Зарегистрируйтесь на сайте <small>[Uptimerobot](https://uptimerobot.com/).  
После этого на главной панели нажмите зелёную кнопку ***Add New Monitor***, в открывшемся окне укажите тип монитора: HTTP(s), придумайте имя для этого монитора, введите URL, который вы хотите мониторить.


Автор проекта | Мой вебсайт
------------- | -------------
[the-world-at-large](https://images3.memedroid.com/images/UPLOADED449/62a36080ec892.jpeg)
