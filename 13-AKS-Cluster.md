# Lab 13 - Déployer un Cluster AKS avec Azure CLI

## Vue d'overview

Ce lab vous guide à travers le déploiement d'un cluster Azure Kubernetes Service (AKS) en utilisant l'Azure CLI, puis le déploiement d'une application multi-conteneurs. Vous apprendrez les concepts essentiels de l'orchestration de conteneurs.

---

## Objectifs Pédagogiques

À l'issue de ce lab, vous serez capable de :

- **Comprendre Kubernetes et AKS** : Concepts de l'orchestration
- **Créer un cluster AKS** : Déployer un cluster Kubernetes géré
- **Se connecter au cluster** : Utiliser kubectl pour gérer
- **Déployer une application** : Lancer une app multi-tier
- **Exposer l'application** : Créer des services Kubernetes
- **Monitorer l'application** : Consulter logs et métriques
- **Gérer les ressources** : Vérifier nœuds, pods et services

---


## Prérequis

- Azure CLI version 2.0.55+ installé
- Compte Azure actif
- kubectl installé localement
- Accès à Azure Cloud Shell (recommandé)
- Compréhension basique de Kubernetes

---

## Architecture et Ressources

```
┌─────────────────────────────────────┐
│       Azure Subscription            │
│   ┌────────────────────────────┐   │
│   │ Groupe de Ressources       │   │
│   │ myAKSCluster               │   │
│   │ (Région: eastus)           │   │
│   │                            │   │
│   │  ┌──────────────────────┐ │   │
│   │  │ AKS Cluster          │ │   │
│   │  │ myAKSCluster         │ │   │
│   │  │                      │ │   │
│   │  │ ┌──────────────────┐│ │   │
│   │  │ │ Node 1 (Linux)   ││ │   │
│   │  │ │ ┌─ azure-vote   ││ │   │
│   │  │ │ └─ front pods    ││ │   │
│   │  │ └──────────────────┘│ │   │
│   │  │                      │ │   │
│   │  │ ┌──────────────────┐│ │   │
│   │  │ │ Node 2+ (opt.)   ││ │   │
│   │  │ │ ┌─ azure-vote   ││ │   │
│   │  │ │ └─ back pods     ││ │   │
│   │  │ └──────────────────┘│ │   │
│   │  └──────────────────────┘ │   │
│   │                            │   │
│   │ Services Kubernetes:       │   │
│   │ • LoadBalancer (Frontend)  │   │
│   │ • ClusterIP (Backend)      │   │
│   │                            │   │
│   │ Monitoring:                │   │
│   │ • Container Insights       │   │
│   └────────────────────────────┘   │
└─────────────────────────────────────┘
```

---

## Étapes de Réalisation

### Tâche 1 : Ouvrir Azure CLI

1. Allez à https://shell.azure.com (Cloud Shell) - **Recommandé**
2. Ou ouvrez votre terminal local et tapez: `az login`
3. Authentifiez-vous avec votre compte Azure

---

### Tâche 2 : Créer un Groupe de Ressources

1. Exécutez:
   ```bash
   az group create --name myAKSCluster --location eastus
   ```

2. Résultat:
   ```json
   {
     "id": "/subscriptions/.../myAKSCluster",
     "name": "myAKSCluster",
     "properties": {
       "provisioningState": "Succeeded"
     }
   }
   ```

---

### Tâche 3 : Créer le Cluster AKS

⚠️ **Cette étape peut prendre 5-10 minutes, soyez patient!**

1. Exécutez:
   ```bash
   az aks create \
     --resource-group myAKSCluster \
     --name myAKSCluster \
     --node-count 1 \
     --enable-addons monitoring \
     --generate-ssh-keys
   ```

2. Cette commande crée:
   - Cluster Kubernetes avec **1 nœud**
   - **Monitoring** activé (Container Insights)
   - **Clés SSH** générées

3. Attendez que le déploiement soit terminé (5-10 minutes)

4. Vous verrez un JSON volumineux avec les détails

---

### Tâche 4 : Configurer kubectl

1. Téléchargez les credentials:
   ```bash
   az aks get-credentials --resource-group myAKSCluster --name myAKSCluster
   ```

2. Vérifiez que kubectl est configuré:
   ```bash
   kubectl cluster-info
   ```

3. Résultat:
   ```
   Kubernetes control plane is running at https://myakscluster-xxxx.hcp.eastus.azmk8s.io:443
   ```

---

### Tâche 5 : Vérifier les Nœuds

1. Listez les nœuds:
   ```bash
   kubectl get nodes
   ```

