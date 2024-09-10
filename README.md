# linux-10-11

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

