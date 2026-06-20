## Objectif
Mettre en place du stockage persistant pour les applications du cluster, puis utiliser une ConfigMap et un Secret pour externaliser la configuration.

## 1. Vérifier le namespace

Toutes les ressources seront créées dans le namespace `job2` :

```bash
sudo k3s kubectl get ns
```

Si besoin :

```bash
sudo k3s kubectl create ns job2
```

## 2. Stockage persistant pour nginx

Créer un PersistentVolume et un PersistentVolumeClaim pour nginx.

### nginx-pv.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/nginx
```

### nginx-pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
  namespace: job2
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### nginx.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: job2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27-alpine
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-storage
              mountPath: /usr/share/nginx/html
      volumes:
        - name: nginx-storage
          persistentVolumeClaim:
            claimName: nginx-pvc
***
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: job2
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: NodePort
```

## 3. Stockage persistant pour MariaDB

### mariadb-pv.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/mariadb
```

### mariadb-pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pvc
  namespace: job2
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### mariadb.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  namespace: job2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
        - name: mariadb
          image: mariadb:10.11
          ports:
            - containerPort: 3306
          env:
            - name: MARIADB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mariadb-secret
                  key: root-password
          volumeMounts:
            - name: mariadb-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mariadb-storage
          persistentVolumeClaim:
            claimName: mariadb-pvc
***
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
  namespace: job2
spec:
  selector:
    app: mariadb
  ports:
    - port: 3306
      targetPort: 3306
  type: ClusterIP
```

## 4. ConfigMap et Secret

### configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: job2
data:
  index.html: |
    <html>
    <body>
    <h1>Bonjour depuis nginx avec ConfigMap</h1>
    </body>
    </html>
```

### secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-secret
  namespace: job2
type: Opaque
stringData:
  root-password: rootpass
```

## 5. Appliquer les fichiers

```bash
sudo k3s kubectl apply -f nginx-pv.yaml
sudo k3s kubectl apply -f nginx-pvc.yaml
sudo k3s kubectl apply -f configmap.yaml
sudo k3s kubectl apply -f secret.yaml
sudo k3s kubectl apply -f nginx.yaml
sudo k3s kubectl apply -f mariadb-pv.yaml
sudo k3s kubectl apply -f mariadb-pvc.yaml
sudo k3s kubectl apply -f mariadb.yaml
```

![[Pasted image 20260609175646.png]]
## 6. Vérifier le déploiement

```bash
sudo k3s kubectl get pv
sudo k3s kubectl get pvc -n job2
sudo k3s kubectl get pods -n job2 -o wide
sudo k3s kubectl get svc -n job2
sudo k3s kubectl get configmap -n job2
sudo k3s kubectl get secret -n job2
```

![[Pasted image 20260609175734.png]]
## 7. Vérifier la persistance

Supprimer un pod puis le recréer :

```bash
sudo k3s kubectl delete pod -n job2 <nom_du_pod>
sudo k3s kubectl get pods -n job2 -o wide
```

![[Pasted image 20260609174209.png]]

Les données doivent rester disponibles grâce au volume persistant.

Après suppression du pod MariaDB, Kubernetes a recréé automatiquement un nouveau pod. Le PVC `mariadb-pvc` reste en attente car le `StorageClass local-path` utilise `WaitForFirstConsumer`, ce qui retarde le binding jusqu’à la création du premier consommateur.