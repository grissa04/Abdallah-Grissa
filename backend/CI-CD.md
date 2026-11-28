# Atelier DevOps – Automatisation du processus CI/CD avec Jenkins

**Étudiant :** Abdallah Grissa  
**Classe :** 4SSA15  
**Projet :** Application de gestion des projets

## 1. Objectif de l'atelier

L'objectif de cet atelier est d'automatiser le processus d'intégration continue (CI) et de déploiement continu (CD) d'une application Spring Boot à l'aide de Jenkins, Maven, Docker et Docker Hub.

## 2. Technologies utilisées

- Git et GitHub : gestion du code source.
- Jenkins : automatisation du pipeline CI/CD.
- Maven : compilation, tests et génération du fichier JAR.
- Docker : conteneurisation de l'application.
- Docker Hub : stockage et publication des images Docker.
- Spring Boot : développement du backend.
- MySQL : gestion de la base de données.

## 3. Les différentes étapes du pipeline CI/CD

| Étape | Description |
|---|---|
| Checkout | Récupération du code source depuis GitHub. |
| Compile | Compilation du projet Spring Boot avec Maven. |
| Build & Test Maven | Exécution des tests et génération du fichier JAR. |
| Docker Build | Construction de l'image Docker du backend à partir du Dockerfile. |
| Docker Push | Authentification avec les Credentials Jenkins et publication de l'image sur Docker Hub. |
| Deploy MySQL | Démarrage du conteneur MySQL et vérification de sa disponibilité. |
| Deploy Backend | Arrêt et suppression de l'ancien conteneur, récupération de l'image publiée et lancement du nouveau conteneur. |
| Verification | Vérification des conteneurs actifs et des logs de démarrage du backend. |

## 4. Dockerfile

Le Dockerfile permet de créer une image Docker contenant l'application Spring Boot et son environnement d'exécution Java 17.

```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## 5. Commandes Docker utilisées

### Construction de l'image

```bash
docker build -t backend-app:latest .
```

### Attribution du tag

```bash
docker tag backend-app:latest grissa04/backend-app:latest
```

### Authentification et publication

```bash
docker login
docker push grissa04/backend-app:latest
```

Dans le pipeline Jenkins, l'authentification est automatisée à l'aide des Credentials Jenkins et de l'option `--password-stdin`.

### Création du réseau Docker

```bash
docker network create backend-network
```

Ce réseau permet la communication entre les conteneurs MySQL et Spring Boot.

### Déploiement du conteneur MySQL

```bash
docker run -d --name mysql \
    --network backend-network \
    -p 127.0.0.1:3306:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -e MYSQL_DATABASE=test_db \
    grissa04/mysql:v1
```

Si le conteneur existe déjà, le pipeline le redémarre et le connecte au réseau Docker si nécessaire.

### Arrêt et suppression de l'ancien backend

```bash
docker stop backend-app
docker rm backend-app
```

### Récupération de l'image publiée

```bash
docker pull grissa04/backend-app:latest
```

### Déploiement du backend

```bash
docker run -d --name backend-app \
    --network backend-network \
    -p 8081:8080 \
    -e SPRING_DATASOURCE_URL='jdbc:mysql://mysql:3306/test_db?createDatabaseIfNotExist=true' \
    -e SPRING_DATASOURCE_USERNAME=root \
    -e SPRING_DATASOURCE_PASSWORD=root \
    grissa04/backend-app:latest
```

### Vérification du déploiement

```bash
docker ps
docker logs backend-app
```

## 6. Configuration du déploiement

| Paramètre | Valeur |
|---|---|
| Réseau Docker | backend-network |
| Conteneur MySQL | mysql |
| Conteneur Spring Boot | backend-app |
| Port MySQL | 3306 |
| Port interne du backend | 8080 |
| Port externe du backend | 8081 |
| Base de données | test_db |
| Image MySQL | grissa04/mysql:v1 |
| Image backend | grissa04/backend-app:latest |

Le port externe 8081 est utilisé pour éviter un conflit avec Jenkins, qui utilise le port 8080.

## 7. Résultats obtenus

Le pipeline Jenkins numéro 7 s'est terminé avec le statut **SUCCESS**, avec une durée totale de 6 minutes et 39 secondes.

Les résultats obtenus sont les suivants :

- Compilation Maven réussie.
- Tests Maven réussis.
- Construction de l'image Docker réussie.
- Publication de l'image sur Docker Hub réussie.
- Déploiement du conteneur MySQL réussi.
- Déploiement du conteneur Spring Boot réussi.
- Connexion entre Spring Boot et MySQL vérifiée.
- Démarrage du serveur Tomcat confirmé.

## 8. Prérequis et remarques

- MySQL doit être disponible avant l'exécution des tests Maven.
- Les identifiants Docker Hub sont gérés par les Credentials Jenkins.
- Les identifiants MySQL utilisés correspondent à un environnement de travaux pratiques et non à un environnement de production.
- Le dépôt Docker Hub est actuellement public. Une visibilité privée est nécessaire pour respecter entièrement les consignes de l'atelier.
- Le pipeline réalisé concerne le backend Spring Boot.

## 9. Liens utiles

- Dépôt GitHub : https://github.com/grissa04/Abdallah-Grissa
- Image Docker du backend : https://hub.docker.com/r/grissa04/backend-app
- Image Docker MySQL : https://hub.docker.com/r/grissa04/mysql

## 10. Conclusion

Cet atelier a permis de mettre en place un pipeline CI/CD automatisé avec Jenkins, Maven et Docker. Le processus couvre la récupération du code source, la compilation, les tests, la construction et la publication de l'image Docker ainsi que le déploiement et la vérification de l'application Spring Boot avec MySQL.
