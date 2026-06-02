# Dev_Ops
## Docker TD/TP 4A S8 

### 1-1 — Pourquoi utiliser -e plutôt que mettre les variables dans le Dockerfile ?

Car le Dockerfile est souvent publié sur Git. Si le mot de passe est dedans, tout le monde peut le voir. Avec -e on passe les secrets uniquement au moment du lancement, ils ne sont jamais écrits dans un fichier.

### 1-2 — Pourquoi avoir un volume attaché au conteneur PostgreSQL ?
Sans volume, si on supprime le conteneur toutes les données sont perdues. Le volume stocke les données sur le disque de la machine hôte, elles survivent donc même si le conteneur est supprimé.

### 1-3 — Les commandes et Dockerfile essentiels :
FROM postgres:17.2-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY sql/ /docker-entrypoint-initdb.d/

docker network create app-network
docker build -t my-database .
docker run -d --name my-postgres --network app-network -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -v /my/own/datadir:/var/lib/postgresql/data my-database



### 1-4 Pourquoi le multistage build ?
Sans multistage, l'image finale contiendrait le JDK + Maven + tout le code source → image très lourde. Avec le multistage, l'image finale contient uniquement le JRE et le jar, c'est beaucoup plus léger.


### 1-5 : Le reverse proxy
Le reverse proxy (Apache) sert d'intermédiaire entre le client et le backend.

Sécurité : Masque l'architecture interne (le client voit le port 80, pas le port 8080 du backend).

Fonctionnalités : Gère le SSL (HTTPS), le load balancing et distribue le frontend statique.

### 1-6 : Intérêt de Docker Compose
Il permet d'orchestrer une application multi-containers.

Sans lui : Il faut configurer et lancer chaque container manuellement un par un.

Avec lui : Un seul fichier (docker-compose.yml) et une seule commande (docker compose up) lancent tout automatiquement.

### 1-7 : Commandes essentielles

docker compose up -d : Démarre les services en arrière-plan.

docker compose down : Arrête et supprime les containers.

docker compose build : Reconstruit les images.

docker compose logs : Affiche les lignes de log des containers.

docker compose ps : Statut des containers (en cours d'exécution ou arrêtés).


### 1-8 : Commentaires
Voir les commentaires directement dans le fichier de configuration.


### 1-9 : Publication Docker Hub
Commandes utilisées pour taguer et publier les versions 1.0 :


docker login

Tag

docker tag flask-app-backend victor7934/flask-app-backend:1.0
docker tag flask-app-database victor7934/flask-app-database:1.0
docker tag flask-app-httpd victor7934/flask-app-httpd:1.0

docker push victor7934/flask-app-backend:1.0
docker push victor7934/flask-app-database:1.0
docker push victor7934/flask-app-httpd:1.0


### 1-10 : Pourquoi un registre en ligne ?
Partage & Déploiement : Télécharger l'application sur n'importe quelle machine sans la reconstruire.

Reproductibilité : Garantir que l'environnement est identique partout (Dev, Test, Prod).

Automatisation : Faciliter l'intégration et le déploiement continu (CI/CD).

Gestion des versions : Suivre et archiver l'historique du code grâce aux tags (1.0, latest).

# TP2

### 2-1 What are testcontainers?