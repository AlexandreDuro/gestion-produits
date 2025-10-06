# Livrable Kubernetes - Partie 2

## 1. Préparation du cluster Kubernetes

### Cluster utilisé
- **Type** : Minikube (cluster local)
- **Version Minikube** : v1.37.0
- **Version Kubernetes** : v1.34.0
- **Driver** : Docker

### Configuration kubectl
```bash
# Démarrage du cluster
minikube start

# Vérification de la configuration
kubectl get nodes
```

**Résultat** :
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   15d   v1.34.0
```

---

## 2. Fichiers de déploiement

### Structure des fichiers Kubernetes

```
k8s/
├── mysql-deployment.yaml    # Deployment MySQL + PVC
├── mysql-service.yaml        # Service MySQL (ClusterIP)
├── php-deployment.yaml       # Deployment PHP (3 replicas)
└── php-service.yaml          # Service PHP (NodePort)
```

### mysql-deployment.yaml
- **PersistentVolumeClaim** : 5Gi avec accessMode `ReadWriteOnce`
- **Deployment** : 1 replica MySQL
- **Image** : `alexandreduro/gestion-produits-mysql:v1.0`
- **Volume persistant** monté sur `/var/lib/mysql`

### mysql-service.yaml
- **Type** : ClusterIP (communication interne)
- **Port** : 3306

### php-deployment.yaml
- **Replicas** : 3 (pour la scalabilité)
- **Image** : `alexandreduro/gestion-produits-php:v1.0`
- **Variables d'environnement** :
  - `DB_HOST=mysql-service`
  - `DB_PASSWORD=root`

### php-service.yaml
- **Type** : NodePort (exposition externe)
- **Port** : 80
- **NodePort** : 30080

---

## 3. Commandes de déploiement

### Déploiement de MySQL
```bash
kubectl apply -f k8s/mysql-deployment.yaml
kubectl apply -f k8s/mysql-service.yaml
```

### Déploiement de l'application PHP
```bash
kubectl apply -f k8s/php-deployment.yaml
kubectl apply -f k8s/php-service.yaml
```

### Vérification du déploiement
```bash
# Vérifier les pods
kubectl get pods

# Vérifier les déploiements
kubectl get deployments

# Vérifier les services
kubectl get services

# Vérifier le stockage persistant
kubectl get pvc
```

---

## 4. Scalabilité - Preuve de déploiement

### État des pods (3 replicas PHP)
```
NAME                                   READY   STATUS    RESTARTS   AGE   IP
pod/mysql-deployment-b658b5844-d55fh   1/1     Running   0          46m   10.244.0.16
pod/php-deployment-fc4dc554-5xqdw      1/1     Running   0          45m   10.244.0.18
pod/php-deployment-fc4dc554-755wc      1/1     Running   0          46m   10.244.0.17
pod/php-deployment-fc4dc554-t2qn9      1/1     Running   0          45m   10.244.0.19
```

### État des déploiements
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES
mysql-deployment   1/1     1            1           15d   mysql        alexandreduro/gestion-produits-mysql:v1.0
php-deployment     3/3     3            3           15d   php-app      alexandreduro/gestion-produits-php:v1.0
```

### État des services
```
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP        15d
mysql-service   ClusterIP   10.98.166.207    <none>        3306/TCP       15d
php-service     NodePort    10.108.248.241   <none>        80:30080/TCP   15d
```

### Stockage persistant
```
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
mysql-pv-claim   Bound    pvc-1a8fc180-bbe4-4f1b-895c-bd2dc74256c1   5Gi        RWO            standard
```

---

## 5. Test du Load Balancing

### Accès à l'application

**Option 1 : Via port-forward (recommandé pour Minikube sur macOS)**
```bash
kubectl port-forward service/php-service 8090:80
```
Puis accéder à : http://localhost:8090

**Option 2 : Via NodePort**
```bash
# Obtenir l'IP Minikube
minikube ip

# Accéder via NodePort
curl http://$(minikube ip):30080
```

**Option 3 : Via Minikube tunnel**
```bash
minikube service php-service --url
```

### Preuve du Load Balancing

Les 3 replicas PHP sont actifs et le service `php-service` de type NodePort distribue automatiquement le trafic entre les 3 pods :

```
php-deployment-fc4dc554-5xqdw   1/1   Running   IP: 10.244.0.18
php-deployment-fc4dc554-755wc   1/1   Running   IP: 10.244.0.17
php-deployment-fc4dc554-t2qn9   1/1   Running   IP: 10.244.0.19
```

Le service utilise le **selector `app: php-app`** qui cible automatiquement les 3 pods, assurant ainsi la distribution de charge.

### Commande de scaling manuel (si besoin)
```bash
# Augmenter à 5 replicas
kubectl scale deployment php-deployment --replicas=5

# Revenir à 3 replicas
kubectl scale deployment php-deployment --replicas=3
```

---

## 6. Résumé

✅ **Cluster Kubernetes** : Minikube configuré et opérationnel
✅ **Stockage persistant** : PVC de 5Gi pour MySQL
✅ **Déploiements** : MySQL (1 replica) + PHP (3 replicas)
✅ **Services** : ClusterIP pour MySQL, NodePort pour PHP
✅ **Scalabilité** : 3 replicas PHP actifs avec load balancing automatique
✅ **Application fonctionnelle** : Accessible via port-forward ou NodePort

---

## 7. Commandes utiles

```bash
# Voir les logs d'un pod
kubectl logs <pod-name>

# Accéder à un pod en mode shell
kubectl exec -it <pod-name> -- /bin/bash

# Supprimer tous les déploiements
kubectl delete -f k8s/

# Redémarrer un deployment
kubectl rollout restart deployment/php-deployment

# Voir l'historique des rollouts
kubectl rollout history deployment/php-deployment
```
