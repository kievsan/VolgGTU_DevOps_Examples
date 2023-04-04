# Лабораторная №# "Nginx - некоторые нюансы при настройкке конфигурации сайта"

## Этап 1. Создание SSL сертификата

Создадим свой сертификат для сайта. Но прежде чем сгенерировать ключ, скачаем openssl 

```
sudo apt install openssl
```

Далее генерируем файл с ключом следующей командой и задаём ключевую фразу

```
openssl genrsa -des3 -out YOUR_KEY.key 4096
```

![Снимок экрана 2023-04-04 075523](https://user-images.githubusercontent.com/66121979/229875593-e78a7c7c-0757-41f1-8a92-44c1679d1478.png)


Создаём сертификат на основе ключа и заполняем данные о нём
```
openssl req -new -key ca.key -out ca.csr
```
![Снимок экрана 2023-04-04 080018](https://user-images.githubusercontent.com/66121979/229875571-88576832-3aa8-46b9-b88b-48205cd68fbb.png)


![Снимок экрана 2023-04-04 080238](https://user-images.githubusercontent.com/66121979/229875539-d04c5a71-ceeb-4eab-bc11-a9c1759b4045.png)


![Снимок экрана 2023-04-04 080555](https://user-images.githubusercontent.com/66121979/229875818-898b7cd0-b9d0-4206-9b90-0e082a331273.png)

Проверим наличие содержимого файла сертификата

![Снимок экрана 2023-04-04 080833](https://user-images.githubusercontent.com/66121979/229875938-34bac30f-db38-4c1b-9601-542bb816f117.png)

Проверяем сигнатуру и копируем файлы созданного сертификата в каталог nginx

![image](https://user-images.githubusercontent.com/66121979/229876064-a42c51ed-2d70-4bd9-ba2b-181c73e12590.png)


## Этап 2. Изменение конфигурационного файла nginx

Чтобы сертификат работал, нужно изменить файл nginx.conf:
```
 server {
        listen       80;
        server_name  server.local;
return 301 https://$host$request_uri;
```
После редиректа работаем с портом 443, который и понадобится для теста сесртификата:

```
server {
                listen 443 ssl http2;
		
                server_name server.local;	
	
		proxy_read_timeout 720s;
proxy_connect_timeout 720s;
proxy_send_timeout 720s;
client_max_body_size 50m;

proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-Proto $scheme;

ssl_certificate /etc/nginx/ssl/myhostname.crt;
ssl_certificate_key /etc/nginx/ssl/myhostname.key;




# log files
access_log /var/log/nginx/gogs.example.com.access.log;
error_log /var/log/nginx/gogs.example.com.error.log;
                location /git/ {
		proxy_redirect off;
                        proxy_pass http://localhost:3000/;
                }
        }
```


## Этап 3. Особенности редиректа

Посел настройки можно увидеть, что стили сайта опять сломаны. "Пофиксим" ошибку таким странным образом - изменим строку EXTERNAL_URL в файле конфигурации gogs :

```
sudo nano ~/gogs/custom/conf/app.ini
```

Вот как надо исправить строку :

```
tEXTERNAL_URL     = http://localhost:3000/git/
```

Теперь у нас есть сертификат для сайта (хоть и не подтверждённый)

![image](https://user-images.githubusercontent.com/66121979/229881931-4e9776cd-803e-4f6e-ae4f-b81ac6d1dbe0.png)
