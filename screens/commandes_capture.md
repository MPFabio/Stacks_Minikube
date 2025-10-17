# Les 8 captures d'écran requises pour le TP1

## 1. État initial du déploiement
```bash
kubectl get deploy,rs,pods,svc -n demo -o wide
```

## 2. Vue k9s (pods/services) - SI DISPONIBLE
```bash
k9s -n demo
# Puis capture de l'interface k9s
```

## 3. Page Front (navigateur) - Port-forward
```bash
# Option 1: Script automatique (recommandé)
.\port-forward-auto.ps1

# Option 2: Port manuel (trouver un port libre)
kubectl -n demo port-forward svc/front 8084:80
# Puis ouvrir http://localhost:8084 dans le navigateur
```

## 4. Appel API via Service (DNS interne)
```bash
kubectl -n demo exec deploy/front -- wget -qO- http://api:80/
```

## 5. État après modification (ressources)
```bash
kubectl -n demo get pods -o wide
```

## 6. Détails du pod avec ressources appliquées
```bash
kubectl -n demo describe pod api-664f7967c9-59t2s
```

## 7. Rollout status après modification
```bash
kubectl -n demo rollout status deploy/api
```

## 8. Vérification des services et ports
```bash
kubectl -n demo get services -o wide
```

