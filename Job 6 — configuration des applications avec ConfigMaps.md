vérifie la ConfigMap et le déploiement
```bash
sudo k3s kubectl get configmap -n job2
sudo k3s kubectl describe configmap app-config -n job2
sudo k3s kubectl get pods -n job2 -o wide
sudo k3s kubectl describe pod -n job2 -l app=nginx
```
![[Pasted image 20260611160617.png]]![[Pasted image 20260611160634.png]]
![[Pasted image 20260611160652.png]]![[Pasted image 20260611160715.png]]