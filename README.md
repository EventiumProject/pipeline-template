# Pipeline Template Eventium

Ce dépôt contient des workflows GitHub Actions réutilisables pour automatiser le build, le push et le déploiement des applications Eventium.

## 📋 Workflows disponibles

### 1. Docker Build and Push

Construit une image Docker et la pousse sur Docker Hub.

**Fichier** : `.github/workflows/docker-build-and-push.yml`

#### Inputs

| Paramètre | Description | Type | Requis | Défaut |
|-----------|-------------|------|--------|--------|
| `image-name` | Nom de l'image Docker (ex: `eventium/eventium-portail-backend`) | `string` | ✅ Oui | - |
| `docker-context` | Contexte de build Docker | `string` | ❌ Non | `.` |
| `dockerfile-path` | Chemin vers le Dockerfile | `string` | ❌ Non | `./Dockerfile` |
| `docker-push` | Pousser l'image sur le registry | `boolean` | ❌ Non | `true` |

#### Secrets requis

- `DOCKER_HUB_USERNAME` : Nom d'utilisateur Docker Hub
- `DOCKER_HUB_PASSWORD` : Mot de passe ou token Docker Hub

#### Exemple d'utilisation

```yaml
jobs:
  docker-build-and-push:
    uses: EventiumProject/pipeline-template/.github/workflows/docker-build-and-push.yml@main
    with:
      image-name: eventium/eventium-portail-backend
      docker-context: .
      dockerfile-path: ./Dockerfile
      docker-push: true
    secrets: inherit
```

---

### 2. Deploy

Déploie l'application sur un serveur distant via SSH et Docker Compose.

**Fichier** : `.github/workflows/deploy.yml`

#### Inputs

| Paramètre | Description | Type | Requis | Défaut |
|-----------|-------------|------|--------|--------|
| `docker-image-name` | Nom de la variable d'environnement pour la version Docker | `string` | ✅ Oui | - |

#### Secrets requis

**Pour la production (branche `main`)** :
- `ENV_FILE` : Contenu du fichier .env
- `DEPLOY_PATH` : Chemin de déploiement sur le serveur

**Pour la pré-production (branche `develop` ou autres)** :
- `ENV_FILE_PREPROD` : Contenu du fichier .env
- `DEPLOY_PATH_PREPROD` : Chemin de déploiement sur le serveur

**Communs** :
- `SSH_PRIVATE_KEY` : Clé SSH privée pour se connecter au serveur
- `DEPLOY_HOST` : Adresse du serveur de déploiement
- `DEPLOY_USER` : Utilisateur SSH
- `DISCORD_WEBHOOK` : Webhook Discord pour les notifications (optionnel)

#### Exemple d'utilisation

```yaml
jobs:
  deploy:
    uses: EventiumProject/pipeline-template/.github/workflows/deploy.yml@main
    needs: docker-build-and-push
    with:
      docker-image-name: EVENTIUM_API_DOCKER_VERSION
    secrets: inherit
```

---

## 🚀 Exemple complet

Voici un exemple de workflow complet qui build et déploie une application :

```yaml
name: Pipeline Deploy CICD

on:
  push:
    branches:
      - main
      - develop

jobs:
  docker-build-and-push:
    uses: EventiumProject/pipeline-template/.github/workflows/docker-build-and-push.yml@main
    with:
      image-name: eventium/eventium-portail-backend
      docker-context: .
      dockerfile-path: ./Dockerfile
      docker-push: true
    secrets: inherit

  deploy:
    uses: EventiumProject/pipeline-template/.github/workflows/deploy.yml@main
    needs: docker-build-and-push
    with:
      docker-image-name: EVENTIUM_API_DOCKER_VERSION
    secrets: inherit
```

---

## 📝 Notes importantes

### Versions et branches

- Utilisez `@main` pour la version stable
- Vous pouvez référencer une branche spécifique : `@develop`, `@feature/my-feature`
- Vous pouvez référencer un tag : `@v1.0.0`

### Structure du projet cible

Votre projet doit contenir un dossier `deploy/` avec :
- `docker-compose.prod.yml` : Configuration Docker Compose pour la production
- `docker-compose.preprod.yml` : Configuration Docker Compose pour la pré-production
- `Makefile` : Avec une target `deploy` pour lancer le déploiement

### Héritage des secrets

L'option `secrets: inherit` permet au workflow réutilisable d'accéder aux secrets du dépôt appelant.

---

## 🔧 Configuration

### Configurer les secrets GitHub

1. Allez dans **Settings → Secrets and variables → Actions**
2. Cliquez sur **New repository secret**
3. Ajoutez les secrets nécessaires listés ci-dessus

### Format du fichier .env

Le secret `ENV_FILE` ou `ENV_FILE_PREPROD` doit contenir le contenu complet de votre fichier `.env`, incluant les retours à la ligne.

---

## 📦 Tags Docker

Les images sont automatiquement taguées avec :
- `latest` : Dernière version buildée
- `<short-sha>` : Les 7 premiers caractères du SHA du commit (ex: `a1b2c3d`)

---

## 🤝 Contribution

Ce template est maintenu par l'équipe Eventium. Pour toute modification, créez une pull request sur ce dépôt.