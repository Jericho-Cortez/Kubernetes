Créer le secret MariaDB (ne pas committer la valeur) :

kubectl create secret generic mariadb-secret --from-literal=root-password='CHANGE_ME' -n job2

Pour CI/infra, utiliser SealedSecrets ou ExternalSecrets pour stocker les secrets en dehors du dépôt.
