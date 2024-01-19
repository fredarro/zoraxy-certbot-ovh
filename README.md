# zoraxy-certbot-ovh
permet la création d'un LXC sur Proxmox pour Zoraxy avec Certbot pour la génération de certificat wildcard

A partir du script qui permet la création du LXC sur Proxmox:

```bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/zoraxy.sh)"```

Mise a jour des paquets plus install de Certbot avec plugin pour OVH
```
apt update && apt upgrade
apt install certbot python3-certbot-dns-ovh -y
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
Création des key pour API OVH avec ces privilèges: sur https://api.ovh.com/createToken/
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

Commande de renouvelementautomatique:
```
certbot renew --dry-run```

```                                                                          


Création des liens symbolique vers le répertoire Zoraxy avec le bon nom:
```
ln -s /etc/letsencrypt/live/fredarro.ovh/fullchain.pem /opt/zoraxy/src/conf/certs/default.crt
ln -s /etc/letsencrypt/live/fredarro.ovh/privkey.pem /opt/zoraxy/src/conf/certs/default.key
```
