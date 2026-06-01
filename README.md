# Dev_Ops
Docker TD/TP 4A S8 

1-1 — Pourquoi utiliser -e plutôt que mettre les variables dans le Dockerfile ?
Car le Dockerfile est souvent publié sur Git. Si le mot de passe est dedans, tout le monde peut le voir. Avec -e on passe les secrets uniquement au moment du lancement, ils ne sont jamais écrits dans un fichier.

1-2 — Pourquoi avoir un volume attaché au conteneur PostgreSQL ?
Sans volume, si on supprime le conteneur toutes les données sont perdues. Le volume stocke les données sur le disque de la machine hôte, elles survivent donc même si le conteneur est supprimé.

1-3 — Les commandes et Dockerfile essentiels :
FROM postgres:17.2-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY sql/ /docker-entrypoint-initdb.d/

docker network create app-network
docker build -t my-database .
docker run -d --name my-postgres --network app-network -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -v /my/own/datadir:/var/lib/postgresql/data my-database



1-4 Pourquoi le multistage build ?
Sans multistage, l'image finale contiendrait le JDK + Maven + tout le code source → image très lourde. Avec le multistage, l'image finale contient uniquement le JRE et le jar, c'est beaucoup plus léger.
