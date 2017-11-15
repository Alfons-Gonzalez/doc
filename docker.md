# Docker 


## Part 1: Instalacio

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce -y
mkdir /etc/docker
vi /etc/docker/daemon.json
cat /etc/docker/daemon.json
{
  "storage-driver": "devicemapper"
}

systemctl start docker
systemctl enable docker

si l'ha de fer servir l'usuari alfons cal afegir-lo al grup docker:

usermod alfons -a -G docker

## Part 2: Us

1.- Test que va ok:

[alfons@kubo ~]$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9a0669468bf7: Pull complete 
Digest:
sha256:0e06ef5e1945a718b02a8c319e15bae44f47039005530bc617a5d071190ed3fc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

[alfons@kubo ~]$

2.- Buscar una imatge:

[alfons@kubo ~]$ docker search httpd


Veiem la llista de imatges disponibles i ens diu si es la oficial o no (millor
fer servir imatges oficials)

3.- baixar una imatge:

[alfons@kubo ~]$ docker pull mariadb
Using default tag: latest
latest: Pulling from library/mariadb
85b1f47fba49: Pull complete 
5671503d4f93: Pull complete 
3b43b3b913cb: Pull complete 
4fbb803665d0: Pull complete 
f70c53a1be24: Pull complete 
ab247c7432b9: Pull complete 
5437523d0396: Pull complete 
02185372c549: Pull complete 
ee8416aab538: Pull complete 
10247ed22fa3: Pull complete 
0e0e5b5aa0b7: Pull complete 
Digest:
sha256:c25fb0ada1733c736e13994d210e77b93562fdf61abdd8cb9d32e9f1489a9fbb
Status: Downloaded newer image for mariadb:latest
[alfons@kubo ~]$


3.- Baixar imatges

[alfons@kubo ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED SIZE
mariadb             latest              7fcf8c1a96d2        9 days ago 397MB
hello-world         latest              725dcfab7d63        10 days ago 1.84kB
[alfons@kubo ~]$ 

4.- llistar containers estan en marxa (amb docker ps -a es veuen tots)

[alfons@kubo ~]$ docker ps

Engego un apache i vinculo el /var/www/html amb un volum al filesystem:

[alfons@kubo ~]$ docker run -d -it --name=apachetest -v apache-data:/var/www/html
httpd
Unable to find image 'httpd:latest' locally
latest: Pulling from library/httpd
85b1f47fba49: Already exists 
45bea5eb3b59: Pull complete 
d360abbf616c: Pull complete 
91c7cdd03f84: Pull complete 
30623dd230a8: Pull complete 
cc21a2e04dd3: Pull complete 
f789cd8382be: Pull complete 
Digest:
sha256:8ac08d0fdc49f2dc83bf5dab36c029ffe7776f846617335225d2796c74a247b4
Status: Downloaded newer image for httpd:latest
b173bb8a3f82f12a25731f68ee37676fb1f7029cd533c18d052820c2faf2e223
[alfons@kubo ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED
STATUS              PORTS               NAMES
b173bb8a3f82        httpd               "httpd-foreground"   7 seconds ago
Up 5 seconds        80/tcp              apachetest
[alfons@kubo ~]$

[alfons@kubo ~]$ docker volume ls
DRIVER              VOLUME NAME
local               apache-data
[alfons@kubo ~]$

[alfons@kubo ~]$ docker volume inspect apache-data
[
    {
        "CreatedAt": "2017-11-14T10:42:53+01:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/apache-data/_data",
        "Name": "apache-data",
        "Options": {},
        "Scope": "local"
    }
]


tot i aixi no es el qeu volem:

a) el volum al que he associat /var/www/html al container no es accessible per
el usuari alfons de manera facil

b) el port 80 només està escoltant al container

Parem i esborrem el contenidor:

