# advanced-minecraft-project
[Tutorial] **How To Create Minecraft Advanced Project with Launcher, Server, Site & TelegramBot**

[Туториал] **Делаем продвинутый проект Minecraft ссобственным лаунчером, сервером, сайтом, почтой и телеграм-ботом**

#### _Короткое вступление_:

>Все процессы, конечно, есть на просторах интернета,
>но они так разбросаны, что я потратил несколько месяцев,
>чтоб собрать все данные в одном месте. 
>Теперь делюсь их с вами. Думаю, они они много кому
>пригодятся. Вам предстоит долгий лонгрид с подробным 
>обьясением каждого пункта. Приятного чтения!

#### _Что понадобиться:_

1. Прямые руки
2. Немного терпения
3. Удалённый сервер
4. Много желания

### _Последовательность действий_:

### - Первоначальная настройка удалённого VPS сервера

>То, где будет распологатся ваш проект
>Лучше брать помощнее, т.к. всё будет
>находиться на одном сервере и чтоб он
>не сошёл с ума, не жалеть оперативную
>память и место на диске. Использовать
>будем операционную систему CentOS 7
>
>Минимальные требования к боксу:
>
>1. 3Gb - RAM
>2. 2-core - CPU
>3. 32Gb - NVMe
>4. CentOS 7 - OS

- Покупка домена

>Пока домен будет проксироваться, мы
>настроим бокс для дальнейшего к нему
>подключения. Брать домен лучше в зоне
>в который будет находиться сервер.
>
>Дешевле и безопаснее.

- Настройка подключения по SSH и sFTP

>Создаём SSH ключ и заливаем его в
>панель управления и подключаем
>его к необходимому боксу

_Команда в Windows_:
```bash
ssh-keygen
```
_Команда в macOS_:
```bash 
ssh-keygen -t rsa
```

- Установка необходимых зависимостей

>Обновляем бокс, приводим в порядок,
>создаём пользователя, даём ему права.
>Это действие необходимо для правильной
>работы модуля NGINX

```bash
sudo yum -y update 
```

- Создаем группу и пользователя

>Мы будем создавать группу для разработчиков нашего проекта и внесём в неё будущий nginx
>Так он сможет задействовать необходимые ему файлы, запускать их и завершать.
>
>Переходим в пользователя root, если этого ещё не сделали. Создаём группу "groupname" и пользователя "username".


```bash
sudo su
groupadd groupname
useradd -d /home/username -G groupname, wheel username
passwd username
```
>##### _Группа wheel нужна для того, чтобы у пользователя были права sudo_
>Комагда _passwd_ нужна для изменения/создания пароля. Она предложит ввести несколько раз
>новый пароль для пользователя _username_

- _(Опционально)_ Аутентификация созданного пользователя по ssh-ключу

>Чтобы войти в бокс по ssh-ключу не под root, нужно создать отдельную папку
>в директории по умолчанию данного пользователя и скопировать туда
>вновь сгенерированный или старый public_key. Затем в конфиге sshd (/etc/ssh/sshd_config)
>поменять строку StrictModes c _"yes"_ на _"no", закомментировать PasswordAuthentification
>или поменять его значение на "no"

```bash 
sudo su
mkdir /home/username/.ssh
cp ~/.ssh/authorized_keys /home/username/.ssh/authorized_keys
systemctl restart sshd
```

### - Настройка Бокса под проект

>Мы хотим сделать проект Minecraft с лаунчером и сервером и обвязать его всем необходимым для хорошего проекта.
>Приавяжем сайт, написанный на python. Добавим WebMail, phpMyAdmin и конвертер скинов на php. И небольшую фичу от 
>автора: Телеграмм бот на пайтоне, через websocket для обслуживания сайта. Очень надеюсь, что у вас всех получится,
>не с первого, но со второго раза точно.

- Установим необходимые зависимости для проекта

```bash
yum -y install epel-release nginx mariadb mariadb-server wget curl unzip nano httpd
```

