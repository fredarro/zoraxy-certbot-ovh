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
certbot renew --dry-run

```                                                                          


Création des liens symbolique vers le répertoire Zoraxy avec le bon nom:
```
ln -s /etc/letsencrypt/live/fredarro.ovh/fullchain.pem /opt/zoraxy/src/conf/certs/default.crt
ln -s /etc/letsencrypt/live/fredarro.ovh/privkey.pem /opt/zoraxy/src/conf/certs/default.key
```


Nous verrons ci-dessous comment installer ZORAXY directement sur une machine virtuel linux (ubuntu 22.04 LTS) avec la configuration et l'utilisation de systemD pour lancer/arreter le service zoraxy

Sur une ubuntu 22/04 LTS fraichement installer installer GO (il faut une version minimum go 1.20)
Installation de GO :

1er méthode:
```
1. sudo apt-get update
2. wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz
3. sudo tar -xvf go1.21.6.linux-amd64.tar.gz
4. sudo mv go /usr/local
5. export GOROOT=/usr/local/go
6. export GOPATH=$HOME/go
7. export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
8. source ~/.profile
```

2eme méthode:
```
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go
```

Vérifier la version de Go installer: (résultat attendu -> go version go1.21.6 linux/amd64)
```
go version
```

Installation Zoraxy depuis la source (en root ou utiliser la commande sudo avant de passer les commandes suivantes)
```
git clone https://github.com/tobychui/zoraxy
cd ./zoraxy/src/
go mod tidy
go build
```

Lors du lancement de zoraxy des paramètres peuvent être ajouter:

fastgeoip (true / false)

Permet d'utiliser plus de ram pour GEO-IP_localisation, par default = False, pour l'activer utiliser -fastgeoip=true

noauth (true / false)

Si vous souhaiter accedé a zoraxy sans authentification ajouter le paramètre -noauth=true.
ATTENTION c'est un risque de sécurité si jamais vous n'avez pas d'outils pour l'authentification lors de l'accès a zoraxy

sshlb (true / false)

Permet de se connecter en ssh grace à l'interface loopback du serveur depuis l'interface d'administration de zoraxy sur un host.
Pour activer cette option il faut ajouter -sshlb=true

Lancez zoraxy avec la commande suivante: (dans l'exemple nous activons le fast-GEO-IP et l'accès SSH:
```
sudo ./zoraxy -port=:8000 -fastgeoip=true -sshlb=true
```

Une fois lancer vous devriez pouvoir avoir accès a Zoraxy en allant sur: 
```
http://IP_DE_VOTRE_MACHINE:8000
```

Si tel et le cas tous est bien installer, faite un CTRL+C pour arreter le service.

récupéré le chemin ou se trouve le binaire zoraxy en faisant: (A retenir pour utiliser le chemin dans le fichier zoraxy.service ci-dessous)
```
pwd
```

Nous allons maintenant configurer systemD pour lancer et arreter le service Zoraxy comme tout autre service linux.

Pour cela creer le fichier suivant "/etc/systemd/system/zoraxy.service":

```
touch /etc/systemd/system/zoraxy.service
```
et editer le fichier:
```
nano /etc/systemd/system/zoraxy.service
```

Puis ajouter les lignes ci-dessous:

```
[Unit]
Description=Zoraxy reverse-proxy Service
ConditionPathExists=RESULTAT_DE_LA_COMMANDE_PWD_FAIT_AU_DESSUS
After=network.target

[Service]
Type=simple
User=root
Group=root

WorkingDirectory=RESULTAT_DE_LA_COMMANDE_PWD_FAIT_AU_DESSUS
ExecStart=RESULTAT_DE_LA_COMMANDE_PWD_FAIT_AU_DESSUS/zoraxy -port=:8000 -fastgeoip=true -sshlb=tru
Restart=on-failure
RestartSec=10

StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=zoraxy

[Install]
WantedBy=multi-user.target

```

Enregistrer le fichier en faisant "CTRL + X" puis "Y" ou "O"

Nous allons maintenant faire un reload du daemon systemD et démarrer zoraxy:
```
1. systemctl daemon-reload
2. systemctl start zoraxy
3. systemctl status zoraxy
```

le resultat de la dernière commande devrais etre :

```
zoraxy.service - Zoraxy reverse-proxy Service
     Loaded: loaded (/etc/systemd/system/zoraxy.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2024-02-04 17:34:46 UTC; 1h 39min ago
   Main PID: 1348 (zoraxy)
      Tasks: 8 (limit: 2309)
     Memory: 678.1M
        CPU: 34.675s
     CGroup: /system.slice/zoraxy.service
             └─1348 /home/xxxx/zoraxy/src/zoraxy -port=:8000 -fastgeoip
```
Nous activons maintenant le service zoraxy au démarrage du linux
```
systemctl enable zoraxy
```

Redemarrer votre machine linux, et verifier après redemarrage que le service Zoraxy est bien demarrer:
```
service zoraxy status
```

Après redémarrage, le service zoraxy devrait être démarrer ! 
:)
