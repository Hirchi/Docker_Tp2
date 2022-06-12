# DEVOPS - TP1

# Objectifs

L’objectif du TP est de créer un un wrapper qui retourne la météo d'un lieu donné avec sa latitude et sa longitude (passées en variable d'environnement) en utilisant Openweather API dans le langage de programmation de votre choix (bash, python, go, nodejs, etc) et packager son code dans une image Docker.

# Les repositories du TP

## GitHub repository

[GitHub - Hirchi/Docker_Tp1](https://github.com/Hirchi/Docker_Tp1)

## DockerHub

[Docker Hub](https://hub.docker.com/repository/docker/abdelhadihirchi/myapp)

# Creation du wrapper

## Openweather API

### ****Current weather data****

C’est une API qui permet à l’utilisateur d’accéder aux données de la météo de plus de 200000 villes. Les données sont disponible sous plusieurs format: JSON, XML, ou HTML.

### API call

Pour appeler l’API on utilise le lien suivant:

```markdown
https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API key}
```

## Le wrapper

On a utilisé le langage Python pour créer mon wrapper. Le request est nécessaire pour faire l’appel de l’API. 

Le os.environ est nécessaire pour récupérer les variable depuis une commande docker -env .

On récupère la donnée de la ville avec la commande request avec un type Json et on l’affiche à l’utilisateur.

On peut ajouter la commande units à l’url de l’api pour choisir l’unité de mesure de la météo.

### Le code:

```python
import requests
import json
from datetime import datetime as dt
import os
api_key = "2ceb1995c8cc45dbdfd4e287128c7057"
lat = os.environ['LAT']
lon = os.environ['LONG']
url = "https://api.openweathermap.org/data/2.5/weather?lat=%s&lon=%s&appid=%s&units=metric" % (lat, lon, api_key)
response = requests.get(url)
data = json.loads(response.text)
print(data)
```

## Docker

pour faire marcher notre wrapper on a besoin d’un environnement Python, c’est pour ça on utilise docker pour créer cet environnement. 

 

### Le Dockerfile

```docker
FROM python:3.8-alpine
RUN mkdir /app
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

### Le fichier requirements.txt

```markdown
requests
datetime
```

# Commande pour faire marcher le projet

## Container docker

### Build

```powershell
docker build . -f .\docker.dockerfile -t  myapp
```

### Execute

```powershell
docker run --env LAT="5.902785" --env LONG="102.754175" --env API_KEY=2ceb1995c8cc45dbdfd4e287128c7057 myapp
```

### Résultat

```powershell
{'coord': {'lon': 102.7542, 'lat': 5.9028}, 'weather': [{'id': 803, 'main': 'Clouds', 'description': 'broken clouds', 'icon': '04d'}], 'base': 'stations', 'main': {'temp': 28.29, 'feels_like': 30.67, 'temp_min': 28.29, 'temp_max': 28.29, 'pressure': 1006, 'humidity': 66, 'sea_level': 1006, 'grnd_level': 980}, 'visibility': 10000, 'wind': {'speed': 2.86, 'deg': 85, 'gust': 2.7}, 'clouds': {'all': 59}, 'dt': 1654423617, 'sys': {'country': 'MY', 'sunrise': 1654383247, 'sunset': 1654428060}, 'timezone': 28800, 'id': 1736405, 'name': 'Jertih', 'cod': 200}
```

### Taguer l’image

```powershell
docker tag myapp abdelhadihirchi/myapp
```

### Publier l’image sur mon depository DockerHub

```powershell
docker push abdelhadihirchi/myapp
```

# Bonus

## recherche de vulnérabilité avec trivy

```powershell
abdel@LAPTOP-VKS08S91:~$ trivy image abdelhadihirchi/myapp
2022-06-05T12:18:31.684+0200    INFO    Need to update DB
2022-06-05T12:18:31.684+0200    INFO    Downloading DB...
28.12 MiB / 28.12 MiB [------------------------------------------------------------------------] 100.00% 5.24 MiB p/s 5s
2022-06-05T12:18:44.658+0200    INFO    Detected OS: alpine
2022-06-05T12:18:44.658+0200    WARN    This OS version is not on the EOL list: alpine 3.16
2022-06-05T12:18:44.658+0200    INFO    Detecting Alpine vulnerabilities...
2022-06-05T12:18:44.664+0200    INFO    Number of PL dependency files: 0
2022-06-05T12:18:44.664+0200    WARN    This OS version is no longer supported by the distribution: alpine 3.16.0
2022-06-05T12:18:44.665+0200    WARN    The vulnerability detection may be insufficient because security updates are not provided

abdelhadihirchi/myapp (alpine 3.16.0)
=====================================
Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
```

## handolint

```powershell
abdel@LAPTOP-VKS08S91:/mnt/c/Users/abdelhadi/Desktop/docker$ docker run --rm -i hadolint/hadolint < docker.dockerfile
-:5 DL3042 warning: Avoid use of cache directory with pip. Use `pip install --no-cache-dir <package>`
```

## Données sensible

on change le api_key par celui de l’environnement et avec cette méthode on aura pas notre API_KEY disponible pour tout le monde

```powershell
import requests
import json
from datetime import datetime as dt
import os
api_key = os.environ['API_KEY']
lat = os.environ['LAT']
lon = os.environ['LONG']
url = "https://api.openweathermap.org/data/2.5/weather?lat=%s&lon=%s&appid=%s&units=metric" % (lat, lon, api_key)
response = requests.get(url)
data = json.loads(response.text)
print(data)
```