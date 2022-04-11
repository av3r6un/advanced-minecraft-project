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

Создадим нового пользователя, который может подключаться с любого IP-адреса и дадим ему права на упрвление таблицами и создание новых пользователей, а также добавим базу и пользователя для PostFix. Если нет нужды в WebMailServer пропускаем вторую часть

```sql
CREATE USER 'superuser'@'%' IDENTIFIED BY 'yoursecretpassword';
GRANT ALL ON *.* to 'superuser' WITH GRANT OPTION;
FLUSH PRIVILEGES;

 # Создание базы данных для postfix.

CREATE DATABASE postfix DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON postfix.* TO 'postfix'@'localhost' IDENTIFIED BY 'yoursecretpassword';
\q
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

Можно проверить встал ли phpMyAdmin перейдя по ссылке pma.yourdomain.com. Но перед этим, нужно перезагрузить nginx httpd и php-fpm, а также выдать права доступа папке NGINX'у.

```bash
chmod -R 775 /web/pma/www
chown -R nginx:nginx /web
systemctl restart nginx httpd php-fpm
```

### - Настроим WebMail Server и WebMail Client

>В данную связку входят несколько зависимостей, сначала мы всех их установим. Каждый настраивается отдельно, занимает некоторое время.
> Если нет нужды в WebMailServer то можно перейти к следующему пункту


Задаем временную зону (в данном примере московское время):

```bash
timedatectl set-timezone Europe/Moscow
yum -y update
```
- Отключение SELinux

Для отключения дополнительного компонента безопасности вводим 2 команды:

```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
```

Теперь установим postfixadmin

```bash
yum -y install php-mysql php-mbstring php-imap
systemctl restart php-fpm
cd /var/tmp && wget https://sourceforge.net/projects/postfixadmin/files/latest/download -O postfixadmin.tar.gz
mkdir /web/postfix && mkdir /web/postfix/www && tar -C /web/postfix/www -xvf postfixadmin.tar.gz --strip-components 1
chown -R apache:apache /web/postfix/www
```
>Несмотря на то, что мы используем веб-сервер nginx, php-fpm по умолчанию, запускается от пользователя apache.

Создаем каталог templates_c внутри папки портала (без него не запустится установка) и добавим конфигурационный файл postfixadmin:

```bash
mkdir /web/postfix/www/templates_c
nano /web/postfix/www/config.local.php
```

И добавим следующее:

```php
<?php

$CONF['configured'] = true;
$CONF['default_language'] = 'ru';
$CONF['database_password'] = 'postfix123';
$CONF['emailcheck_resolve_domain']='NO';

?>
```
> где _configured_ говорит приложению, что администратор закончил его конфигурирование; _default_language_ — используемый язык по умолчанию; _database_password_ — пароль для базы данных, который мы задали на предыдущем шаге; _emailcheck_resolve_domain_ — задает необходимость проверки домена при создании ящиков и псевдонимов.

- Настроим NGINX чтобы postfixadmin открывался на виртуальном домене.

```bash
server {
    listen       80;
    server_name postfix.yourdomain.com;
    set $root_path /usr/share/nginx/html;

    location / {
        root $root_path;
        index index.php;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $root_path$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_param DOCUMENT_ROOT $root_path;
    }
}
```

Переходим по ссылке postfix.yourdomain.com/public/setup.php для первоначальной настройки. Задаём дважды пароль установки и генерируем хэш, нажав на кнопку **Generate setup_password hash**:

![alt-текст](https://www.dmosk.ru/img/instruktions/postfix-centos/01.jpg "Генерация setup_password hash")

После перезагрузки страницы копируем хэш:

![alt-текст](https://www.dmosk.ru/img/instruktions/postfix-centos/02.jpg "Пример хэша postfix")

Теперь открываем конфигурационный файл postfixadmin и вставляем туда скопированную строку:

```bash
nano /web/postfix/www/config.local.php
```

```php
...
$CONF['setup_password'] = '$2y$10...BMK';
```

Теперь нужно перезагрузить страницу и ввести тот пароль, который вы использовали для генерации хэша:

![alt-текст](https://www.dmosk.ru/img/instruktions/postfix-centos/03.jpg "Авторизация postfix_setup")

После этого будет выполнена установка PostFix. После установки в нижней части страницы будет форма добавления суперпользователя, то есть его регистрации.

![alt-текст](https://raw.githubusercontent.com/av3rgun/advanced-minecraft-project/main/src/img/05.jpg "Создание суперпользователя")


- Установим postfix

```bash
yum -y install postfix
groupadd -g 1024 vmail
useradd -d /home/mail -g 1024 -u 1024 vmail -m
```

>Cначала мы создаем группу vmail и guid 1024, после — пользователя vmail с uid 1024 и домашней директорией /home/mail. Обратите внимание, что в некоторых системах идентификатор группы и пользователя 1024 может быть занят. В таком случае необходимо создать другой, а в данной инструкции ниже заменить все 1024 на альтернативный.

Теперь отредактируем конфигурационный файл почтвого сервера:

```bash
nano /etc/postfix/main.cf

 # Изменяем следующие строки

