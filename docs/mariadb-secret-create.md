# Créer le Secret MariaDB

Ne jamais committer un mot de passe en clair dans le dépôt.
Créer le Secret directement sur le cluster avec la commande suivante :

```bash
kubectl create secret generic mariadb-secret \
  --from-literal=root-password='CHANGE_ME' \
  -n job2
```

Vérifier que le Secret est bien créé :

```bash
kubectl get secret mariadb-secret -n job2
kubectl describe secret mariadb-secret -n job2
```

## En production

Pour un environnement de production, ne pas utiliser `kubectl create secret` avec un mot de passe en clair dans l'historique bash.
Préférer :
- **SealedSecrets** : chiffrement côté dépôt Git, déchiffrement par un contrôleur dans le cluster.
- **ExternalSecrets** : synchronisation depuis un coffre externe (HashiCorp Vault, AWS Secrets Manager, etc.).