- Настроим Apache

Apache требует дополнительной настройки при установке в паре с NGINX. Так и NGINX нужно дополнительно настроивать. Чтобы они не конфиктовали между собой, пусть Apache слушает порт 8080, так как NGINX по умолчанию слушает 80 порт.

```bash
nano /etc/httpd/conf/httpd.conf
```

```bash
...
Listen :8080
ServerName yourdomain.com:8080
```

Запустим его и проверим на работоспособность. Если система не вывела ошибок при запуске демона httpd, значит всё настроено правильно и мы можем двигать к следующему пункту.

```bash
systemctl start httpd
systemctl enable httpd
```

- Теперь необходимо установить php последней версии. Делается чем репозиторий remi.

```bash
rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
cd /etc/yum.repos.d/
ls
```

Выйдет список файлов, установленных вместе с репозиторием. Нам нужно установить последнюю версию php. Файл будет называться remi-php81.repo. Его нужно открыть редактором nano и включить (параметр enabled переверсти в положение 1). Пример ниже:

>/// {Встать картнку примера включение новой версии php}

- Дальше будет обычная установка php.

```bash
yum -y install php
yum -y update
yum -y install php-fpm
systemctl enable php-fpm
systemctl start php-fpm
```

- Теперь нужно установить phpMyAdmin.

```bash
cd /var/tmp
wget https://files.phpmyadmin.net/phpMyAdmin/5.1.3/phpMyAdmin-5.1.3-all-languages.zip
```

#### - Теперь отвлечемся от скачивания и настроим Базу данных

Проведём безопасную установку. Первым вопрос оно запросит пароль. Если на машине в первый раз создаётся БД, то пароль будет пустым и просто смело жмите Enter. С каждым следующим вопросом нужно согласиться. (y) На первом шаге необходимо установить пароль root пользователя.

```bash
systemctl start mariadb
systemctl enable mariadb
mysql-secure-installation
```

Создадим пользователя, который сможет подключаться извне и создавать новые таблицы и новых пользователей. Покдлючимся к нашей mariadb с пользователем root и паролем, который вы создали на предыдущем этапе.

```bash
mysql -u root -p
```

Создадим нового пользователя, который может подключаться с любого IP-адреса и дадим ему права на упрвление таблицами и создание новых пользователей.

