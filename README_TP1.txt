TP1 — Comprendre, tester, ajuster (Kubernetes) 
=================================================================


Objectif
--------
1) Déployer un mini-stack **Front + API** fourni (manifests prêts à l’emploi).
2) **Comprendre** et **expliquer** le fonctionnement (control plane, Services, probes, DNS interne).
3) Réaliser **une modification légère** (au choix) et **justifier** vos choix.
4) Produire un **rendu structuré** avec **captures** et **commandes**.

Prérequis & configuration
-------------------------
- OS : Linux, macOS, Windows.
- Outils : `kubectl` (≥1.27), **k3d** *ou* **kind**, navigateur web, (optionnel) **k9s**.
- Réseau : port NodePort **30080** libre sur votre machine (à modifier si occupé).
- Clonage : placez ce dossier dans un répertoire court (sans espaces).
- Choisissez un moteur de cluster local :
  - **k3d** : `k3d cluster create s3 --agents 1`
  - **kind** : `kind create cluster --name s3`
- Contexte : vérifiez `kubectl config current-context` puis `kubectl cluster-info`.

Contenu du pack
---------------
- `manifests/`
  - `00-namespace.yaml` — Namespace `demo`
  - `10-deploy-api.yaml` — Deployment `api` (3 réplicas) basé sur **ealen/echo-server**
  - `20-svc-api-clusterip.yaml` — Service `api` en **ClusterIP** (80 → 80)
  - `30-deploy-front.yaml` — Deployment `front` (NGINX)
  - `40-svc-front-nodeport.yaml` — Service `front` en **NodePort** (30080)
- `kubectl_memo.txt` — commandes utiles

Étapes (séquence conseillée)
----------------------------
1) Appliquez tous les manifests :
   ```bash
   kubectl apply -f manifests/
   ```
2) Vérifiez l’état :
   ```bash
   kubectl get deploy,rs,pods,svc -n demo -o wide
   ```
3) Accès front (NodePort) : ouvrez **http://<NodeIP>:30080**
   - Avec **k3d** : <NodeIP> est souvent `127.0.0.1`.
   - Avec **kind** : récupérez l’IP d’un nœud (`kubectl get nodes -o wide`) ou utilisez un port-forward :
     `kubectl -n demo port-forward svc/front 8080:80` puis http://localhost:8080
4) DNS interne : depuis un Pod, testez l’API via le **Service** interne
   ```bash
   kubectl -n demo exec deploy/front -- wget -qO- http://api:80/
   ```
   (Vous devriez voir une réponse JSON générée par **ealen/echo-server**.)

Partie A — Compréhension & exploration (réponses courtes + captures)
--------------------------------------------------------------------
- **Q1. Scheduling** (4–6 lignes) : décrivez la chaîne **apiserver → etcd → controllers → scheduler → kubelet** suite à `kubectl apply`.
- **Q2. Service/Endpoints** : comment un **Service ClusterIP** associe-t-il ses **backends** ? Parlez de **labels/selectors** et **EndpointSlice**.
- **Q3. Probes** : différence **readiness** vs **liveness**. Donnez un exemple de paramètres adaptés (période, timeout, échecs).
- **Q4. DNS interne** : pourquoi `http://api:80/` marche sans IP ? Donnez la forme **FQDN** complète.
- **Q5. NodePort** : atouts en local, limites en prod face à **Ingress**.

> **Captures attendues** : `kubectl get … -o wide`, vue k9s (pods/services), page Front (navigateur), appel API via Service.

Partie B — Modification légère (faites **UN** choix, + justification 2–4 lignes)
---------------------------------------------------------------------------------
1) **Probes** : ajoutez une `startupProbe` à l’API (calibrage réaliste) **et** justifiez vos seuils.
2) **Ressources** : ajoutez `resources.requests/limits` à l’API (au moins mémoire) **et** justifiez.
3) **Service** : changez le `nodePort` (ex. `31080`) **et** expliquez l’impact (collisions, firewall, doc).

Après modification :
```bash
kubectl apply -f manifests/
kubectl -n demo rollout status deploy/api
kubectl -n demo get pods -o wide
```

Livrables & format
------------------
- **README (PDF ou MD)** : réponses Q1–Q5 + justification de la modification choisie.
- **Captures** : regroupez-les dans un dossier `screens/` (k9s, navigateur, sorties kubectl).
- **Manifests modifiés** : uniquement les fichiers impactés.
- **Log de 5–10 commandes** réellement exécutées (copier/coller propre).

Barème (20 pts)
----------------
- Déploiement OK + vérifs de base ......................... **6 pts**
- Compréhension (Q1–Q5) ................................... **8 pts**
- Modification légère + justification ..................... **4 pts**
- Présentation du rendu (clarté, structure, captures) ..... **2 pts**

Nettoyage
---------
```bash
kubectl delete ns demo
# k3d :
k3d cluster delete s3
# kind :
kind delete cluster --name s3
```
