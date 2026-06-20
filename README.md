# Kubernetes K3s Project

Ce dépôt contient des exercices et manifests pour déployer un petit cluster K3s et des applications (nginx, apache, mariadb) à des fins pédagogiques.

Principaux contenus
- manifests/: manifests YAML extraits des notes (job1..job9)
- .github/workflows/ci.yml: CI basique (yamllint)
- SECURITY.md: recommandations de sécurité

Avant de publier
- Ne committer aucun secret en clair. Utiliser `kubectl create secret` ou SealedSecrets.
- Vérifier et adapter StorageClass et PV pour l'environnement cible.

Déploiement rapide (lab)
1. kubectl apply -f manifests/job2
2. kubectl get pods -n job2

Pour production
- Convertir MariaDB en StatefulSet, utiliser un StorageClass réseau, ajouter probes, limits, NetworkPolicy, monitoring et backups.
