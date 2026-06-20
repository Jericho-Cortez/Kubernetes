Sécurité

- Ne pas committer de secrets en clair.
- Pour MariaDB: créer le secret avec `kubectl create secret generic mariadb-secret --from-literal=root-password=CHANGEME -n job2` ou utiliser SealedSecrets/External Secrets.
- Les PV hostPath conviennent au lab uniquement; pour prod utiliser StorageClass réseau (NFS, Ceph, cloud).
