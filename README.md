# Stacks Minikube

Stack Kubernetes minimale pour le TP1 (Minikube). On déploie un echo-server côté API et un frontend nginx exposé en NodePort, le tout dans le namespace `demo`. Ce dépôt conserve également les réponses théoriques (`RENDU_TP1.md`), un mémo de commandes (`kubectl_memo.txt`) et les captures d'écran réalisées pendant le TP (`screens/`).

## Contenu principal
- `manifests/00-namespace.yaml` : création du namespace `demo`.
- `manifests/10-deploy-api.yaml` : déploiement de l'API `ealen/echo-server` avec probes et requests/limits.
- `manifests/20-svc-api-clusterip.yaml` : service interne pour exposer l'API dans le cluster.
- `manifests/30-deploy-front.yaml` : déploiement nginx servant de client et de page d'accueil.
- `manifests/40-svc-front-nodeport.yaml` : NodePort (30080) pour accéder au front depuis la machine hôte.
- `kubectl_memo.txt` : rappel des principales commandes kubectl.
- `RENDU_TP1.md` : réponses aux questions du TP et détails sur la modification apportée (ajout de ressources sur l'API).

## Prérequis
- Minikube (ou un cluster Kubernetes équivalent) démarré.
- `kubectl` configuré sur le contexte du cluster.
- Optionnel : `k9s` pour la visualisation.

## Déploiement rapide
```bash
minikube start               # si ce n'est pas déjà fait
kubectl apply -f manifests/  # crée le namespace, les deployments et les services
```

## Vérifications utiles
```bash
kubectl get ns
kubectl get deploy,rs,pods,svc -n demo -o wide
kubectl describe svc api -n demo
kubectl get endpointslices -n demo
```

### Tester l'API depuis le front
```bash
kubectl -n demo exec deploy/front -- wget -qO- http://api:80/
```

### Accéder au front
- NodePort : `minikube service front -n demo --url` (ou `http://$(minikube ip):30080`).
- Port-forward alternatif : `kubectl -n demo port-forward svc/front 8080:80` puis `http://localhost:8080`.

### Suivre le rollout après modification
```bash
kubectl -n demo rollout status deploy/api
kubectl -n demo describe pod -l app=api
```

## Nettoyage
```bash
kubectl delete ns demo
minikube stop   # si vous souhaitez éteindre le cluster
```

## Ressources complémentaires
- `RENDU_TP1.md` détaille les réponses théoriques (scheduling, services, probes, DNS, NodePort) ainsi que le rationnel des requests/limits.
- `screens/` contient toutes les captures mentionnées dans le rendu (état initial, rollout, tests DNS, etc.).
- `kubectl_memo.txt` liste les commandes exactes utilisées pendant le TP.
-  liste les commandes exactes utilisées pendant le TP.
