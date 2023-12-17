## Создание RPM-пакетов

### Задание

1) Создадать свой RPM пакет на основе nginx с поддержкой openssl
2) Создадать свой репозиторий и разместим там ранее собранный RPM

Реализовать это все либо в Vagrant, либо развернуть у себя через NGINX и дать ссылку на репозиторий.

### Выполнение - пакет NGINX, собиранный с поддержкой openssl.

- Необходимо установить следующие пакеты:
```bash
[root@nginxc8s ~]# yum install -y redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils gcc perl-IPC-Cmd perl-Data-Dumper

CentOS Stream 8 - AppStream                                              3.4 MB/s |  35 MB     00:10    
CentOS Stream 8 - BaseOS                                                 5.5 MB/s |  56 MB     00:10    
CentOS Stream 8 - Extras                                                  12 kB/s |  18 kB     00:01    
CentOS Stream 8 - Extras common packages                                 5.1 kB/s | 6.9 kB     00:01    
Extra Packages for Enterprise Linux 8 - x86_64                           3.0 MB/s |  16 MB     00:05    
Extra Packages for Enterprise Linux 8 - Next - x86_64                    154 kB/s | 368 kB     00:02    
Package wget-1.19.5-11.el8.x86_64 is already installed.
Package yum-utils-4.0.21-23.el8.noarch is already installed.
Package gcc-8.5.0-21.el8.x86_64 is already installed.
Package perl-Data-Dumper-2.167-399.el8.x86_64 is already installed.
Dependencies resolved.
=========================================================================================================
 Package                              Architecture   Version                     Repository         Size
=========================================================================================================
Installing:
 createrepo_c                         x86_64         0.17.7-6.el8                appstream          89 k
 perl-IPC-Cmd                         noarch         2:1.02-1.el8                appstream          43 k
 redhat-lsb-core                      x86_64         4.1-47.el8                  appstream          46 k
 rpm-build                            x86_64         4.14.3-26.el8               appstream         174 k
 rpmdevtools                          noarch         8.10-8.el8                  apps

....
redhat-rpm-config-131-1.el8.noarch                 rpm-build-4.14.3-26.el8.x86_64                     
  rpmdevtools-8.10-8.el8.noarch                      rust-srpm-macros-5-2.el8.noarch                    
  spax-1.5.3-13.el8.x86_64                           time-1.9-3.el8.x86_64                              
  unzip-6.0-46.el8.x86_64                            util-linux-user-2.32.1-43.el8.x86_64               
  zip-3.0-23.el8.x86_64                              zstd-1.4.4-1.el8.x86_64                            

Complete!
```

- Загрузить SRPM пакет NGINX для дальнейшей работы над ним.
>- При установке такого пакета в домашней директории создается древо каталогов для сборки.

```
[root@nginxc8s ~]# wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm
--2023-12-17 08:33:59--  https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm
Resolving nginx.org (nginx.org)... 3.125.197.172, 52.58.199.22, 2a05:d014:edb:5702::6, ...
Connecting to nginx.org (nginx.org)|3.125.197.172|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1086865 (1.0M) [application/x-redhat-package-manager]
Saving to: 'nginx-1.20.2-1.el8.ngx.src.rpm'

nginx-1.20.2-1.el8.ngx.src 100%[=====================================>]   1.04M   946KB/s    in 1.1s    

2023-12-17 08:34:01 (946 KB/s) - 'nginx-1.20.2-1.el8.ngx.src.rpm' saved [1086865/1086865]
```

- Скачать и разархивировать исходники для openssl - они потребуются при сборке.
```
[root@nginxc8s ~]# wget https://github.com/openssl/openssl/archive/refs/heads/OpenSSL_1_1_1-stable.zip
--2023-12-17 08:39:58--  https://github.com/openssl/openssl/archive/refs/heads/OpenSSL_1_1_1-stable.zip
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://codeload.github.com/openssl/openssl/zip/refs/heads/OpenSSL_1_1_1-stable [following]
--2023-12-17 08:39:59--  https://codeload.github.com/openssl/openssl/zip/refs/heads/OpenSSL_1_1_1-stable
Resolving codeload.github.com (codeload.github.com)... 140.82.121.9
Connecting to codeload.github.com (codeload.github.com)|140.82.121.9|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/zip]
Saving to: 'OpenSSL_1_1_1-stable.zip'

OpenSSL_1_1_1-stable.zip       [                 <=>                  ]  11.37M  3.21MB/s    in 3.5s    

2023-12-17 08:40:03 (3.21 MB/s) - 'OpenSSL_1_1_1-stable.zip' saved [11924330]
```
```
[root@nginxc8s ~]# unzip OpenSSL_1_1_1-stable.zip
```