```sql
CREATE USER 'superuser'@'%' IDENTIFIED BY 'yoursecretpassword';
GRANT ALL ON *.* to 'superuser' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Ну и на последок создадим базу данных для нашего сайта. Куда будет подключаться и сервер и сайт и лаунчер.

```sql
CREATE DATABASE main_site DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```

#### - Включим и настроим PHP-FPM

>Тут все просто, изменяем *что* он будет слушать, от какого *имени* он будет слушать и от какой *группы* он будет
> слушать

```bash
systemctl enable php-fpm
systemctl start php-fpm
nano /etc/php-fpm.d/www.conf
```

- Редактируем файл www.conf

```bash
...
listen = /run/php-fpm/www.sock
listen.owner = nginx
listen.group = nginx
...
php_admin_value[error_log]          = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors]          = on
php_admin_value[memory_limit]       = 128M
...
php_value[session.save_handler]     = files
php_value[session.save_path]        = /var/lib/php/session
php_value[session.wsdl_cache_dir]   = /var/lib/php/wdlcache
...
```

- Перезапускаем php-fpm

```bash
systemctl restart php-fpm
```

### - Настроим phpMyAdmin

Переместим скачанный архив из временной папки, удалим его оттуда и настроим NGINX. Будем настраивать конфиг pma

```bash
mkdir /web && mkdir /web/pma && mkdir /web/pma/www && mkdir /web/pma/logs
mv /var/tmp/phpMyAdmin-*-all-languages.zip /web/pma/www
cd /web/pma/www && unzip phpMyAdmin-*-all-language.zip && rm phpMyAdmin-*-all-languages.zip
```

Так, мы создали папку, в которой будет лежать phpMyAdmin. Теперь настроим Apache, что он правильно слушал поддомен сайта.
Добавляем в самый конец конфигурационного файла Apache следующую строку:

```bash
...
IncludeOptional conf.d/*.conf
```

>Скорее всего она по умолчанию будет присутствовать, нужно будет только расскомментировать её.

После установки phpMyAdmin должен был создаться конфигурационный файл для Apache. Если он не создался или выглядит по другому, приведите его к такому виду.

```bash
nano /etc/httpd/conf.d/phpMyAdmin.conf
```

**Вот пример:**
```bash
Alias /phpMyAdmin /web/pma/www/phpMyAdmin
Alias /phpmyadmin /web/pma/www/phpMyAdmin

<Directory /web/pma/www/phpMyAdmin/>
   AddDefaultCharset UTF-8

   <IfModule mod_authz_core.c>
     <RequireAny>
       #Require ip 127.0.0.1
       #Require ip ::1
     </RequireAny>
   </IfModule>
   <IfModule !mod_authz_core.c>
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>

<Directory /web/pma/www/phpMyAdmin/setup/>
   <IfModule mod_authz_core.c>
     <RequireAny>
       Require ip 127.0.0.1
       Require ip ::1
     </RequireAny>
   </IfModule>
   <IfModule !mod_authz_core.c>
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>

<Directory /web/pma/www/phpMyAdmin/libraries/>
   <IfModule mod_authz_core.c>
     Require all denied
   </IfModule>
   <IfModule !mod_authz_core.c>
     Order Deny,Allow
     Deny from All
     Allow from None
   </IfModule>
</Directory>

<Directory /web/pma/www/phpMyAdmin/setup/lib/>
   <IfModule mod_authz_core.c>
     Require all denied
   </IfModule>
   <IfModule !mod_authz_core.c>
     Order Deny,Allow
     Deny from All
     Allow from None
   </IfModule>
</Directory>

<Directory /web/pma/www/phpMyAdmin/setup/frames/>
   <IfModule mod_authz_core.c>
     Require all denied
   </IfModule>
   <IfModule !mod_authz_core.c>
     # Apache 2.2
     Order Deny,Allow
     Deny from All
     Allow from None
   </IfModule>
</Directory>
```
- Теперь настроим NGINX для phpMyAdmin

```bash
nano /etc/nginx/conf.d/pma.conf
```

```bash
server {
	listen 80;
	server_name pma.yourdomain.com www.pma.yourdomain.com;

	gzip on;
	gzip_disable "msie6";
	gzip_min_length 1000;
	gzip_vary on;
	gzip_proxied expired no-cache no-store private auth;
	gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;

	access_log /web/pma/logs/access.log;
	
	location ~* ^.+\.(jpg|jpeg|gif|png|css|zip|tgz|gz|rar|bz2|doc|docx|xls|xlsx|exe|pdf|ppt|tar|wav|bmp|rtf|js)$ {
		root /web/pma/www;
		expires 10d;
	}
	location / {
    	alias /web/pma/www/phpMyAdmin/;
		index index.php;
		try_files $uri $uri/ /index.php?$args;
		
		allow your.public.ip.address;
		deny all;

		location ~ \.php$ {
				fastcgi_pass unix:/run/php-fpm/www.sock;
    			fastcgi_index index.php;
   				include fastcgi_params;
    			fastcgi_param DOCUMENT_ROOT $document_root;
    			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    			fastcgi_param PATH_TRANSLATED $document_root$fastcgi_script_name;
    			fastcgi_param QUERY_STRING $query_string;
    			fastcgi_param REQUEST_METHOD $request_method;
    			fastcgi_param CONTENT_TYPE $content_type;
    			fastcgi_param HTTP_PROXY "";
		}

	            
	    location ~* ^/(.+.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
	       	alias /web/pma/www/phpMyAdmin/$1;
	    }

    }
}
```


