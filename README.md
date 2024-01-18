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
