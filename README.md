# DevOps — Compte-Rendu Docker & CI/CD (4A S8)

Ce dépôt contient les réponses, concepts clés et configurations essentielles pour les TP1,TP2 et TP3 du module DevOps.

---

## 🐳 TP1 — Fondations Docker & Multi-containers

### 1-1 — Gestion des variables d'environnement (`-e` vs Dockerfile)
* **Le problème :** Le fichier `Dockerfile` est fréquemment poussé sur des dépôts distants publics ou partagés (Git). Y inscrire des mots de passe ou des clés d'API expose ces secrets à toute personne ayant accès au code.
* **La solution :** L'option `-e` (ou `--env`) permet d'injecter les variables d'environnement **uniquement au moment du runtime** (lancement du conteneur via `docker run`). Les secrets restent ainsi en mémoire volatile et ne sont jamais écrits dans le code source ou figés dans l'image.

### 1-2 — Persistance des données (Volume PostgreSQL)
Par défaut, le système de fichiers d'un conteneur est éphémère. Si le conteneur PostgreSQL est arrêté et supprimé (`docker rm`), toutes les données et bases créées sont définitivement perdues.
* **Le volume** permet de lier (mapper) un dossier de la machine hôte au dossier de stockage interne de PostgreSQL (`/var/lib/postgresql/data`).
* **Résultat :** Les données survivent aux arrêts, suppressions et mises à jour des conteneurs.

### 1-3 — Fichiers et commandes essentiels (PostgreSQL)

#### Mon Dockerfile
```dockerfile
FROM postgres:17.2-alpine

# Configuration des variables par défaut
ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

# Copie des scripts d'initialisation SQL
COPY sql/ /docker-entrypoint-initdb.d/

# 1. Création du réseau isolé pour l'application
docker network create app-network

# 2. Build de l'image personnalisée
docker build -t my-database .

# 3. Lancement du conteneur avec volume et réseau configurés
docker run -d \
  --name my-postgres \
  --network app-network \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=usr \
  -e POSTGRES_PASSWORD=pwd \
  -v /my/own/datadir:/var/lib/postgresql/data \
  my-database

```

### 1-4 — Pourquoi utiliser le Multistage Build ?
Sans multistage build, l'image finale embarque tout l'environnement de développement (JDK, Maven, code source, caches de dépendances), ce qui produit des images extrêmement lourdes (parfois supérieures à 800 Mo) et augmente la surface d'attaque (failles de sécurité).

* **Avec le Multistage :** On utilise une première image pour compiler le projet (`builder`), puis on copie uniquement l'artefact final compilé (le fichier `.jar`) dans une seconde image de runtime beaucoup plus légère contenant un simple JRE. L'image finale est ainsi optimisée et sécurisée.

### 1-5 — Le rôle du Reverse Proxy (Apache / HTTPD)
Le reverse proxy se place en intermédiaire entre les clients (navigateurs) et nos services applicatifs en arrière-plan :

* **Sécurité :** Il masque l'architecture interne. Le client interagit uniquement avec le port standard `80` (ou `443`), masquant ainsi le port réel du backend (ex: `8080`).
* **Fonctionnalités :** Il centralise la gestion du chiffrement SSL/TLS (HTTPS), permet de faire du *Load Balancing* (répartition de charge entre plusieurs conteneurs) et distribue efficacement les fichiers statiques du frontend.

---

## 🐙 Orchestration avec Docker Compose

### 1-6 — Intérêt de Docker Compose
Au lieu de devoir configurer et lancer chaque conteneur, réseau et volume manuellement un par un avec de longues lignes de commande `docker run`, Docker Compose permet de **déclarer** l'ensemble de l'architecture multi-conteneurs dans un unique fichier structuré (`docker-compose.yml`). Une seule commande suffit pour tout orchestrer automatiquement.

### 1-7 — Commandes Docker Compose incontournables
* `docker compose up -d` : Démarre tous les services définis en arrière-plan.
* `docker compose down` : Arrête et détruit les conteneurs, réseaux et volumes associés.
* `docker compose build` : Force la reconstruction des images personnalisées du projet.
* `docker compose logs -f` : Affiche et suit les lignes de log de tous les conteneurs en temps réel.
* `docker compose ps` : Liste et affiche l'état/statut des conteneurs (en cours d'exécution ou arrêtés).

### 1-8 — Commentaires de configuration
> 💡 **Note :** Se référer directement aux commentaires détaillés présents dans notre fichier de configuration `docker-compose.yml` pour comprendre les liaisons entre services.

---

## 🚀 Publication sur le Docker Hub

