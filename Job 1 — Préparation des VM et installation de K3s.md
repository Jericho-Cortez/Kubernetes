## Objectif
Mettre en place 3 VM Debian sans interface graphique, leur attribuer un nom d’hôte unique, puis installer K3s sur chacune.

## Machines du projet
- kubes-01.local — 192.168.10.10
- kubes-02.local — 192.168.10.11
- kubes-03.local — 192.168.10.12

## 1. Changer le nom d’hôte

Sur chaque VM, configurer le hostname correspondant :

```bash
sudo hostnamectl set-hostname kubes-01.local
sudo hostnamectl set-hostname kubes-02.local
sudo hostnamectl set-hostname kubes-03.local
```

Vérifier ensuite avec :

```bash
hostnamectl
```

## 2. Vérifier la résolution locale

Sur chaque machine, vérifier que `/etc/hosts` contient bien le nom correct associé à l’IP locale :

```txt
127.0.0.1   localhost
127.0.1.1   kubes-01.local kubes-01
```

Adapter la ligne selon la VM concernée :
- kubes-02.local sur la VM 02
- kubes-03.local sur la VM 03

## 3. Installer K3s

Sur chaque machine, lancer l’installation de base :

```bash
curl -sfL https://get.k3s.io | sh -
```
![[Pasted image 20260609095051.png]]
## 4. Vérifier que K3s fonctionne

Contrôler l’état du service :

```bash
sudo systemctl status k3s
```

Puis vérifier les nœuds Kubernetes :

```bash
sudo k3s kubectl get nodes
```
![[Pasted image 20260609102352.png]]