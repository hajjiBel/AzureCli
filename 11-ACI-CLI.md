# Lab 11 - Déployer un Conteneur ACI via Azure CLI

## Vue d'overview

Ce lab vous guide à travers le déploiement automatisé d'un conteneur avec Azure CLI. Vous apprendrez à déployer rapidement des conteneurs sans passer par l'interface graphique.

---

## Objectifs Pédagogiques

À l'issue de ce lab, vous serez capable de :

- **Utiliser Azure CLI** : Commandes pour ACI
- **Déployer automatiquement** : Scripts et commandes
- **Gérer les credentials** : ACR et authentification
- **Afficher les logs** : Déboguer via CLI
- **Scaler les conteneurs** : Ajuster les ressources
- **Monitorer les performances** : Via CLI
- **Automatiser avec scripts** : Bash/PowerShell

---

## Prérequis

- Azure CLI 2.0.29+ installé
- Compte Azure actif
- Accès à Cloud Shell ou terminal local
- Compréhension basique de Azure CLI

---

## Étapes de Réalisation

### Tâche 1 : Ouvrir Cloud Shell

1. Allez à [https://shell.azure.com](https://shell.azure.com)
2. Sélectionnez **"Bash"**
3. Ou utilisez votre terminal local avec `az login`

---

### Tâche 2 : Se Connecter à Azure

1. Exécutez:
   ```bash
   az login
   ```

2. Suivez les instructions de connexion

---

### Tâche 3 : Créer un Groupe de Ressources

1. Exécutez:
   ```bash
   az group create --name myAciRG --location eastus
   ```

2. Résultat:
   ```json
   {
     "id": "/subscriptions/.../myAciRG",
     "location": "eastus",
     "name": "myAciRG",
     "properties": {"provisioningState": "Succeeded"}
   }
   ```

---

### Tâche 4 : Déployer un Conteneur

1. Exécutez:
   ```bash
   az container create \
     --resource-group myAciRG \
     --name mycontainer-cli \
     --image nginx \
     --dns-name-label mycontainer-cli \
     --ports 80 \
     --os-type Linux \
     --cpu 1 \
     --memory 1.5
   ```

2. ⏳ Attendez 2-3 minutes

#### Erreur fréquente : RegistryErrorResponse

Si vous obtenez cette erreur :

```
RegistryErrorResponse
An error response is received from the docker registry 'index.docker.io'
```

Cela signifie que Azure Container Instances n'a pas pu tirer l'image nginx depuis Docker Hub.

👉 Ce n'est pas un problème dans ta commande.
C'est un problème côté Docker Hub ou dans la connexion entre Azure ↔ Docker Hub.

**Causes possibles**

1. Pic de charge ou limite rate-limit de Docker Hub
   - Docker Hub applique un rate limit :
     - 100 pulls / 6h pour les utilisateurs anonymes
     - 200 / 6h pour les utilisateurs authentifiés

**Solution 1 — Ajouter authentification Docker Hub**

Ainsi, ACI ne fera pas un pull anonyme.

```bash
az container create \
  --resource-group myAciRG \
  --name mycontainer-cli \
  --image nginx \
  --dns-name-label mycontainer-cli \
  --ports 80 \
  --os-type Linux \
  --cpu 1 \
  --memory 1.5 \
  --registry-login-server index.docker.io \
  --registry-username YOUR_DOCKER_USERNAME \
  --registry-password YOUR_DOCKER_PASSWORD
```

---

### Tâche 5 : Vérifier le Déploiement

1. Exécutez:
   ```bash
   az container show \
     --resource-group myAciRG \
     --name mycontainer-cli \
     --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}"
   ```

2. Résultat:
   ```json
   {
     "FQDN": "mycontainer-cli.eastus.azurecontainerinstances.io",
     "ProvisioningState": "Succeeded"
   }
   ```

---

### Tâche 6 : Accéder l'Application

1. Copiez le FQDN
2. Ouvrez un navigateur
3. Allez à: [http://mycontainer-cli.eastus.azurecontainerinstances.io](http://mycontainer-cli.eastus.azurecontainerinstances.io)

---

### Tâche 7 : Afficher les Logs

1. Exécutez:
   ```bash
   az container logs \
     --resource-group myAciRG \
     --name mycontainer-cli
   ```

2. Vous verrez les logs du conteneur nginx

---

### Tâche 8 : Obtenir les Détails

1. Exécutez:
   ```bash
   az container show \
     --resource-group myAciRG \
     --name mycontainer-cli
   ```

2. Vous verrez toutes les propriétés

---

### Tâche 9 : Lister tous les Conteneurs

1. Exécutez:
   ```bash
   az container list --resource-group myAciRG -o table
   ```

2. Résultat:
   ```
   Name               ResourceGroup    Status    Image    IP:Ports
   mycontainer-cli    myAciRG          Running   nginx    xx.xx.xx.xx:80
   ```

---

### Tâche 10 : Exécuter une Commande dans le Conteneur

1. Exécutez:
   ```bash
   az container exec \
     --resource-group myAciRG \
     --name mycontainer-cli \
     --exec-command "ls -la /usr/share/nginx/html"
   ```

---

### Tâche 11 : Redémarrer le Conteneur

1. Exécutez:
   ```bash
   az container restart \
     --resource-group myAciRG \
     --name mycontainer-cli
   ```

---

### Tâche 12 : Supprimer le Conteneur

1. Exécutez:
   ```bash
   az container delete \
     --resource-group myAciRG \
     --name mycontainer-cli \
     --yes
   ```

---

### Tâche 13 : Supprimer le Groupe de Ressources

1. Exécutez:
   ```bash
   az group delete --name myAciRG --yes
   ```

---

## Résumé des Apprentissages

| Concept | Description |
|---------|-------------|
| **Azure CLI** | Interface ligne de commande pour Azure |
| **Container create** | Déployer un conteneur ACI |
| **Container show** | Afficher les détails d'un conteneur |
| **Container logs** | Consulter les logs |
| **Container exec** | Exécuter une commande dans le conteneur |
| **DNS Label** | Nom public du conteneur |
| **Automation** | Scripts pour déployer rapidement |

---

## Dépannage Courant

| Problème | Solution |
|---------|----------|
| **Commande not found** | Installez ou mettez à jour Azure CLI |
| **Authentification échouée** | Exécutez `az login` |
| **DNS Label déjà utilisé** | Utilisez un nom unique |
| **Application inaccessible** | Vérifiez le FQDN et le port |

---

## Ressources Supplémentaires

- [Azure CLI Documentation](https://learn.microsoft.com/fr-fr/cli/azure/)
- [Container Instances CLI](https://learn.microsoft.com/fr-fr/cli/azure/container)
- [Azure CLI Quickstart](https://learn.microsoft.com/en-us/cli/azure/get-started-with-azure-cli)

---

**Dernière mise à jour**: Décembre 2025