2. Résultat:
   ```
   NAME                        STATUS   ROLES   AGE   VERSION
   aks-nodepool1-12345678-0    Ready    agent   2m    v1.29.2
   ```

---

### Tâche 6 : Créer le Fichier YAML de Déploiement

1. Créez un fichier `azure-vote.yaml`:
   ```bash
   cat > azure-vote.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-back
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-back
        image: redis
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-front
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-front
        image: microsoft/azure-vote-front:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
EOF
   ```

---

### Tâche 7 : Déployer l'Application

1. Déployez:
   ```bash
   kubectl apply -f azure-vote.yaml
   ```

2. Résultat:
   ```
   deployment.apps/azure-vote-back created
   service/azure-vote-back created
   deployment.apps/azure-vote-front created
   service/azure-vote-front created
   ```

---

### Tâche 8 : Vérifier les Déploiements

1. Listez:
   ```bash
   kubectl get deployments
   ```

2. Résultat:
   ```
   NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
   azure-vote-back      1/1     1            1           30s
   azure-vote-front     1/1     1            1           30s
   ```

---

### Tâche 9 : Vérifier les Pods

1. Listez:
   ```bash
   kubectl get pods
   ```

2. Résultat:
   ```
   NAME                                 READY   STATUS    RESTARTS   AGE
   azure-vote-back-xxxxx                1/1     Running   0          45s
   azure-vote-front-xxxxx               1/1     Running   0          45s
   ```

---

### Tâche 10 : Monitorer l'IP Publique

1. Attendez que l'IP soit attribuée:
   ```bash
   kubectl get service azure-vote-front --watch
   ```

2. Initialement: `EXTERNAL-IP` est `<pending>`

3. Après quelques minutes: Une IP publique apparaît

4. **Appuyez sur Ctrl+C** pour arrêter

---

### Tâche 11 : Accéder à l'Application

1. Copiez l'EXTERNAL-IP
2. Ouvrez un navigateur
3. Allez à: http://52.179.23.131
4. Vous verrez l'application **Azure Vote App**
5. Cliquez pour voter

---

### Tâche 12 : Consulter les Logs

1. Obtenez les pods:
   ```bash
   kubectl get pods
   ```

2. Consultez les logs:
   ```bash
   kubectl logs <pod-name>
   ```

---

### Tâche 13 : Décrire un Pod

1. Exécutez:
   ```bash
   kubectl describe pod <pod-name>
   ```

2. Vous verrez les détails complets

---

### Tâche 14 : Scaling (Optionnel)

1. Augmentez les réplicas:
   ```bash
   kubectl scale deployment azure-vote-front --replicas=3
   ```

2. Vérifiez:
   ```bash
   kubectl get pods
   ```

3. Vous verrez **3 pods** en cours d'exécution

---

### Tâche 15 : Nettoyer les Ressources

1. Supprimez l'application:
   ```bash
   kubectl delete -f azure-vote.yaml
   ```

2. Supprimez le cluster (quelques minutes):
   ```bash
   az aks delete --resource-group myAKSCluster --name myAKSCluster --yes
   ```

3. Supprimez le groupe de ressources:
   ```bash
   az group delete --name myAKSCluster --yes
   ```

---

## Résumé des Apprentissages

| Concept | Description |
|---------|-------------|
| **Kubernetes** | Plateforme d'orchestration de conteneurs |
| **AKS** | Service Kubernetes géré par Azure |
| **Cluster** | Ensemble de nœuds exécutant des conteneurs |
| **Nœud** | Machine (VM) dans le cluster |
| **Pod** | Plus petite unité Kubernetes |
| **Déploiement** | Gère les pods |
| **Service** | Expose les pods |
| **LoadBalancer** | Crée une IP publique |
| **kubectl** | Outil CLI pour Kubernetes |

---

## Questions de Révision

1. Quelle est la différence entre un nœud et un pod?
2. Pourquoi utiliser Kubernetes?
3. Quel est le rôle d'un Service LoadBalancer?
4. Comment afficher les logs?
5. Comment scaler les pods?

---

## Dépannage Courant

| Problème | Solution |
|---------|----------|
| **Déploiement lent** | C'est normal, attendez 10 minutes |
| **Pod en Pending** | Vérifiez les ressources disponibles |
| **IP publique ne s'affiche pas** | Attendez quelques minutes |
| **Application inaccessible** | Vérifiez l'IP et le port 80 |

---

## Ressources Supplémentaires

- [AKS Documentation](https://learn.microsoft.com/fr-fr/azure/aks/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

**Dernière mise à jour**: Décembre 2025
