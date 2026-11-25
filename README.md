# TP4 : Docker et Docker Compose, application MERN

**Auteur :** Amir Mijaoui
**Cours :** Introduction au DevOps, ISG de Sousse (3ème année LIG)

## Liens

- GitHub : https://github.com/367fvd/MyAppReact
- Docker Hub (serveur) : https://hub.docker.com/r/amirmijaoui/mern-server
- Docker Hub (client) : https://hub.docker.com/r/amirmijaoui/mern-client

## Objectif

Containeriser une application MERN (MongoDB, Express, React, Node.js) avec Docker,
l'orchestrer avec Docker Compose, puis automatiser la construction et le déploiement
avec un pipeline Jenkins.

## Structure du projet

```
MyAppReact/
├── client/               # application React (Vite)
│   └── Dockerfile
├── server/               # API Express
│   └── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── screenshots/
└── README.md
```

## 1. Dockerfiles

### Serveur (`server/Dockerfile`)

- Image de base `node:lts-alpine`
- Répertoire de travail `/app`
- Copie de `package*.json`, puis `npm install`
- Copie du reste des fichiers
- Port 9000 exposé, démarrage avec `npm start`
- La variable `MONGO_URI` est fournie à l'exécution (Docker Compose ou `-e`)

### Client (`client/Dockerfile`)

- Image de base `node:lts-alpine`
- Copie de `package*.json`, puis `npm install`
- Copie du reste des fichiers, puis `npm run build`
- Installation du serveur HTTP léger `serve`
- Port 3000 exposé, démarrage avec `serve -s dist -l 3000`

**Problème rencontré :** le client affichait une erreur 404. Le projet utilise Vite,
qui génère le dossier `dist` (et non `build`). La commande a été corrigée en
`serve -s dist -l 3000`.

## 2. Test manuel (sans Docker Compose)

```bash
docker build -t mern-server ./server
docker build -t mern-client ./client
docker network create mern-network
docker run -d --name mongodb --network mern-network mongo
docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mydb mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```

Les trois conteneurs sont vérifiés avec `docker ps`.

## 3. Docker Compose

Le fichier `docker-compose.yml` définit trois services (`mongodb`, `server`, `client`)
connectés au réseau `mern-network` (driver bridge). Le serveur dépend de `mongodb`
et le client dépend du serveur.

Lancer l'application :

```bash
docker compose up --build
```

- Client : http://localhost:3000
- API : http://localhost:9000

Arrêter l'application :

```bash
docker compose down
```

## 4. Publication sur Docker Hub

```bash
docker login
docker tag mern-server amirmijaoui/mern-server:latest
docker tag mern-client amirmijaoui/mern-client:latest
docker push amirmijaoui/mern-server:latest
docker push amirmijaoui/mern-client:latest
```

## 5. Pipeline Jenkins

Le `Jenkinsfile` (agent Windows, commandes `bat`) exécute 4 étapes :

1. **Build serveur** : construit l'image `amirmijaoui/mern-server`
2. **Build client** : construit l'image `amirmijaoui/mern-client`
3. **Push Docker Hub** : connexion avec le credential Jenkins `dockerhub`, puis envoi des deux images
4. **Lancer les conteneurs** : crée le réseau et démarre `mongodb`, `server` et `client`

**Problèmes rencontrés et solutions :**

- `sh` introuvable : Jenkins tourne sous Windows, `sh` remplacé par `bat`.
- `docker` non reconnu : ajout du dossier de Docker Desktop au `PATH` dans le Jenkinsfile.
- Credential `dockerhub` introuvable : il était créé dans le store de l'utilisateur, recréé dans le store System (global).
- Erreur de connexion Docker Hub : jeton d'accès personnel (Lecture et écriture) régénéré.

## 6. Captures d'écran

### Conteneurs en cours d'exécution
![docker ps](screenshots/docker-ps.png)

### Images Docker
![docker images](screenshots/docker-images.png)

### Application dans le navigateur
![Site](screenshots/site.png)

### Docker Compose
![Docker Compose](screenshots/compose-up.png)

### Images sur Docker Hub
![Docker Hub serveur](screenshots/dockerhub.png)
![Docker Hub client](screenshots/dockerhub-client.png)

### Pipeline Jenkins
![Pipeline Jenkins](screenshots/jenkins-pipeline.png)
![Console Jenkins](screenshots/jenkins-console.png)