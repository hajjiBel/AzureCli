# Lab 12 - Pousser une Image Docker vers Azure Container Registry

## Vue d'overview

Ce lab vous guide à travers la création d'une image Docker personnalisée et son envoi vers un registre privé Azure. Vous apprendrez à gérer des images de conteneurs en tant que code.

---

## Objectifs Pédagogiques

À l'issue de ce lab, vous serez capable de :

- **Créer une image Docker** : Écrire un Dockerfile
- **Construire l'image** : Compiler le Dockerfile
- **Créer un registre ACR** : Azure Container Registry privé
- **Taguer l'image** : Ajouter des étiquettes de version
- **Pousser l'image** : Envoyer vers le registre
- **Gérer les versions** : Plusieurs tags et versions
- **Sécuriser l'accès** : Contrôle d'accès aux images

---

## Prérequis

- Docker Engine installé localement
- Azure CLI 2.0.29+ installé
- Compte Azure actif
- Git installé (optionnel)
- Terminal/PowerShell avec accès administrateur

---

## Architecture et Ressources

```
┌─────────────────────────────────────┐
│     Azure Subscription              │
│  ┌───────────────────────────────┐ │
│  │  Groupe de Ressources         │ │
│  │  myAcrRG                      │ │
│  │                               │ │
│  │ ┌─────────────────────────┐   │ │
│  │ │ Container Registry (ACR)│   │ │
│  │ │ myacr.azurecr.io        │   │ │
│  │ │                         │   │ │
│  │ │ Images:                 │   │ │
│  │ │ • myapp:v1.0           │   │ │
│  │ │ • myapp:latest         │   │ │
│  │ │ • myapp:v1.1           │   │ │
│  │ │                         │   │ │
│  │ │ Authentification:       │   │ │
│  │ │ • Admin user            │   │ │
│  │ │ • Service Principal     │   │ │
│  │ └─────────────────────────┘   │ │
│  │                               │ │
│  │ Sécurité:                      │ │
│  │ • Registre privé              │ │
│  │ • Chiffrement au repos        │ │
│  └───────────────────────────────┘ │
└─────────────────────────────────────┘
```

---

## Étapes de Réalisation

### Tâche 1 : Créer un Dockerfile

1. Créez un dossier:
   ```bash
   mkdir myapp
   cd myapp
   ```

2. Créez un fichier `Dockerfile`:
   ```dockerfile
   FROM nginx:latest
   COPY index.html /usr/share/nginx/html/
   EXPOSE 80
   ```

3. Créez un fichier `index.html`:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Mon App Azure</title>
   </head>
   <body>
       <h1>Bienvenue sur mon app Azure!</h1>
       <p>Image construite avec Docker et ACR</p>
   </body>
   </html>
   ```

---

### Tâche 2 : Construire l'Image Localement

1. Exécutez:
   ```bash
   docker build -t myapp:v1.0 .
   ```

2. Vérifiez:
   ```bash
   docker images | grep myapp
   ```

3. Résultat:
   ```
   REPOSITORY   TAG    IMAGE ID     CREATED
   myapp        v1.0   1234abcd     2 minutes ago
   ```

---

### Tâche 3 : Tester l'Image Localement

1. Exécutez:
   ```bash
   docker run -d -p 8080:80 --name test-app myapp:v1.0
   ```

2. Accédez à: http://localhost:8080

3. Arrêtez le conteneur:
   ```bash
   docker stop test-app
   docker rm test-app
   ```

---

### Tâche 4 : Créer un Groupe de Ressources Azure

1. Exécutez:
   ```bash
   az group create --name myAcrRG --location eastus
   ```

---

### Tâche 5 : Créer un Registre ACR

1. Exécutez:
   ```bash
   az acr create \
     --resource-group myAcrRG \
     --name myacr \
     --sku Basic
   ```

2. Attendez 2-3 minutes

---

### Tâche 6 : Se Connecter au Registre

1. Exécutez:
   ```bash
   az acr login --name myacr
   ```

2. Résultat:
   ```
   Login Succeeded
   ```

---

### Tâche 7 : Obtenir le Serveur de Connexion

1. Exécutez:
   ```bash
   az acr show --name myacr --query loginServer
   ```

2. Résultat:
   ```
   "myacr.azurecr.io"
   ```

---

### Tâche 8 : Taguer l'Image

1. Exécutez:
   ```bash
   docker tag myapp:v1.0 myacr.azurecr.io/myapp:v1.0
   docker tag myapp:v1.0 myacr.azurecr.io/myapp:latest
   ```

2. Vérifiez:
   ```bash
   docker images | grep myacr
   ```

---

### Tâche 9 : Pousser l'Image vers ACR

1. Exécutez:
   ```bash
   docker push myacr.azurecr.io/myapp:v1.0
   docker push myacr.azurecr.io/myapp:latest
   ```

2. ⏳ Attendez que le push soit complété

---

### Tâche 10 : Lister les Images dans ACR

1. Exécutez:
   ```bash
   az acr repository list --name myacr
   ```

2. Résultat:
   ```
   [
     "myapp"
   ]
   ```

---

### Tâche 11 : Afficher les Tags

1. Exécutez:
   ```bash
   az acr repository show-tags --name myacr --repository myapp
   ```

2. Résultat:
   ```
   [
     "latest",
     "v1.0"
   ]
   ```

---

### Tâche 12 : Afficher les Manifests

1. Exécutez:
   ```bash
   az acr repository show --name myacr --repository myapp
   ```

---

### Tâche 13 : Créer une Nouvelle Version

1. Modifiez `index.html`
2. Reconstruisez:
   ```bash
   docker build -t myapp:v1.1 .
   docker tag myapp:v1.1 myacr.azurecr.io/myapp:v1.1
   docker push myacr.azurecr.io/myapp:v1.1
   ```

---

### Tâche 14 : Nettoyer

1. Supprimez l'ACR:
   ```bash
   az acr delete --name myacr --resource-group myAcrRG --yes
   ```

2. Supprimez le groupe de ressources:
   ```bash
   az group delete --name myAcrRG --yes
   ```

---

## Résumé des Apprentissages

| Concept | Description |
|---------|-------------|
| **Dockerfile** | Fichier pour construire une image Docker |
| **Docker image** | Template pour créer des conteneurs |
| **Docker build** | Compiler une image |
| **ACR** | Azure Container Registry privé |
| **Docker push** | Envoyer l'image vers un registre |
| **Tag** | Étiquette de version |
| **Authentification** | Accès sécurisé au registre |

---

## Questions de Révision

1. Qu'est-ce qu'un Dockerfile?
2. Pourquoi utiliser un registre privé?
3. Comment taguer une image Docker?
4. Comment pousser une image vers ACR?
5. Comment gérer plusieurs versions?

---

## Dépannage Courant

| Problème | Solution |
|---------|----------|
| **Erreur: docker not found** | Installez Docker Desktop |
| **Login failed** | Exécutez `az acr login --name myacr` |
| **Push échoue** | Vérifiez les permissions et les tags |
| **Image not found** | Vérifiez le nom et le tag |

---

## Ressources Supplémentaires

- [Docker Documentation](https://docs.docker.com/)
- [Azure Container Registry](https://learn.microsoft.com/fr-fr/azure/container-registry/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)

---

**Dernière mise à jour**: Décembre 2025