[alfons@kubo ~]$ docker stop  apachetest
apachetest
[alfons@kubo ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED STATUS              PORTS               NAMES 
[alfons@kubo ~]$ docker rm apachetest apachetest
[alfons@kubo ~]$

Em carrego tambe el volum que havia creat:

[alfons@kubo ~]$ docker volume ls
DRIVER              VOLUME NAME
local               apache-data
[alfons@kubo ~]$ docker volume rm apache-data
apache-data
[alfons@kubo ~]$

Torno a executar el container de httpd:

[alfons@kubo ~]$ docker run --name=apachetest -v ~/apache_data:/var/www/html -p 80:80 -d httpd
0e54d14f857aa7b3475e72a8856c0b609dee404e196f1652f94a6d10a59d2e69
[alfons@kubo ~]$

[alfons@kubo ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED 		  STATUS              PORTS                NAMES
0e54d14f857a        httpd               "httpd-foreground"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   apachetest
[alfons@kubo ~]$ 

I ja podem anar a http://172.20.16.95 i funciona (ip del host kubo)

ha creat també el directori apache_data:

[alfons@kubo ~]$ ls
apache_data
[alfons@kubo ~]$

Pero no puc posar res pq ha estat creat com a root

[alfons@kubo ~]$ ls -ld apache_data/
drwxr-xr-x. 2 root root 6 Nov 14 11:01 apache_data/
[alfons@kubo ~]$ 

miro a veure si realment funciona:

[root@kubo ~]# chown alfons:alfons /home/alfons/apache_data

No funciona pq el container del httpd te els paths de apache
compilat...(/usr/local/apache2/htdocs...)

Faig servir el 'docker cp' per copiar el fitxer:

[alfons@kubo ~]$ docker cp apache_data/index.html apachetest:/usr/local/apache2/htdocs/index.html
[alfons@kubo ~]$

funciona ok. Em carrego la imatge, falta seguir provant el tema dels volums.

## Part 3: Seguretat

Amb el selinux actiu, vaig a provar comprometre el sistema fent anar volumes


[alfons@kubo ~]$ docker run -it -v /:/home/host -d centos 

[alfons@kubo ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED 		STATUS              PORTS               NAMES
1ec2d53f7645        centos              "/bin/bash"         17 seconds ago Up 15 seconds                           quirky_brahmagupta
[alfons@kubo ~]$


[alfons@kubo ~]$ docker exec -it 1ec2d53f7645 /bin/bash
[root@1ec2d53f7645 /]# 

[root@1ec2d53f7645 /]# ls /home/host/
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin
srv  sys  tmp  usr  var
[root@1ec2d53f7645 /]# vi /home/host/root/zz
[root@1ec2d53f7645 /]#

[root@kubo ~]# pwd
/root
[root@kubo ~]# cat zz
Just testing

[root@kubo ~]# 


sembla que no puc fer un chroot tan panxo:

[root@1ec2d53f7645 /]# chroot /home/host
sh-4.2# useradd hacking
useradd: failure while writing changes to /etc/passwd
sh-4.2#
pero si que he pogut modificar fitxers a ma:


[root@1ec2d53f7645 /]# vi /home/host/etc/passwd
[root@1ec2d53f7645 /]# vi /home/host/etc/shadow

[root@1ec2d53f7645 /]# chroot /home/host
sh-4.2# passwd hack
passwd: SELinux denying access due to security policy.
sh-4.2#


[alfons@kubo ~]$ su - hack
Password: 
Last login: Tue Nov 14 11:55:37 CET 2017 from moebius.prib.upf.edu on pts/0
-bash-4.2#


So ==> Hem de mirar com configurar SElinux per evitar que desde el docker es
modifiquin fitxers (o es faci el mount)


## Part 4: docker-compose

1.- Instalacio:

curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version

2.- Us
Aixo ens permet desplegar diversos contenedors conjuntament:

[root@kubo wordpress]# pwd
/root/wordpress
[root@kubo wordpress]#

[root@kubo wordpress]# cat wp.yml 
version: '3.2'

services:

  wordpress:
    image: wordpress
    restart: always
    container_name: wordpress
    depends_on:
	- mysql
    ports:
      - 80:80
    privileged: true
    links:
      - mysql
    environment:
      WORDPRESS_DB_PASSWORD: no_tocar

  mysql:
    image: mysql
    restart: always
    container_name: mysql
    expose:
	- 3306
    environment:
      MYSQL_ROOT_PASSWORD: no_tocar


[root@kubo wordpress]#

[root@kubo wordpress]# docker-compose -f wp.yml up

Baixa i instal.la el que cal


[root@kubo wordpress]# docker-compose -f wp.yml ps
  Name                 Command               State         Ports       
-----------------------------------------------------------------------
mysql       docker-entrypoint.sh mysqld      Up      3306/tcp          
wordpress   docker-entrypoint.sh apach ...   Up      0.0.0.0:80->80/tcp
[root@kubo wordpress]#

I funciona (podem instal.lar el wordpress)

El paro:

[root@kubo wordpress]# docker-compose -f wp.yml stop
Stopping wordpress ... done
Stopping mysql     ... done
[root@kubo wordpress]# 

[root@kubo wordpress]# docker-compose -f wp.yml ps
  Name                 Command               State    Ports
-----------------------------------------------------------
mysql       docker-entrypoint.sh mysqld      Exit 0        
wordpress   docker-entrypoint.sh apach ...   Exit 0        
[root@kubo wordpress]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED
SIZE
wordpress           latest              fcf3e41b8864        9 days ago
408MB
mariadb             latest              7fcf8c1a96d2        9 days ago
397MB
mysql               latest              5709795eeffa        9 days ago
408MB
httpd               latest              74ad7f48867f        10 days ago
177MB
hello-world         latest              725dcfab7d63        10 days ago
1.84kB
centos              latest              d123f4e55e12        10 days ago
197MB
[root@kubo wordpress]#



## Part 5: Engeguem 2 containers d'una mateixa imatge 

Per exemple 2 contenidors, cadascun d'ells amb el seu apache, en un port
diferent, i desde l'apache del host farem proxypass 

1.- Instalem el primer container

[root@kubo ~]# docker run --name=web1 -v /web1:/usr/local/apache2/htdocs -p
8001:80 -d httpd
5a7b6eb3f4e64489db52f18116cabd4d03c455a60ade1d97897eb868d41a765e
[root@kubo ~]# docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED       STATUS              PORTS                  NAMES
5a7b6eb3f4e6        httpd               "httpd-foreground"   5 seconds ago Up 3 seconds        0.0.0.0:8001->80/tcp   web1
[root@kubo ~]#

[root@kubo ~]# cat /web1/index.html 
<html>
web del container web1
</html>
[root@kubo ~]#

A kubo isntalo un apache, si vaig a http://172.20.16.95 veig la web de
l'apache de CentOS

Si vaig a http://172.20.16.95:8001 veig la web del container

Ara a l'apache de kubo posem:

[root@kubo conf.d]# cat web1.conf 

ProxyPass /web1 http://localhost:8001
ProxyPassReverse /web1  http://localhost:8001
[root@kubo conf.d]# 

Pero quan intentem accedir via web dona un 'service unavailable' i mirant els
logs:

[Wed Nov 15 10:08:11.127820 2017] [proxy:error] [pid 13081] (13)Permission
denied: AH00957: HTTP: attempt to connect to 127.0.0.1:8001 (localhost) failed
[Wed Nov 15 10:08:11.127890 2017] [proxy:error] [pid 13081] AH00959:
ap_proxy_connect_backend disabling worker for (localhost) for 60s
[Wed Nov 15 10:08:11.127904 2017] [proxy_http:error] [pid 13081] [client
172.20.16.221:37583] AH01114: HTTP: failed to make connection to backend:

El problema no sembla que localhost no escolti:

[root@kubo ~]# telnet localhost 8001
Trying ::1...
Connected to localhost.
Escape character is '^]'.
^]

telnet> quit
Connection closed.
[root@kubo ~]#


El problema és el selinux (amb setenforce 0 ja funciona) De moment ho deixo
aixi per seguir fent proves

pero ==> mirar selinux config per a poder fer proxypass i/o per poder accedir
a /web1 


2.- crear n segon container:

[root@kubo ~]# docker run --name=web2 -v /web2:/usr/local/apache2/htdocs -p
8002:80 -d httpd
af6a92f051539ea48a155648c7b699df10e7a7af6b31f0b1826f9594942aac2f
[root@kubo ~]# 

[root@kubo ~]# docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED 		STATUS              PORTS                  NAMES
af6a92f05153        httpd               "httpd-foreground"   9 seconds ago 	Up 6 seconds        0.0.0.0:8002->80/tcp   web2
5a7b6eb3f4e6        httpd               "httpd-foreground"   12 minutes ago 	Up 12 minutes       0.0.0.0:8001->80/tcp   web1
[root@kubo ~]#


[root@kubo ~]# cat /web2/index.html
<html>
web del container web2
</html>
[root@kubo ~]#


[root@kubo conf.d]# cat web2.conf 
ProxyPass /web2 http://localhost:8002
ProxyPassReverse /web2 http://localhost:8002
[root@kubo conf.d]#

NOTA:

veiem que encara que a nivell de firewalld esta capat, la maquina es
accessible desde fora als ports 8001 i 8002...


[root@kubo ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: 
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="172.20.16.0/24" service name="ssh"
accept
[root@kubo ~]#

[alfons@moebius ~]$ nmap 172.20.16.95 -PN

Starting Nmap 5.00 ( http://nmap.org ) at 2017-11-15 10:34 CET
Interesting ports on kubo.prib.upf.edu (172.20.16.95):
Not shown: 997 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh
8001/tcp open  unknown
8002/tcp open  teradataordbms

Nmap done: 1 IP address (1 host up) scanned in 5.05 seconds
[alfons@moebius ~]$ telnet 172.20.16.95 8001
Trying 172.20.16.95...
Connected to 172.20.16.95.
Escape character is '^]'.
^]

telnet> quit
Connection closed.
[alfons@moebius ~]$ 

3.- Hem de mirar com fer que docker/firewalld parlin ok:

## Part 6: Xarxa

1.- veure que hi ha

[root@kubo ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
28bc10147ab4        bridge              bridge              local
d89bacee4ba5        host                host                local
3de027785248        none                null                local
ec72043b50eb        wordpress_default   bridge              local
[root@kubo ~]#


[root@kubo ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP
qlen 1000
    link/ether 62:35:1b:39:aa:9a brd ff:ff:ff:ff:ff:ff
    inet 172.20.16.95/24 brd 172.20.16.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::284b:1b30:c887:c3e3/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state
DOWN 
    link/ether 02:42:7e:9f:f5:52 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:7eff:fe9f:f552/64 scope link 
       valid_lft forever preferred_lft forever
18: br-ec72043b50eb: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc
noqueue state DOWN 
    link/ether 02:42:28:10:f2:c2 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 scope global br-ec72043b50eb
       valid_lft forever preferred_lft forever
    inet6 fe80::42:28ff:fe10:f2c2/64 scope link 
       valid_lft forever preferred_lft forever
[root@kubo ~]#







