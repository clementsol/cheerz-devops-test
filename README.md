# cheerz-devops-test

URL de l'application déployée : [csol-cheerz-hk6wc3tvqq-od.a.run.app](https://csol-cheerz-hk6wc3tvqq-od.a.run.app/)

## Instructions de déploiement

Push un tag au format `X.Y.Z` (semantic versioning) afin de déclencher le pipeline GitHub Actions qui effectuera les actions suivantes :

- Build de l'application avec Gradle en utilisant le tag GitHub comme version
- Extraction du JAR pour optimiser le layering Docker
- Build de l'image Docker en utilisant le tag GitHub comme tag de l'image
- Push de la nouvelle version de l'image Docker sur un repository Artifact Registry
- Déploiement de la nouvelle version de l'image Docker sur un service Cloud Run

## Build de l'image en local

Build de l'application avec Gradle au sein d'un conteneur

```plaintext
docker run --rm -v "$PWD":/cheerz-devops-test -w /cheerz-devops-test eclipse-temurin:17-jdk ./gradlew build
```

Extraction du JAR pour optimiser le layering Docker

```plaintext
mkdir -p build/dependency && (cd build/dependency; jar -xf ../libs/cheerz-devops-test.jar)
```

Build de l'image Docker

```plaintext
docker build -t cheerz-devops-test:test .
```

## Infrastructure Google Cloud

### Justification du choix de plateforme

Le besoin était donc de déployer sur Google Cloud Platform une simple application Spring Boot conteneurisée et de la rendre publiquement accessible. Mes critères de sélection ont été les suivants : pertinence vis-à-vis du besoin et simplicité de mise en œuvre vis-à-vis de la contrainte de temps.

Mon choix s'est donc naturellement porté vers Cloud Run, qui a précisément été conçu pour répondre à ce type de besoin. En effet, en une simple commande, le service permet de déployer une image et de la rendre accessible.

App Engine aurait également été simple à mettre en œuvre mais m'a semblé moins pertinent étant donné que cela a été conçu pour déployer du code directement. Un service Kubernetes tel qu'un cluster GKE aurait quant à lui été complètement disproportionné par rapport au besoin et bien trop coûteux en temps.

### Procédé de mise en place

Le déploiement de l'infrastructure a été effectué via la CLI gcloud et a impliqué les étapes suivantes :

- Création d'un nouveau projet dédié à l'application
- Activation des API IAM Service Account Credentials, Artifact Registry et Cloud Run Admin
- Création d'une Workload Identity Federation pour gérer l'authentification de GitHub sur GCP via token temporaire
- Création d'un compte de service pour s'authentifier à GCP depuis le pipeline GitHub Actions
- Affectation des permissions nécessaires au compte de service afin qu'il puisse push sur le repository Docker et déployer sur Cloud Run

## Améliorations possibles

- Adopter un workflow Git type Gitflow
- Déporter les secrets sur une solution externe type Google Secret Manager
- Ajouter du linting au workflow GitHub Actions
- Ajouter un scan de vulnérabilités au workflow GitHub Actions en utilisant une solution comme CodeQL ou SonarCloud
- Activer le scan des images Docker via Artifact Analysis suite au push sur Artifact Registry
- Mettre en place de l'alerting sur Cloud Monitoring
- Gérer le déploiement de l'infrastructure GCP en IaC via Terraform sur un nouveau repository
- Déployer un environnement de développement
