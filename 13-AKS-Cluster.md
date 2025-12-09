# Lab 13 - Déployer un Cluster AKS avec Azure CLI (Version Finale)

## Vue d'ensemble

Ce lab vous guide à travers le déploiement d'un cluster Azure Kubernetes Service (AKS) en utilisant l'Azure CLI, puis le déploiement d'une application multi-conteneurs. Vous apprendrez les concepts essentiels de l'orchestration de conteneurs avec Kubernetes.

---

## Objectifs pédagogiques

À l'issue de ce lab, vous serez capable de :

- **Comprendre Kubernetes et AKS** : Concepts de l'orchestration de conteneurs
- **Créer un cluster AKS** : Déployer un cluster Kubernetes géré par Azure
- **Se connecter au cluster** : Utiliser kubectl pour gérer les ressources
- **Déployer une application** : Lancer une app multi-tier (frontend + redis backend)
- **Exposer l'application** : Créer des services Kubernetes LoadBalancer
- **Monitorer l'application** : Consulter logs, métriques et événements
- **Gérer les ressources** : Vérifier nœuds, pods et services
- **Scaler l'application** : Augmenter/diminuer les réplicas

---

## Prérequis

- Azure CLI 2.0.55+ installé
- Compte Azure actif
- kubectl installé localement (ou utilisé via Cloud Shell)
- Accès à Azure Cloud Shell (recommandé : https://shell.azure.com)
- Compréhension basique des concepts Kubernetes (pods, services, déploiements)
- Environ 15-20 minutes pour la création du cluster

### Permissions requises

- **Kubernetes Service Contributor** : Pour créer/gérer les clusters AKS
- **Virtual Machine Contributor** : Pour les nœuds worker
- Accès administrateur pour récupérer les credentials

---

## Architecture et ressources

```
┌─────────────────────────────────────────────────────┐
│       Azure Subscription                            │
│   ┌──────────────────────────────────────────┐     │
│   │ Groupe de Ressources : myAKSCluster      │     │
│   │ (Région: eastus)                         │     │
│   │                                          │     │
│   │  ┌──────────────────────────────────┐   │     │
│   │  │ AKS Cluster : myAKSCluster       │   │     │
│   │  │                                  │   │     │
│   │  │ Control Plane (Géré par Azure)   │   │     │
│   │  │ • Kubernetes API Server          │   │     │
│   │  │ • Scheduler                      │   │     │
│   │  │ • Controller Manager             │   │     │
│   │  │                                  │   │     │
│   │  │ ┌──────────────────────────────┐ │   │     │
│   │  │ │ Node Pool (Linux)            │ │   │     │
│   │  │ │                              │ │   │     │
│   │  │ │ ┌────────────────────────┐   │ │   │     │
│   │  │ │ │ Node 1 (VM)            │   │ │   │     │
│   │  │ │ │ • nginx-front pod      │   │ │   │     │
│   │  │ │ │ • redis pod (backup)   │   │ │   │     │
│   │  │ │ └────────────────────────┘   │ │   │     │
│   │  │ │                              │ │   │     │
│   │  │ │ (Additional nodes optionnel) │ │   │     │
│   │  │ └──────────────────────────────┘ │   │     │
│   │  │                                  │   │     │
│   │  │ Services Kubernetes :            │   │     │
│   │  │ • LoadBalancer (Frontend - IP)   │   │     │
│   │  │ • ClusterIP (Backend - interne)  │   │     │
│   │  │                                  │   │     │
│   │  │ Monitoring & Logging :           │   │     │
│   │  │ • Container Insights (optional)  │   │     │
│   │  │ • kubectl logs & describe        │   │     │
│   │  └──────────────────────────────────┘   │     │
│   │                                          │     │
│   └──────────────────────────────────────────┘     │
│                                                     │
│  Données :                                          │
│  • VM credentials : ~/.kube/config                 │
│  • App data : Redis en mémoire                     │
│  • Persistent Storage : Non utilisé pour ce lab    │
└─────────────────────────────────────────────────────┘
```

---

## Étapes de réalisation

### Tâche 1 : Accéder à Azure CLI

1. **Option A : Cloud Shell (Recommandé)**
   - Allez à <https://shell.azure.com>
   - Sélectionnez **Bash**
   - Vous êtes automatiquement authentifiés

2. **Option B : Terminal local**
   - Ouvrez votre terminal (PowerShell, Bash, ou Command Prompt)
   - Exécutez :

   ```bash
   az login
   ```

   - Suivez les instructions pour vous authentifier

---

### Tâche 2 : Vérifier la version d'Azure CLI

1. Exécutez :

   ```bash
   az --version
   ```

2. Vérifiez que la version est **2.0.55+** (généralement 2.x.x)

---

### Tâche 3 : Créer un groupe de ressources

1. Exécutez :

   ```bash
   az group create --name myAKSCluster --location eastus
   ```

2. Résultat attendu :

   ```json
   {
     "id": "/subscriptions/.../myAKSCluster",
     "location": "eastus",
     "name": "myAKSCluster",
     "properties": {
       "provisioningState": "Succeeded"
     }
   }
   ```

---

### Tâche 4 : Créer le cluster AKS

⚠️ **IMPORTANT** : Cette étape peut prendre **5-10 minutes**. Soyez patient !

1. Exécutez :

   ```bash
   az aks create \
     --resource-group myAKSCluster \
     --name myAKSCluster \
     --node-count 1 \
     --enable-addons monitoring \
     --generate-ssh-keys
   ```

2. Cette commande crée :
   - Cluster Kubernetes avec **1 nœud Linux**
   - **Monitoring** activé (Container Insights optionnel)
   - **Clés SSH** générées automatiquement

3. ⏳ **Attendez 5-10 minutes** que le déploiement soit terminé

4. À la fin, vous verrez une sortie JSON volumineux contenant :
   - `provisioningState: "Succeeded"`
   - `fqdn` du cluster
   - `kubeConfig` encodée

> **Que se passe-t-il pendant ce temps ?**
> - Azure crée la VM pour le nœud worker
> - Installe les composants Kubernetes
> - Configure le réseau et le stockage
> - Configure les identités gérées

---

### Tâche 5 : Configurer kubectl

1. Téléchargez les credentials du cluster :

   ```bash
   az aks get-credentials --resource-group myAKSCluster --name myAKSCluster
   ```

2. Vous verrez :

   ```
   Merged "myAKSCluster" as current context in /home/<user>/.kube/config
   ```

   Cela crée/met à jour le fichier de configuration kubeconfig local.

3. Vérifiez que kubectl est configuré :

   ```bash
   kubectl cluster-info
   ```

4. Résultat attendu :

   ```
   Kubernetes control plane is running at https://myakscluster-xxxxx.hcp.eastus.azmk8s.io:443
   CoreDNS is running at https://myakscluster-xxxxx.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
   ```

---

### Tâche 6 : Vérifier les nœuds

1. Listez les nœuds du cluster :

   ```bash
   kubectl get nodes
   ```

2. Résultat attendu :

   ```
   NAME                        STATUS   ROLES   AGE   VERSION
   aks-nodepool1-12345678-0    Ready    agent   2m    v1.29.2
   ```

   > Le statut doit être **Ready**

---

### Tâche 7 : Télécharger le manifeste Kubernetes

⚠️ **IMPORTANT** : Le fichier original contenait une image Microsoft supprimée. Utilisez le fichier corrigé ci-dessous.

1. Créez le fichier `azure-vote.yaml` :

   ```
  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-back
  namespace: default
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
        image: redis:6.0
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
  namespace: default
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
  namespace: default
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
        image: nginx:latest
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
  namespace: default
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: azure-vote-front

   ```

   **Points importants du fichier corrigé** :
   - **Backend** : `redis:6.0` (image Docker Hub valide)
   - **Frontend** : `nginx:latest` (image publique stable - **CORRIGÉ** de `mcr.microsoft.com/azuredocs/azure-vote-front:v1`)
   - **Service Frontend** : Type `LoadBalancer` (crée une IP publique)
   - **Service Backend** : Type `ClusterIP` (interne seulement)
   - **Variable d'environnement** : `REDIS: "azure-vote-back"` (nom du service backend)

---

### Tâche 8 : Déployer l'application

1. Appliquez le manifeste :

   ```bash
   kubectl apply -f azure-vote.yaml
   ```

2. Résultat attendu :

   ```
   deployment.apps/azure-vote-back created
   service/azure-vote-back created
   deployment.apps/azure-vote-front created
   service/azure-vote-front created
   ```

   ⏳ Les pods peuvent prendre **1-2 minutes** pour démarrer.

---

### Tâche 9 : Vérifier les déploiements

1. Listez les déploiements :

   ```bash
   kubectl get deployments
   ```

2. Résultat attendu :

   ```
   NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
   azure-vote-back      1/1     1            1           45s
   azure-vote-front     1/1     1            1           45s
   ```

   > **READY 1/1** signifie que le pod est lancé et prêt

---

### Tâche 10 : Vérifier les pods

1. Listez tous les pods :

   ```bash
   kubectl get pods
   ```

2. Résultat attendu :

   ```
   NAME                                 READY   STATUS    RESTARTS   AGE
   azure-vote-back-xxxxxxxx-xxxxx       1/1     Running   0          60s
   azure-vote-front-xxxxxxxx-xxxxx      1/1     Running   0          60s
   ```

   > **STATUS: Running** = Pod démarré avec succès

3. **Si STATUS = Pending ou ImagePullBackOff** : Attendez 30-60 secondes supplémentaires

---

### Tâche 11 : Obtenir l'adresse IP publique

1. Attendez que l'IP soit attribuée (peut prendre 2-3 minutes) :

   ```bash
   kubectl get service azure-vote-front --watch
   ```

2. Résultat attendu :

   ```
   NAME               TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
   azure-vote-front   LoadBalancer   10.0.1.xxx     52.179.23.131  80:32xxx/TCP   2m
   ```

   - Initialement : `EXTERNAL-IP` = `<pending>`
   - Après quelques minutes : Une **IP publique** apparaît

3. **Appuyez sur Ctrl+C** pour arrêter le watch

---

### Tâche 12 : Accéder à l'application

1. Copiez la valeur de **EXTERNAL-IP** (ex: `52.179.23.131`)
2. Ouvrez un navigateur
3. Allez à : `http://<EXTERNAL-IP>` (ex: http://52.179.23.131)
4. Vous verrez la **page d'accueil NGINX** (application déployée avec succès)

---

### Tâche 13 : Consulter les logs des pods

1. Listez les noms de pods :

   ```bash
   kubectl get pods
   ```

2. Consultez les logs du frontend :

   ```bash
   kubectl logs <nom-du-pod-frontend>
   ```

   Exemple :

   ```bash
   kubectl logs azure-vote-front-xxxxxxxx-xxxxx
   ```

3. Consultez les logs du backend :

   ```bash
   kubectl logs <nom-du-pod-backend>
   ```

4. Vous verrez des lignes comme :

   ```
   [1] 18 Nov 2024 12:34:56.789 * The server is now ready to accept connections on port 6379
   ```

---

### Tâche 14 : Décrire un pod (pour le dépannage)

1. Exécutez :

   ```bash
   kubectl describe pod <nom-du-pod>
   ```

2. Vous verrez des informations détaillées :
   - Labels
   - Containers
   - State
   - **Events** (très utile pour le dépannage)

3. Regardez la section **Events** pour les erreurs

---

### Tâche 15 : Scaler l'application (Augmenter les réplicas)

1. Augmentez les réplicas du frontend à 3 :

   ```bash
   kubectl scale deployment azure-vote-front --replicas=3
   ```

2. Vérifiez :

   ```bash
   kubectl get pods
   ```

3. Résultat : Vous verrez **3 pods** du frontend en cours d'exécution

4. Réduisez à 1 replica :

   ```bash
   kubectl scale deployment azure-vote-front --replicas=1
   ```

---

### Tâche 16 : Afficher les services

1. Listez tous les services :

   ```bash
   kubectl get services
   ```

2. Résultat :

   ```
   NAME               TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
   kubernetes         ClusterIP      10.0.0.1       <none>           443/TCP        15m
   azure-vote-back    ClusterIP      10.0.1.xxx     <none>           6379/TCP       10m
   azure-vote-front   LoadBalancer   10.0.1.yyy     52.179.23.131    80:32xxx/TCP   10m
   ```

   > **ClusterIP** = Interne seulement  
   > **LoadBalancer** = IP publique

---

### Tâche 17 : Nettoyer l'application

1. Supprimez l'application :

   ```bash
   kubectl delete -f azure-vote.yaml
   ```

2. Vérifiez que les pods ont disparu :

   ```bash
   kubectl get pods
   ```

   > Vous verrez le message : `No resources found in default namespace.`

---

### Tâche 18 : Supprimer le cluster AKS (Optionnel)

⚠️ **ATTENTION** : Cette action supprime tout et génère des frais.

1. Supprimez le cluster (quelques minutes) :

   ```bash
   az aks delete --resource-group myAKSCluster --name myAKSCluster --yes
   ```

2. Supprimez le groupe de ressources entier :

   ```bash
   az group delete --name myAKSCluster --yes
   ```

---

## Résumé des apprentissages

| Concept | Description | Exemple |
|---------|-------------|---------|
| **Kubernetes** | Plateforme d'orchestration de conteneurs | Gère le cycle de vie des pods |
| **AKS** | Service Kubernetes géré par Azure | Gère le control plane automatiquement |
| **Cluster** | Ensemble de nœuds exécutant Kubernetes | `myAKSCluster` |
| **Nœud** | Machine virtuelle Linux/Windows | `aks-nodepool1-12345678-0` |
| **Pod** | Plus petite unité Kubernetes déployable | `azure-vote-front-xxxxx` |
| **Déploiement** | Gère les pods et leurs réplicas | `azure-vote-back`, `azure-vote-front` |
| **Service** | Expose les pods au réseau | `LoadBalancer`, `ClusterIP` |
| **LoadBalancer** | Crée une IP publique Azure | `azure-vote-front` (IP: 52.179.23.131) |
| **ClusterIP** | Service interne (default) | `azure-vote-back` (interne seulement) |
| **kubectl** | CLI pour Kubernetes | Commandes: apply, get, logs, scale |
| **Image Docker** | Template pour les conteneurs | `redis:6.0`, `nginx:latest` |
| **Replica** | Nombre de copies d'un pod | Scaling de 1 à 3 pods |

---

## Questions de révision

1. **Quelle est la différence entre un nœud et un pod ?**
   - Un **nœud** est une VM qui exécute le runtime Kubernetes
   - Un **pod** est le conteneur (application) qui s'exécute sur le nœud

2. **Pourquoi utiliser Kubernetes ?**
   - Orchestration automatique des conteneurs
   - Scaling automatique
   - Gestion de la disponibilité
   - Mises à jour sans temps d'arrêt

3. **Quel est le rôle d'un Service LoadBalancer ?**
   - Crée une adresse IP publique Azure
   - Route le trafic vers les pods
   - Assure la répartition de charge

4. **Comment afficher les logs ?**
   - `kubectl logs <nom-pod>`

5. **Comment scaler les pods ?**
   - `kubectl scale deployment <nom> --replicas=<nombre>`

---

## Dépannage courant

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| **Déploiement lent** | C'est normal, création des nœuds | Attendez 5-10 minutes |
| **Pod en Pending** | Image en cours de téléchargement ou ressources insuffisantes | Attendez 30-60 secondes ou vérifiez `kubectl describe pod` |
| **ImagePullBackOff** | Image Docker introuvable ou invalide | Vérifiez le nom de l'image (utiliser `nginx:latest` au lieu de l'image Microsoft supprimée) |
| **IP publique ne s'affiche pas** | Allocation en cours | Attendez 2-3 minutes |
| **Application inaccessible** | Port, NSG, ou routage bloqué | Vérifiez l'IP avec `kubectl get svc` et testez `curl http://<IP>:80` |
| **Erreur de permissions** | Utilisateur non autorisé | Vérifiez le rôle RBAC (Kubernetes Service Contributor) |
| **Cluster ne se crée pas** | Quota Azure dépassé ou région invalide | Vérifiez les quotas Azure et la région disponible |

### Commandes utiles pour le dépannage

```bash
# Détails complets d'un pod
kubectl describe pod <nom-pod>

# Logs en continu
kubectl logs -f <nom-pod>

# Logs des 10 dernières minutes
kubectl logs --since=10m <nom-pod>

# Événements du cluster
kubectl get events

# État détaillé du cluster
kubectl cluster-info dump

# Accès shell au conteneur (si bash disponible)
kubectl exec -it <nom-pod> -- /bin/bash
```



---



## Ressources supplémentaires

- [Documentation AKS officielle](https://learn.microsoft.com/fr-fr/azure/aks/)
- [Documentation Kubernetes](https://kubernetes.io/docs/home/)
- [Cheat Sheet kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Azure Docs - Déployer AKS](https://learn.microsoft.com/fr-fr/azure/aks/kubernetes-walkthrough)
- [Docker Hub Images](https://hub.docker.com)

---

**Version** : 3.0 FINALE (Décembre 2025)  
**Statut** : ✅ Testé et validé (Avec corrections d'images)  
**Dernière mise à jour** : 09 Décembre 2025
