# Part I : Docker basics

## 1. Install

🌞 **Installer Docker votre machine Azure**

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
```
sudo systemctl start docker
```
```
sudo usermod -aG docker $(whoami)
```

## 3. Lancement de conteneurs

🌞 **Utiliser la commande `docker run`**

```
azureuser@machine2:~$ docker run --name web -d -p 9999:80 nginx
```

🌞 **Rendre le service dispo sur internet**
```
PS C:\Users\menan> curl http://172.160.250.172:9999


StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html>
                    <html>
                    <head>
                    <title>Welcome to nginx!</title>
                    <style>
                    html { color-scheme: light dark; }
                    body { width: 35em; margin: 0 auto;
                    font-family: Tahoma, Verdana, Arial, sans-serif; }
                    </style...
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Accept-Ranges: bytes
                    Content-Length: 615
                    Content-Type: text/html
                    Date: Fri, 14 Mar 2025 11:02:27 GMT
                    ETag: "67a34638-267"
                    Last-Modified: Wed, 05 Feb 2025 ...
Forms             : {}
Headers           : {[Connection, keep-alive], [Accept-Ranges, bytes],
                    [Content-Length, 615], [Content-Type, text/html]...}
Images            : {}
InputFields       : {}
Links             : {@{innerHTML=nginx.org; innerText=nginx.org;
                    outerHTML=<A href="http://nginx.org/">nginx.org</A>;
                    outerText=nginx.org; tagName=A;
                    href=http://nginx.org/}, @{innerHTML=nginx.com;
                    innerText=nginx.com; outerHTML=<A
                    href="http://nginx.com/">nginx.com</A>;
                    outerText=nginx.com; tagName=A;
                    href=http://nginx.com/}}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 615

```

🌞 **Custom un peu le lancement du conteneur**

**index.html** ⬇️
```
EZ WIN
```

**conf.conf** ⬇️
```
server {
  # on définit le port où NGINX écoute dans le conteneur
  listen 7777;
  
  # on définit le chemin vers la racine web
  # dans ce dossier doit se trouver un fichier index.html
  root /var/www/tp_docker; 
}
```
**Commande pour lancer le docker** ⬇️
```
 docker run --name meow -d -v /home/azureuser/index.html:/var/www/tp_docker/index.html -v /home/azureuser/conf.conf:/etc/nginx/conf.d/conf.conf -p 7777:7777 nginx
```

🌞 **Call me**

✅

# Part II : Images

🌞 **Construire votre propre image**

**apache2.conf** ⬇️

```
# on définit un port sur lequel écouter
Listen 80

# on charge certains modules Apache strictement nécessaires à son bon fonctionnement
LoadModule mpm_event_module "/usr/lib/apache2/modules/mod_mpm_event.so"
LoadModule dir_module "/usr/lib/apache2/modules/mod_dir.so"
LoadModule authz_core_module "/usr/lib/apache2/modules/mod_authz_core.so"

# on indique le nom du fichier HTML à charger par défaut
DirectoryIndex index.html
# on indique le chemin où se trouve notre site
DocumentRoot "/var/www/html/"

# quelques paramètres pour les logs
ErrorLog "logs/error.log"
LogLevel warn
```
**index.html** ⬇️
```
EZ WIN
```
**Dockerfile** ⬇️
```
FROM ubuntu

RUN apt update -y

RUN apt install -y apache2

RUN mkdir /etc/apache2/logs

COPY index.html /var/www/html/index.html

COPY apache2.conf /etc/apache2/apache2.conf

CMD [ "apache2", "-DFOREGROUND" ]
```

**Commande pour lancer le docker** ⬇️

```
docker build . -t my_own_nginx
```

```
docker run -p 8888:80 my_own_nginx
```




