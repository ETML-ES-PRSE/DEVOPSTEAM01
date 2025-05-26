
## Environnement
### Installation of Nginx
```
sudo apt update
sudo apt install nginx -y
```

### Créer un utilisateur avec permissions restraints pour le service Nginx
```
sudo useradd --system --no-create-home --shell /usr/sbin/nologin rpuser

sudo chown -R rpuser:rpuser /var/log/nginx
sudo chown -R rpuser:rpuser /etc/nginx/sites-available /etc/nginx/sites-enabled
```

### Configurer le service systemd Nginx
```
sudo mkdir -p /etc/systemd/system/nginx.service.d

nano /etc/systemd/system/nginx.service.d/override.conf
```
Contenu fichier, CTRL + S, CTRL + X
```bash
[Service]
Restart=always
RestartSec=5s
```



## Configuration

### Log
```
nano /etc/nginx/nginx.conf
```
Ajouter dans bloc http
```
log_format custom '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent"';
```

Changer user
### Virtual Host

```
nano /etc/nginx/sites-available/reverse-proxy.conf
```
Contenu fichier :
Remplacer [IP MACHINE APACHE]
```
server {
    listen 8080;
    server_name grp1.prse.local;

    access_log /var/log/nginx/rp_access.log custom;
    error_log  /var/log/nginx/rp_error.log warn;

    location /wordpress/ {
        proxy_pass http://[IP MACHINE APACHE]:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location = / {
        return 302 /wordpress/;
    }
}
```


Activer le site et tester
```
sudo ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/
sudo nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

sudo systemctl daemon-reload
sudo systemctl reload nginx
```

### Hosts pour simulation DNS
```
nano /etc/hosts
```

Ajouter cette ligne
```
127.0.0.1   grp1.prse.local
```

## Tests

### Service Nginx
```
sudo systemctl status nginx
```
### Requete
```
curl -I http://grp1.prse.local:8080/wordpress/
```
### Logs
```
tail -f /var/log/nginx/rp_access.log /var/log/nginx/rp_error.log
```