- Установить все зависимости, чтобы в процессе сборки не было ошибок.
```bash
[root@nginxc8s ~]# yum-builddep -y rpmbuild/SPECS/nginx.spec
CentOS Stream 8 - AppStream                                              2.9 kB/s | 4.4 kB     00:01    
CentOS Stream 8 - BaseOS                                                 3.9 kB/s | 3.9 kB     00:01    
CentOS Stream 8 - Extras                                                 3.7 kB/s | 2.9 kB     00:00    
CentOS Stream 8 - Extras common packages                                 4.2 kB/s | 3.0 kB     00:00    
Extra Packages for Enterprise Linux 8 - x86_64                            28 kB/s |  30 kB     00:01    
Extra Packages for Enterprise Linux 8 - Next - x86_64                     26 kB/s |  33 kB     00:01    
Package openssl-devel-1:1.1.1k-9.el8.x86_64 is already installed.
Package systemd-239-78.el8.x86_64 is already installed.
Package zlib-devel-1.2.11-25.el8.x86_64 is already installed.

......
Upgraded:
  openssl-1:1.1.1k-11.el8.x86_64                     openssl-devel-1:1.1.1k-11.el8.x86_64               
  openssl-libs-1:1.1.1k-11.el8.x86_64               
Installed:
  pcre-cpp-8.42-6.el8.x86_64        pcre-devel-8.42-6.el8.x86_64      pcre-utf16-8.42-6.el8.x86_64     
  pcre-utf32-8.42-6.el8.x86_64     

Complete!
```