myorigin = $mydomain
...
mydestination = localhost.$mydomain, localhost, localhost.localdomain
...
local_recipient_maps = unix:passwd.byname $alias_maps
...
mynetworks = 127.0.0.0/8
...
inet_interfaces = all
...
inet_protocols = all
...
myhostname = mx01.yourdomain.com

 # В конец дописываем следующее:

virtual_mailbox_base = /home/mail
virtual_alias_maps = proxy:mysql:/etc/postfix/mysql_virtual_alias_maps.cf
virtual_mailbox_domains = proxy:mysql:/etc/postfix/mysql_virtual_domains_maps.cf
virtual_mailbox_maps = proxy:mysql:/etc/postfix/mysql_virtual_mailbox_maps.cf
virtual_minimum_uid = 1024
virtual_uid_maps = static:1024
virtual_gid_maps = static:1024
virtual_transport = dovecot
dovecot_destination_recipient_limit = 1

smtpd_sasl_auth_enable = yes
smtpd_sasl_exceptions_networks = $mynetworks
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth

smtpd_tls_cert_file = /etc/ssl/mail/public.pem
smtpd_tls_key_file = /etc/ssl/mail/private.key
smtpd_use_tls = yes
smtp_use_tls = yes
smtpd_tls_auth_only = yes
smtpd_helo_required = yes
```

Создаем файл с настройками обращения к базе с алиасами:

```bash
nano /etc/postfix/mysql_virtual_alias_maps.cf
 
 # Вставляем следующее

user = postfix 														# database user
password = postfix123												# database password
hosts = localhost													# database host
dbname = postfix 													# database name
query = SELECT goto FROM alias WHERE address='%s' AND active = '1'	# database query (Не изменять)
```

И файл с почтовыми ящиками:

```bash
nano /etc/postfix/mysql_virtual_mailbox_maps.cf

 # Вставляем следующее:

user = postfix 																					# database user
password = postfix123																			# database password
hosts = localhost																				# database host
dbname = postfix 																				# database name
query = SELECT CONCAT(domain,'/',maildir) FROM mailbox WHERE username='%s' AND active = '1'		# database йгукн (Не изменять)
```

Открываем файл master.cf и дописываем в самый конец:

```bash
nano /etc/postfix/master.cf

 # Вставляем следующее:

submission   inet  n  -  n  -  -  smtpd
  -o smtpd_tls_security_level=may
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=/var/spool/postfix/private/auth
  -o smtpd_sasl_security_options=noanonymous
  -o smtpd_sasl_local_domain=$myhostname

smtps   inet  n  -  n  -  -  smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject

dovecot   unix  -  n  n  -  -  pipe
  flags=DRhu user=vmail:vmail argv=/usr/libexec/dovecot/deliver -d ${recipient}
```

Перезапустим postfix

```bash
systemctl restart postfix
```

- Настройка Dovecot


```bash
yum -y install dovecot dovecot-mysql

nano /etc/dovecot/conf.d/10-mail.conf

 # Вставляем следущее:

mail_location = maildir:/home/mail/%d/%u/
```

>Это нужно для установки хранения способа сообщений

Настраиваем слушателя для аутентификации:

```bash
nano /etc/dovecot/conf.d/10-master.conf

 # Вставляем следующее:

service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }
  unix_listener auth-userdb {
    mode = 0600
    user = vmail
    group = vmail
  }
}
```

Настраиваем аутентификацию в Dovecot:

```bash
nano /etc/dovecot/conf.d/10-auth.conf

 # Комментируем одна, снимаем комментарий с другой строки:

#!include auth-system.conf.ext
!include auth-sql.conf.ext
```

Настраиваем использование шифрования:

```bash
nano /etc/dovecot/conf.d/10-ssl.conf

 # Вставляем следующее:

ssl = required
ssl_cert = </etc/ssl/mail/public.pem
ssl_key = </etc/ssl/mail/private.key
```

Настроим автоматическое создание каталогов при первом подключении пользователя к ящику:

```bash
nano /etc/dovecot/conf.d/15-lda.conf

 # Вставляем следующее:

lda_mailbox_autocreate = yes
```

Открываем файл:

```bash
nano /etc/dovecot/conf.d/20-imap.conf

 # Приводим опцию imap_client_workarounds к виду:

imap_client_workarounds = outlook-idle delay-newmail
```

Настраиваем подключение к нашей базе данных:

```bash
nano /etc/dovecot/conf.d/auth-sql.conf.ext

 # Вставляем следущее:

passdb {
  …
  args = /etc/dovecot/dovecot-sql.conf.ext
}

