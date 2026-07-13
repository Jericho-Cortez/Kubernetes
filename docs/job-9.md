# Job 09 — Helm : gestionnaire de paquets Kubernetes

## Objectif

Installer Helm sur le cluster K3s, puis gérer le cycle de vie complet d'une application :
recherche d'un chart, installation, personnalisation via `values.yaml`, mise à jour et désinstallation.

---

## 1. Installation de Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Vérifier l'installation :

```bash
helm version
```

Sur K3s, s'assurer que Helm utilise le bon kubeconfig :

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get nodes
```

---

## 2. Ajouter un dépôt de charts

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Rechercher un chart disponible :

```bash
helm search repo bitnami/nginx
```

---

## 3. Installer une application

```bash
helm install mon-nginx bitnami/nginx
```

Vérifier que la release est créée et que les pods tournent :

```bash
helm list
kubectl get pods
kubectl get svc
```

---

## 4. Personnaliser avec values.yaml

Créer un fichier de valeurs personnalisées (voir `manifests/job9/values-custom.yaml`) :

```yaml
replicaCount: 2
service:
  type: NodePort
  nodePorts:
    http: "30080"
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

Appliquer ces valeurs à l'installation :

```bash
helm install mon-nginx bitnami/nginx -f manifests/job9/values-custom.yaml
```

Ou inspecter les valeurs par défaut d'un chart avant de le modifier :

```bash
helm show values bitnami/nginx
```

---

## 5. Mettre à jour une release (upgrade)

Modifier les valeurs (ex. passer à 3 replicas) et lancer la mise à jour :

```bash
helm upgrade mon-nginx bitnami/nginx -f manifests/job9/values-custom.yaml
```

Vérifier l'historique des révisions :

```bash
helm history mon-nginx
```

---

## 6. Rollback

Revenir à la révision précédente si la mise à jour pose problème :

```bash
helm rollback mon-nginx 1
```

Vérifier que le rollback est bien effectué :

```bash
helm history mon-nginx
kubectl get pods
```

---

## 7. Désinstaller une release

```bash
helm uninstall mon-nginx
```

Vérifier que les ressources sont bien supprimées :

```bash
helm list
kubectl get pods
kubectl get svc
```

---

## Résumé du cycle de vie Helm

| Commande | Rôle |
|---|---|
| `helm repo add` | Ajouter un dépôt de charts |
| `helm repo update` | Mettre à jour l'index local |
| `helm search repo` | Rechercher un chart |
| `helm install` | Installer une release |
| `helm list` | Lister les releases installées |
| `helm upgrade` | Mettre à jour une release |
| `helm rollback` | Revenir à une révision précédente |
| `helm history` | Voir l'historique d'une release |
| `helm uninstall` | Supprimer une release |