- Поправить сам [spec](https://gist.github.com/lalbrekht/6c4a989758fccf903729fc55531d3a50) файл, чтобы NGINX собирался с необходимыми нам опциями: **--with-openssl=/root/openssl-1.1.1a**

> По этой [ссылке](https://nginx.org/ru/docs/configure.html) можно посмотреть все доступные опции для сборки.

- Собрать RPM пакет.
```bash
rpmbuild -bb rpmbuild/SPECS/nginx.spec
```

- Проеверить, что пакеты создались.

```
root@nginxc8s ~]# ll rpmbuild/RPMS/x86_64/
total 3148
-rw-r--r--. 1 root root  838260 Dec 17 09:17 nginx-1.20.2-1.el8.ngx.x86_64.rpm
-rw-r--r--. 1 root root 2379964 Dec 17 09:17 nginx-debuginfo-1.20.2-1.el8.ngx.x86_64.rpm
```

- Установить пакет и проверить, что nginx работает.
```
[root@nginxc8s ~]# yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm
CentOS Stream 8 - AppStream                                              2.9 kB/s | 4.4 kB     00:01    
CentOS Stream 8 - BaseOS                                                 4.7 kB/s | 3.9 kB     00:00    
CentOS Stream 8 - Extras                                                 2.9 kB/s | 2.9 kB     00:00    
CentOS Stream 8 - Extras common packages                                 4.2 kB/s | 3.0 kB     00:00    
Extra Packages for Enterprise Linux 8 - x86_64                            23 kB/s |  32 kB     00:01    
Extra Packages for Enterprise Linux 8 - Next - x86_64                     31 kB/s |  33 kB     00:01    
Dependencies resolved.
=========================================================================================================
 Package            Architecture        Version                          Repository                 Size
=========================================================================================================
Installing:
 nginx              x86_64              1:1.20.2-1.el8.ngx               @commandline              819 k

Transaction Summary
=========================================================================================================
Install  1 Package

Total size: 819 k
Installed size: 2.8 M
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                 1/1 
  Running scriptlet: nginx-1:1.20.2-1.el8.ngx.x86_64                                                 1/1 
  Installing       : nginx-1:1.20.2-1.el8.ngx.x86_64                                                 1/1 
  Running scriptlet: nginx-1:1.20.2-1.el8.ngx.x86_64                                                 1/1 
----------------------------------------------------------------------

Thanks for using nginx!

Please find the official documentation for nginx here:
* https://nginx.org/en/docs/

Please subscribe to nginx-announce mailing list to get
the most important news about nginx:
* https://nginx.org/en/support.html

Commercial subscriptions for nginx are available on:
* https://nginx.com/products/

----------------------------------------------------------------------

  Verifying        : nginx-1:1.20.2-1.el8.ngx.x86_64                                                 1/1 

Installed:
  nginx-1:1.20.2-1.el8.ngx.x86_64                                                                        

Complete!
```

- Стартуем.
```
[root@nginxc8s ~]# systemctl start nginx
[root@nginxc8s ~]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2023-12-17 09:28:01 UTC; 14s ago
     Docs: http://nginx.org/en/docs/
  Process: 18422 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 18423 (nginx)
    Tasks: 3 (limit: 4694)
   Memory: 2.9M
   CGroup: /system.slice/nginx.service
           ├─18423 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
           ├─18424 nginx: worker process
           └─18425 nginx: worker process

Dec 17 09:28:01 nginxc8s systemd[1]: Starting nginx - high performance web server...
Dec 17 09:28:01 nginxc8s systemd[1]: Started nginx - high performance web server.
```

- Создать свой репозиторий.
>- Директория для статики у NGINX по умолчанию /usr/share/nginx/html
```
mkdir /usr/share/nginx/html/repo
```

- Скопировать в созданный каталог repo собранный RPM и, например, RPM для установки репозитория Percona-Server.
```bash
cp rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm /usr/share/nginx/html/repo/
```
```
[root@nginxc8s ~]# wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm -O /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
--2023-12-17 09:37:18--  https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
Resolving downloads.percona.com (downloads.percona.com)... 49.12.125.205, 2a01:4f8:242:5792::2
Connecting to downloads.percona.com (downloads.percona.com)|49.12.125.205|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5222976 (5.0M) [application/x-redhat-package-manager]
Saving to: '/usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm'

/usr/share/nginx/html/repo 100%[=====================================>]   4.98M  4.18MB/s    in 1.2s    

2023-12-17 09:37:20 (4.18 MB/s) - '/usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm' saved [5222976/5222976]
```
- Инициализация репозитория.
```bash
[root@nginxc8s ~]# createrepo /usr/share/nginx/html/repo/
Directory walk started
Directory walk done - 2 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
```

- Для прозрачности настроить в NGINX доступ к листингу каталога:
  В location / в файле /etc/nginx/conf.d/default.conf добавим директиву autoindex on. В результате location будет выглядеть так

vi /etc/nginx/conf.d/default.conf

```bash
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        autoindex on;
    }
```

- Проверить синтаксис и перезапустить NGINX.
```bash
[root@nginxc8s ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
```
[root@nginxc8s ~]# nginx -s reload
```

- Выполнить curl для проверки.
```
[root@nginxc8s ~]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          17-Dec-2023 09:38                   -
<a href="nginx-1.20.2-1.el8.ngx.x86_64.rpm">nginx-1.20.2-1.el8.ngx.x86_64.rpm</a>                  17-Dec-2023 09:31              838260
<a href="percona-orchestrator-3.2.6-2.el8.x86_64.rpm">percona-orchestrator-3.2.6-2.el8.x86_64.rpm</a>        16-Feb-2022 15:57             5222976
</pre><hr></body>
</html>
```
### Тестируем репозиторий.
- Добавить его в /etc/yum.repos.d
```
[root@nginxc8s ~]# cat >> /etc/yum.repos.d/otus.repo << EOF
> [otus]
> name=otus-linux
> baseurl=http://localhost/repo
> gpgcheck=0
> enabled=1
> EOF
```
- Преверить, что репозиторий подключился и посмотреть, что в нем есть.

```bash
[root@nginxc8s ~]# yum repolist enabled | grep otus
otus                otus-linux
```
```
[root@nginxc8s ~]# yum list | grep otus
otus-linux                                      100 kB/s | 2.8 kB     00:00    
percona-orchestrator.x86_64                                       2:3.2.6-2.el8                                                     otus     
```
- Установить репозиторий percona-release.

```bash
[root@nginxc8s ~]# yum install percona-orchestrator.x86_64 -y
Last metadata expiration check: 0:03:38 ago on Sun Dec 17 09:53:05 2023.
Dependencies resolved.
==================================================================================================================================================
 Package                                   Architecture                Version                               Repository                      Size
==================================================================================================================================================
Installing:
 percona-orchestrator                      x86_64                      2:3.2.6-2.el8                         otus               
........
Installed:
  jq-1.6-8.el8.x86_64                   oniguruma-6.8.2-2.el8.x86_64                   percona-orchestrator-2:3.2.6-2.el8.x86_64                  

Complete!
```

>- При каждом добавлении файлов, требуется обновить репозиторий. Для это выполнить команду **createrepo /usr/share/nginx/html/repo/**


