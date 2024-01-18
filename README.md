# zoraxy-certbot-ovh
permet la création d'un LXC sur Proxmox pour Zoraxy avec Certbot pour la génération de certificat wildcard

A partir du script qui permet la création du LXC sur Proxmox:

```bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/zoraxy.sh)"```

Mise a jour des paquets plus install de Certbot avec plugin pour OVH
```
apt update && apt upgrade
apt install python3-pip
pip3 install certbot
pip3 install certbot-dns-ovh
```
On place  un nouveau fichier de config ``/etc/logrotate.d/certbot`` avec le contenu suivant :
```
nano /etc/logrotate.d/certbot

```


```
/var/log/letsencrypt/*.log {
    monthly
    rotate 6
    compress
    delaycompress
    notifempty
    missingok
    create 640 root adm
}
```
Création des key pour API OVH aves ces privilèges:
```
GET /domain/zone/*
PUT  /domain/zone/* 
POST  /domain/zone/* 
DELETE  /domain/zone/*
```

stockage dans ``/root/.ovhapi``
```
dns_ovh_endpoint = ovh-eu
dns_ovh_application_key = xxxx
dns_ovh_application_secret = xxxx
dns_ovh_consumer_key = xxxx

```
Propriétées du fichier adaptée
```
chmod 600 /root/.ovhapi

```

Génération du premier certificat:
```
certbot certonly --dns-ovh --dns-ovh-credentials ~/.ovhapi --non-interactive --agree-tos --email fred33430@gmail.fr -d fredarro.ovh -d *.fredarro.ovh

```

Commande de renouvelement:
```
nano /usr/local/sbin/renewCerts.sh
```

```                                                                          
#!/bin/bash

certbot renew --cert-name fredarro.ovh
```

Propriétées du fichier adaptée
```
chmod +x /usr/local/sbin/renewCerts.sh
```


mise en place du cron
```
nano /etc/crontab
```

Ajout de la ligne:
```
0 0 1 * * /usr/local/sbin/renewCerts.sh > /dev/null 2>&1
```
relance du service cron
```
sudo systemctl restart cron
```

commande copy du certificat vers répertoire Zoraxy:
```
nano /usr/local/sbin/copyCerts.sh
```

```
#!/bin/bash


cp /etc/letsencrypt/live/fredarro.ovh/fullchain.pem default.crt
cp /etc/letsencrypt/live/fredarro.ovh/privkey.pem default.key
```

reste à faire le INCRON pour lancer ``/usr/local/sbin/copyCerts.s`` lors du changement de Cert
