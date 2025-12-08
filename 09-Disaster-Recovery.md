# Lab 09 - Configurer la Récupération Après Sinistre avec Site Recovery

## Vue d'overview

Ce lab vous guide à travers la configuration d'Azure Site Recovery pour la réplication et le basculement d'une VM. Vous apprendrez à mettre en place la continuité d'activité et la reprise sur sinistre.

---

## Objectifs Pédagogiques

À l'issue de ce lab, vous serez capable de :

- **Comprendre le Disaster Recovery** : Concepts RTO et RPO
- **Configurer Site Recovery** : Mettre en place la réplication
- **Tester un failover** : Faire basculer vers une région secondaire
- **Monitorer la réplication** : Vérifier l'état de la réplication
- **Planifier la reprise** : Créer un plan de basculement
- **Gérer les ressources cibles** : Configurer la région secondaire
- **Nettoyer les ressources** : Supprimer la réplication



---

## Prérequis

- Une VM existante dans Azure (Lab 01 ou 02)
- Compte Azure actif
- Accès administrateur au Portail Azure
- Compréhension des VMs et des régions Azure

---

## Architecture et Ressources

```
┌─────────────────────────────────────────────────────┐
│         Azure Subscription                          │
│                                                     │
│  Région Primaire (eastus)     Région Secondaire    │
│  ┌──────────────────────┐     (westus)             │
│  │ Vault de Récupération├────────────────────┐     │
│  │ myRecoveryVault      │                    │     │
│  │                      │     ┌──────────────┴──┐  │
│  │ ┌──────────────────┐ │     │                 │  │
│  │ │ VM Source        │ │     │ VM Répliquée    │  │
│  │ │ (Primaire)       │ │     │ (En attente)    │  │
│  │ │                  │───────│                 │  │
│  │ └──────────────────┘ │     │                 │  │
│  │                      │     └─────────────────┘  │
│  │ État de Réplication: │                          │
│  │ • En cours           │     En cas de sinistre:  │
│  │ • Synchronisé        │     • Failover           │
│  │ • Sain               │     • Failback           │
│  └──────────────────────┘                          │
│                                                     │
│  Monitoring:                                       │
│  • RTO (Recovery Time Objective)                   │
│  • RPO (Recovery Point Objective)                  │
│  • Points de récupération                          │
└─────────────────────────────────────────────────────┘
```

---

## Étapes de Réalisation

### Tâche 1 : Créer un Vault de Récupération

1. Allez au **Portail Azure** (https://portal.azure.com)
2. Cliquez sur **"+ Créer une ressource"**
3. Recherchez **"Recovery Services vault"**
4. Cliquez sur **"Créer"**

---

### Tâche 2 : Configurer le Vault

1. Remplissez les champs:
   - **Nom**: myRecoveryVault
   - **Groupe de ressources**: Sélectionnez le groupe existant
   - **Région**: Même région que votre VM

2. Cliquez sur **"Examiner + créer"**
3. Cliquez sur **"Créer"**
4. ⏳ Attendez 2-3 minutes

---

### Tâche 3 : Accéder au Vault

1. Allez à votre **Vault de Récupération**
2. Dans le menu de gauche, cliquez sur **"Site Recovery"**
3. Cliquez sur **"Activer la réplication"**

---

### Tâche 4 : Configurer la Source

1. **Source**: 
   - **Région source**: Votre région actuelle (eastus)
   - **Modèle de déploiement**: Resource Manager

2. Cliquez sur **"Suivant"**

---

### Tâche 5 : Sélectionner la VM Source

1. Cliquez sur **"Sélectionner les machines virtuelles"**
2. Cochez votre VM (ex: myVM)
3. Cliquez sur **"OK"**

---

### Tâche 6 : Configurer la Destination

1. **Région cible**: westus
2. **Groupe de ressources cible**: Créer nouveau ou sélectionner
3. **Réseau virtuel cible**: Créer nouveau
4. **Compte de stockage**: Créer nouveau

---

### Tâche 7 : Vérifier les Paramètres de Réplication

1. Cliquez sur **"Paramètres de réplication"**

2. Conservez les valeurs par défaut

---

### Tâche 8 : Activer la Réplication

1. Cliquez sur **"Activer la réplication"**
2. ⏳ La réplication commence (5-10 minutes)
3. Vous verrez l'état passer à "En cours"

---

### Tâche 9 : Monitorer la Réplication

1. Retournez au **Vault**
2. Cliquez sur **"Éléments protégés"**
3. Vérifiez l'état de votre VM
4. Attendez que le statut passe à **"Protégé"**

---

### Tâche 10 : Vérifier les Points de Récupération

1. Cliquez sur votre **VM protégée**
2. Allez à **"Points de récupération"**
3. Vous verrez les snapshots créés

---


### Tâche 11 : Créer un Plan de Récupération

1. Entrez le **Nom**: myRecoveryPlan
2. Sélectionnez votre **VM**
3. Cliquez sur **"Créer"**

---

### Tâche 12 : Tester le Failover

1. Cliquez sur votre **plan**
2. Cliquez sur **"Basculement de test"**
3. Sélectionnez le **point de récupération**
4. Sélectionnez le **réseau** cible
5. Cliquez sur **"OK"**

---

### Tâche 13 : Vérifier le Failover de Test

1. Allez à votre **groupe de ressources cible** (westus)
2. Vous verrez une **VM de test** créée
3. La VM de test est une copie complète de votre VM

---

### Tâche 14 : Nettoyer le Failover de Test

1. Retournez au **Vault**
2. Cliquez sur **"Nettoyer le basculement de test"**
3. Cliquez sur **"Supprimer les machines de test"**
4. Cliquez sur **"Terminer"**

---

### Tâche 15 : Désactiver la Réplication (Optionnel)

1. Allez à **"Éléments protégés"**
2. Cliquez sur votre **VM**
3. Cliquez sur **"Désactiver la réplication"**
4. Cliquez sur **"Supprimer"**

---

## Résumé des Apprentissages

| Concept | Description |
|---------|-------------|
| **Site Recovery** | Service Azure pour la récupération après sinistre |
| **RTO** | Temps maximum acceptable pour restaurer un service |
| **RPO** | Quantité de données qu'on peut perdre |
| **Vault** | Conteneur pour gérer la réplication |
| **Failover** | Basculement vers la région secondaire |
| **Failover de test** | Test sans affecter le système en production |
| **Points de récupération** | Snapshots des données à un moment donné |
| **Réplication asynchrone** | Les données sont répliquées en continu |

---

## Questions de Révision

1. Quelle est la différence entre RTO et RPO?
2. Pourquoi faire un failover de test?
3. Comment vérifier l'état de la réplication?
4. Quelles ressources sont créées dans la région cible?
5. Comment nettoyer les ressources après le lab?

---

## Dépannage Courant

| Problème | Solution |
|---------|----------|
| **Réplication ne démarre pas** | Vérifiez les permissions et la connectivité réseau |
| **VM de test ne se crée pas** | Attendez que la première réplication soit complète |
| **Failover échoue** | Vérifiez les ressources cibles et l'espace disque |
| **Points de récupération manquants** | Attendez le prochain cycle de réplication |

---

## Ressources Supplémentaires

- [Azure Site Recovery Documentation](https://learn.microsoft.com/fr-fr/azure/site-recovery/)
- [RTO et RPO Guide](https://learn.microsoft.com/fr-fr/azure/site-recovery/site-recovery-overview)
- [Disaster Recovery Planning](https://learn.microsoft.com/fr-fr/azure/architecture/framework/resiliency/disaster-recovery)

---

**Dernière mise à jour**: Décembre 2025
