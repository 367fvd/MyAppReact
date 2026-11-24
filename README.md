# TP4 : Docker et Docker Compose, application MERN

## Objectif
Containeriser une application MERN (MongoDB, Express, React) avec Docker
et l'orchestrer avec Docker Compose.

## Étapes réalisées
1. Clonage du dépôt : `git clone https://github.com/wiem-kb/MyAppReact.git`
2. Création de `server/Dockerfile` (image node:lts-alpine, port 9000, `npm start`).
3. Création de `client/Dockerfile` (build Vite, serveur `serve` sur le dossier `dist`, port 3000).
4. Création de `docker-compose.yml` avec 3 services (mongodb, server, client)
   et le réseau `mern-network` (driver bridge).
5. Lancement avec `docker compose up --build`.

## Lancer l'application
docker compose up --build

- Client : http://localhost:3000
- API : http://localhost:9000

## Arrêter l'application
docker compose down

## Problème rencontré
Le client affichait une erreur 404. Le projet utilise Vite, qui génère le
dossier `dist` (et non `build`). J'ai corrigé la commande du Dockerfile
en `serve -s dist -l 3000`.

## Captures d'écran
(ajouter ici la capture de `docker ps` et celle du site)
## Test manuel (sans Docker Compose)
docker build -t mern-server ./server
docker build -t mern-client ./client
docker network create mern-network
docker run -d --name mongodb --network mern-network mongo
docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mydb mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client

Les 3 conteneurs sont visibles avec `docker ps`. Docker Compose automatise
ces mêmes étapes avec un seul fichier.
## Test manuel (sans Docker Compose)
docker build -t mern-server ./server
docker build -t mern-client ./client
docker network create mern-network
docker run -d --name mongodb --network mern-network mongo
docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mydb mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client

Docker Compose automatise ces mêmes étapes avec un seul fichier.

## Captures d'écran
![docker ps](captures/docker-ps.png)
![Site](captures/site.png)
