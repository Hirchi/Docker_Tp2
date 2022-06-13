# DEVOPS-TP2

## Objectifs

l’objectif est de créé une api à partir du wrapper du premier TP, lancer l’api sur docker et automatiser la publication ç chaque push sur DockerHub.

 

# Les repositories du TP

## GitHub repository

[GitHub - Hirchi/Docker_Tp2](https://github.com/Hirchi/Docker_Tp2)

## DockerHub

[Docker Hub](https://hub.docker.com/repository/docker/abdelhadihirchi/efrei-devops-tp2)

# Création de l’Api

### L’api en Python

l’api est baser sur le wrapper, on récupère la latitude et longitude avec des request flask pour permettre de les ajouter dans l’url. On lance l’api sur le [localhost](http://localhost) port:8081 

```python
import requests
import json
from flask import Flask,  request, jsonify
import os
app = Flask(__name__)
api_key = os.environ['API_KEY']

with app.app_context():
    @app.route("/")
    def meteo():
        lat = request.args.get('lat')
        lon = request.args.get('lon')
        url = "https://api.openweathermap.org/data/2.5/weather?lat=%s&lon=%s&appid=%s&units=metric" % (
            lat, lon, api_key)
        response = requests.get(url)
        data = json.loads(response.text)
        if response.status_code != 200:
            return jsonify({
                'status': 'error',
                'message': 'La requête à l\'API météo n\'a pas fonctionné. Voici le message renvoyé par l\'API : {}'.format(data)
            }), 500

        return jsonify({
            'status': 'ok',
            'data': data
        })

if __name__ == "__main__":
    app.run(port=8081, debug=True)
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

C’est le ficher dans on trouve les librairie demander pour faire fonctionner notre code python 

```markdown
requests
datetime
flask
```

## Automatisation du push sur DockerHub

### Le workflow

GitHub actions permet d’automatiser le push de l’image sur dockerhub.

```yaml
name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      -
        name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Build and push
        uses: docker/build-push-action@v3
        with:
          push: true
          tags: abdelhadihirchi/efrei-devops-tp2
```

### Secrets

il faut configurer nos identifiants Dockerhub sur Github secrets.

```yaml
username: ${{ secrets.DOCKERHUB_USERNAME }}
password: ${{ secrets.DOCKERHUB_TOKEN }}
```

## Bonus

### handolint

```yaml
name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      -
        name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: hadolint
        uses: reviewdog/action-hadolint@v1
        with:
          reporter: github-pr-review
      -
        name: Build and push
        uses: docker/build-push-action@v3
        with:
          push: true
          tags: abdelhadihirchi/efrei-devops-tp2
```

### Sécurité

Toute les données sensibles sont hors image ou code .