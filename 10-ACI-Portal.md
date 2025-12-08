# Lab 10 - Déployer un Conteneur ACI via le Portail Azure

## Vue d'overview

Ce lab vous guide à travers le déploiement d'un conteneur Docker sans serveur avec Azure Container Instances. Vous apprendrez à déployer rapidement une application conteneurisée sans gérer l'infrastructure.

---

## Objectifs Pédagogiques

À l'issue de ce lab, vous serez capable de :

- **Comprendre ACI** : Azure Container Instances sans serveur
- **Déployer un conteneur** : Via le Portail Azure
- **Accéder l'application** : Via FQDN publique
- **Configurer le réseau** : Ports et protocoles
- **Consulter les logs** : Déboguer l'application
- **Gérer les ressources** : Créer et supprimer des conteneurs
- **Monitorer les performances** : CPU et mémoire

---

## Prérequis

- Compte Azure actif
- Navigateur web
- Accès au Portail Azure
- Compréhension de base des conteneurs

---

## Architecture et Ressources

```
┌──────────────────────────────┐
│   Azure Subscription         │
│  ┌────────────────────────┐  │
│  │  Groupe de Ressources  │  │
│  │  myAciResourceGroup    │  │
│  │                        │  │
│  │ ┌──────────────────┐   │  │
│  │ │ Container Group  │   │  │
│  │ │ mycontainer      │   │  │
│  │ │                  │   │  │
│  │ │ ┌──────────────┐ │   │  │
│  │ │ │ Conteneur    │ │   │  │
│  │ │ │ nginx:latest │ │   │  │
│  │ │ │ Port: 80     │ │   │  │
│  │ │ └──────────────┘ │   │  │
│  │ │                  │   │  │
│  │ │ FQDN Public:     │   │  │
│  │ │ mycontainer.     │   │  │
│  │ │ eastus.aci.      │   │  │
│  │ │ azurecontainer   │   │  │
│  │ │ instances.io     │   │  │
│  │ └──────────────────┘   │  │
│  │                        │  │
│  │ Ressources:            │  │
│  │ • CPU: 1 core         │  │
│  │ • Mémoire: 1 GB       │  │
│  │ • Restart policy      │  │
│  └────────────────────────┘  │
└──────────────────────────────┘
```

---

## Étapes de Réalisation

### Tâche 1 : Ouvrir le Portail Azure

1. Allez à **https://portal.azure.com**
2. Connectez-vous avec votre compte Azure

---

### Tâche 2 : Créer un Groupe de Ressources

1. Cliquez sur **"+ Créer une ressource"**
2. Recherchez **"Groupe de ressources"**
3. Cliquez sur **"Créer"**
4. Entrez:
   - **Nom**: myAciResourceGroup
   - **Région**: eastus
5. Cliquez sur **"Créer"**

---

### Tâche 3 : Créer une Instance de Conteneur

1. Allez à **"+ Créer une ressource"**
2. Recherchez **"Container Instances"**
3. Cliquez sur **"Créer"**

---

### Tâche 4 : Configurer les Détails du Conteneur

1. Remplissez:
   - **Groupe de ressources**: myAciResourceGroup
   - **Nom du conteneur**: mycontainer
   - **Région**: eastus
   - **Source d'image**: Docker Hub
   - **Image**: nginx
   - **Balise d'image**: latest

---

### Tâche 5 : Configurer le Réseau

1. Cliquez sur **"Suivant: Réseau >"**
2. Entrez:
   - **Nom DNS**: mycontainer
   - **Port**: 80
   - **Protocole**: TCP
3. Cliquez sur **"Suivant"**

---

### Tâche 6 : Configurer les Ressources

1. Entrez:
   - **CPU**: 1
   - **Mémoire (GB)**: 1
2. Cliquez sur **"Examiner + créer"**

---

### Tâche 7 : Créer le Conteneur

1. Vérifiez les paramètres
2. Cliquez sur **"Créer"**
3. ⏳ Attendez 2-3 minutes

---

### Tâche 8 : Vérifier le Déploiement

1. Allez à votre **groupe de ressources**
2. Cliquez sur votre **conteneur**
3. Vérifiez l'état: **En cours d'exécution**

---

### Tâche 9 : Obtenir l'URL Publique

1. Dans les **Propriétés**
2. Copier le **FQDN**: mycontainer.eastus.azurecontainerinstances.io
3. Notez aussi l'**IP externe**

---

### Tâche 10 : Accéder l'Application

1. Ouvrez un navigateur
2. Allez à: http://mycontainer.eastus.azurecontainerinstances.io
3. Vous verrez la page nginx par défaut

---

### Tâche 11 : Consulter les Logs

1. Retournez au conteneur dans le Portail
2. Cliquez sur **"Logs"**
3. Vous verrez les logs de l'application:
   ```
   GET / HTTP/1.1
   GET /favicon.ico HTTP/1.1
   ```

---

### Tâche 12 : Vérifier les Propriétés

1. Cliquez sur **"Propriétés"**
2. Vérifiez:
   - État: En cours d'exécution
   - CPU utilisé
   - Mémoire utilisée
   - FQDN public

---


### Tâche 13 : Supprimer le Conteneur

1. Cliquez sur **"Supprimer"**
2. Confirmez: **Oui**
3. Le conteneur est supprimé immédiatement

---

### Tâche 14 : Nettoyer les Ressources

1. Supprimez le **groupe de ressources**
2. Allez à votre **groupe de ressources**
3. Cliquez sur **"Supprimer le groupe de ressources"**
4. Confirmez

---

## Résumé des Apprentissages

| Concept | Description |
|---------|-------------|
| **ACI** | Azure Container Instances sans serveur |
| **Conteneur** | Environnement isolé pour une application |
| **Image Docker** | Blueprint pour créer des conteneurs |
| **FQDN** | Nom de domaine pleinement qualifié |
| **Port d'écoute** | Port sur lequel le conteneur écoute |
| **Logs** | Sortie de l'application conteneurisée |
| **CPU et Mémoire** | Ressources allouées au conteneur |

---


## Dépannage Courant

| Problème | Solution |
|---------|----------|
| **Conteneur n'apparaît pas** | Attendez quelques minutes et rafraîchissez |
| **Application inaccessible** | Vérifiez le FQDN et le port |
| **Logs vides** | L'application n'a pas reçu de requête |
| **CPU/Mémoire dépassée** | Augmentez les ressources allouées |

---

## Ressources Supplémentaires

- [Azure Container Instances Documentation](https://learn.microsoft.com/fr-fr/azure/container-instances/)
- [Container Instances Tutorial](https://learn.microsoft.com/fr-fr/azure/container-instances/container-instances-quickstart)
- [Docker Hub Images](https://hub.docker.com)

---

**Dernière mise à jour**: Décembre 2025
