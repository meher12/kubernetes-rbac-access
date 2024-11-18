# RBAC (Role-Based Access Control) dans Kubernetes.
---
## Mise en Oeuvre de RBAC pour les Pods:

1. **Namespace** : Nous créons d'abord un namespace `dev-team` pour isoler nos ressources.

2. **ServiceAccount** : Nous créons un compte de service `dev-user` qui sera utilisé pour l'authentification.

3. **Role** : Nous définissons un rôle `pod-reader` qui spécifie les permissions :
   - Accès aux Pods (`get`, `list`, `watch`):
     Ce bloc donne les permissions pour accéder aux pods (lister, get, regarder).
   ```
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "list", "watch"]
   ```
   - Accès aux Logs des Pods (`get` sur `pods/logs`):
     Ce bloc donne la permission d'accéder aux logs des pods (obtenir les logs).
   ```
   - apiGroups: [""]
     resources: ["pods/logs"]
     verbs: ["get"]
   ```
   - Exécution de commandes dans les pods (`create` sur `pods/exec`):
     Ce bloc donne la permission d'exécuter des commandes dans les pods (création d'exécution)
   ```
   - apiGroups: [""]
     resources: ["pods/exec"]
     verbs: ["create"]
   ```

4. **RoleBinding** : Nous lions le rôle au ServiceAccount via un RoleBinding.

5. **Nginx Pod** : Un pod simple pour tester les permissions.

Pour utiliser cet exemple :

```bash
# Appliquer la configuration
kubectl apply -f namespace.yaml  -f service-account.yaml  -f role.yaml  -f role-binding.yaml -f nginx-pod.yaml

# Obtenir le token du ServiceAccount (pour un cluster récent)
kubectl create token dev-user -n dev-team

# Tester les permissions
# Avec le token obtenu, configurez votre kubectl ou testez directement :
kubectl --token=<token> --namespace=dev-team get pods
kubectl --token=<token> --namespace=dev-team logs test-pod
kubectl --token=<token> --namespace=dev-team exec -it test-pod -- /bin/bash
```

Quelques points importants à noter :

1. Les `verbs` définissent les actions autorisées :
   - `get` : lecture d'une ressource spécifique
   - `list` : liste des ressources
   - `watch` : surveillance des changements
   - `create` : création de ressources
   - `update` : modification
   - `delete` : suppression
   - `patch` : modifications partielles

2. Cette configuration est restrictive : l'utilisateur ne peut que :
   - Voir les pods
   - Lire leurs logs
   - Exécuter des commandes dans les pods
---
## Configuration RBAC avec ClusterRole:
### Créeation d'un ClusterRole avec des permissions à l'échelle du cluster.
1. **ServiceAccount** : 
   - Créé dans le namespace `kube-system`
   - Utilisé pour l'authentification à l'échelle du cluster

2. **ClusterRole** :
   - Différent d'un Role standard car il s'applique à tous les namespaces
   - Permissions définies :
     - Lecture des pods et leurs logs partout
     - Surveillance des nodes
     - Gestion complète des deployments
     - Création et lecture des services
     - Lecture des événements
     - Lecture des configmaps et secrets
     - Lecture des namespaces

3. **ClusterRoleBinding** :
   - Lie le ClusterRole au ServiceAccount
   - S'applique globalement dans le cluster

Pour utiliser cette configuration :

```bash
# Appliquer la configuration
kubectl apply -f serviceAccount.yaml -f cluster-role.yaml  -f clusterRoleBinding.yaml -f bitnami-pod.yaml

# Obtenir le token du ServiceAccount
kubectl create token cluster-admin-user -n kube-system

# Tester les permissions (exemples)
kubectl --token=<token> get pods --all-namespaces
kubectl --token=<token> get nodes
kubectl --token=<token> get deployments --all-namespaces
```

Points importants à noter :

1. **Sécurité** :
   - Ces permissions sont puissantes, à utiliser avec précaution
   - Suivez le principe du moindre privilège
   - Évitez de donner des permissions `delete` sauf si nécessaire

2. **Bonnes pratiques** :
   - Groupez les permissions logiquement par ressource
   - Documentez pourquoi chaque permission est nécessaire
   - Révisez régulièrement les permissions accordées

3. **Différences avec les Roles** :
   - ClusterRole : permissions globales sur le cluster
   - Role : permissions limitées à un namespace