### 1-9 — Commandes de Tag et Push (v1.0)
```bash
# 1. Connexion au registre Docker Hub distant
docker login

# 2. Tag des images locales vers le format requis par le Docker Hub
docker tag flask-app-backend victor7934/flask-app-backend:1.0
docker tag flask-app-database victor7934/flask-app-database:1.0
docker tag flask-app-httpd victor7934/flask-app-httpd:1.0

# 3. Publication des versions 1.0 sur le registre en ligne
docker push victor7934/flask-app-backend:1.0
docker push victor7934/flask-app-database:1.0
docker push victor7934/flask-app-httpd:1.0

```

### 1-10 — Pourquoi utiliser un registre en ligne ?
* **Partage & Déploiement :** Permet de télécharger et d'exécuter l'application sur n'importe quelle machine ou serveur cloud sans avoir besoin de re-compiler le code source localement.
* **Reproductibilité :** Garantit que l'environnement et l'image exécutés en production sont strictement identiques à ceux validés en développement.
* **Automatisation (CI/CD) :** Facilite l'intégration et le déploiement continus. Les pipelines peuvent récupérer (`pull`) automatiquement ces images de confiance.
* **Gestion des versions :** Permet de suivre, archiver et historiser le code grâce aux tags (`1.0`, `2.0`, `latest`), rendant les retours en arrière (*rollbacks*) instantanés en cas de bug.

---

## 🛠️ TP2 — Tests & Pipelines de CI/CD

### 2-1 — Qu'est-ce que "Testcontainers" ?
**Testcontainers** est une bibliothèque (disponible pour Java, Python, .NET, etc.) permettant de lever de vrais conteneurs Docker légers et éphémères de manière programmable pendant l'exécution des tests d'intégration.

* **Intérêt :** Au lieu d'utiliser des bases de données simplifiées en mémoire (comme H2) qui n'ont pas le même comportement qu'en production, Testcontainers démarre une instance réelle de PostgreSQL (ou de Redis, RabbitMQ, etc.) au début du test et la détruit proprement à la fin. Cela garantit des tests fiables, jetables et au plus proche de la réalité.

### 2-2 — Pourquoi utiliser des variables sécurisées (Secrets) ?
Dans un pipeline de CI/CD, l'application a besoin d'accéder à des ressources sensibles (mots de passe de base de données, tokens d'API, identifiants Docker Hub ou clés privées SSH).

* **Le rôle des Secrets :** Ils servent à stocker ces données de manière chiffrée dans la plateforme de CI/CD (ex: *GitHub Actions Secrets*).
* **Sécurité :** Le pipeline peut appeler ces variables pour s'authentifier, mais l'outil **masque automatiquement leur valeur** dans les logs de console (remplacées par `***`) et évite qu'elles n'apparaissent en clair dans le code source du dépôt, éliminant ainsi tout risque de fuite ou de piratage.

### 2.3 — Pourquoi avons-nous ajouté la tâche « build-and-test-backend » à ce travail ?

La tâche `build-and-test-backend` est ajoutée au workflow CI pour s'assurer que le code compile correctement et que tous les tests passent avant de continuer. Si on la supprime, des images Docker cassées pourraient être publiées sur DockerHub et déployées en production sans qu'on s'en rende compte. Elle sert de filet de sécurité : pas de déploiement si les tests échouent.

---

### 2.4 — Dans quel but devons-nous publier des images Docker ?

Publier des images Docker sur DockerHub permet de :
- **Partager** les images entre machines et environnements (local, CI, serveur de production)
- **Versionner** les builds de l'application
- **Déployer** facilement sur n'importe quel serveur sans recompiler le code
- **Centraliser** les artefacts de build pour toute l'équipe

Sans publication, chaque machine devrait builder l'image localement, ce qui est lent, peu fiable et non reproductible.

---

## TP3 — Discover Ansible

### 3 — Est-il vraiment sûr de déployer automatiquement chaque nouvelle image sur le hub ? Expliquez pourquoi. Que puis-je faire pour renforcer la sécurité ?

Non, ce n'est pas totalement sûr. Déployer automatiquement chaque nouvelle image présente plusieurs risques :
- Une image buguée ou malveillante peut être déployée directement en production sans validation humaine
- Si le compte DockerHub est compromis, un attaquant peut publier une image malveillante qui sera déployée automatiquement
- L'utilisation du tag `latest` ne permet pas de tracer quelle version exacte est déployée

**Pour sécuriser le déploiement :**
- Utiliser des **tags versionnés** (`v1.0.0`) plutôt que `latest` pour tracer précisément ce qui est déployé
- Ajouter des **tests automatiques** (unitaires, d'intégration) avant chaque déploiement
- Mettre en place une **validation manuelle** avec `environment: production` dans GitHub Actions pour les déploiements en production
- **Scanner les images** avec des outils comme Trivy ou Snyk pour détecter les vulnérabilités avant le déploiement