userdb {
  …
  args = /etc/dovecot/dovecot-sql.conf.ext
}

```

Создаем файл с настройками работы с mysql:

```bash
nano /etc/dovecot/dovecot-sql.conf.ext

 # Вставляем следующее

driver = mysql
connect = host=localhost dbname=postfix user=postfix password=postfix123
default_pass_scheme = MD5-CRYPT
password_query = SELECT password FROM mailbox WHERE username = '%u'
user_query = SELECT maildir, 1024 AS uid, 1024 AS gid FROM mailbox WHERE username = '%u'
user_query = SELECT CONCAT('/home/mail/',LCASE(`domain`),'/',LCASE(`maildir`)), 1024 AS uid, 1024 AS gid FROM mailbox WHERE username = '%u'
```

И, напоследок, настраиваем интерфейс, на котором будет слушать dovecot:

```bash
nano /etc/dovecot/dovecot.conf

 # Вставляем следующее:

listen = *
```

### - Генерируем сертификаты безопасности

Создаем каталог, в котором разместим сертификаты:

```bash
mkdir -p /etc/ssl/mail

 # И сгенерируем их следующей командой:

openssl req -new -x509 -days 1461 -nodes -out /etc/ssl/mail/public.pem -keyout /etc/ssl/mail/private.key -subj "/C=RU/ST=SPb/L=SPb/O=Global Security/OU=IT Department/CN=relay.yourdomain.com"
```

Запускаем dovecot:

```bash
systemctl enable dovecot
systemctl start dovecot
```

### - Cоздаём первый почтовый ящик

Переходим в адресной строке бразуере по ссылке:

```url
http://postfix.yourdomain.com/public/
```

Вводим логин и пароль от суперпользователя. Перед нами появляется страница управления учетными записями. Переход в **Список доменов** и выбриаем **Новый домен**. Заполняем формы и нажимаем **Добавить домен**

![alt-текст](https://raw.githubusercontent.com/av3rgun/advanced-minecraft-project/main/src/img/06.jpg "Создание домена")

Теперь **Обзор** -> **Создать ящик**. Вводим данные нового пользователя и нажимаем создать ящик.

### - Устанавливаем и настраиваем Roundcube Webmail

Создадим папку, где будет располагаться наш WebMail, скачаем туда Roundcube, настроим и проверим в действии.

```bash
mkdir /web/webmail & mkdir /web/webmail/www
wget https://github.com/roundcube/roundcubemail/releases/download/1.5.2/roundcubemail-1.5.2.tar.gz
tar -xvf roundcubemail-*.tar.gz --strip-components 1

cp /web/webmail/www/config/config.inc.php.sample /web/webmail/www/config/config.inc.php
nano /web/webmail/www/config/config.inc.php

 # Меняем на наши данные

$config['db_dsnw'] = 'mysql://roundcube:roundcube123@localhost/roundcubemail';
$config['enable_installer'] = true;

$config['drafts_mbox'] = 'Drafts';        #Черновики
$config['junk_mbox'] = 'Junk';            #СПАМ
$config['sent_mbox'] = 'Sent';            #Отправленные
$config['trash_mbox'] = 'Trash';          #Корзина  
$config['create_default_folders'] = true; #Создать папки по-умолчанию
```

```bash
chown -R nginx:nginx /web/webmail/www
mysql -u root -p
```

Создаём базу, пользователя и передаём привилегии ему:

```sql
CREATE DATABASE roundcubemail DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcube'@'localhost' IDENTIFIED BY 'yoursecretpassword'
quit
```

```bash
mysql -uroot -p roundcubemail < /usr/share/nginx/html/webmail/SQL/mysql.initial.sql
```

- Установим компоненты необходимыеы для Roundcube Mail

```bash
yum -y install php-pear php-mcrypt php-intl php-ldap php-pear-Net-SMTP php-pear-Net-IDNA2 php-pear-Mail-Mime php-imagick

```

Настроим php:

```bash
nano /etc/php.ini

 # Меняем

date.timezone = "Europe/Moscow"
...
allow_url_fopen = On
```

```bash
systemctl restart php-fpm
```

Перейдём к настройке Roundcube. Идём по пути http://webmail.divine.world/installer/. В самом низу нажимаем по кнопке Next. Если кнопка будет неактивна, проверяем, что нет ошибок (NOT OK). На следующей странице проверяем, что все пункты находятся в состоянии OK. Установка выполнена. 

Изменяем конфиг WebMail, убираем installer:

```bash
nano /web/webmail/www/config/config.inc.php
rm -rf /usr/share/nginx/html/webmail/installer

 # Вставляем следующее:

$config ['enable_installer'] = false
```

Если при отправке сообщения мы видим ошибку авторизации SMTP, открываем конфигурационный файл roundcube и добавляем следующую строчку:

```php
$config['smtp_pass'] = '';
```

На этом, мы закончили с установкой WebMail Client и WebMail Server.