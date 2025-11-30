apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: forgejo
  namespace: argocd
spec:
  project: development
  source:
    repoURL: 'https://github.com/tai-nguyenvan/argo-cd.git'
    targetRevision: main
    path: modules/forgejo
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: development
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
# Write updated values to `modules/forgejo/values.yaml`
cat > modules/forgejo/values.yaml <<'YAML'
forgejo:
  app:
    environment: production
  image:
    repository: docker.io/forgejo/forgejo
    name: forgejo
    tag: 13.0.3
    pullPolicy: IfNotPresent

fullnameOverride: forgejo-blue
namespaceOverride: development
serviceNameOverride: forgejo-pool
YAML

# Create ArgoCD Application manifest at `argocd/forgejo-application.yaml`
mkdir -p argocd
cat > argocd/forgejo-application.yaml <<'YAML'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: forgejo
  namespace: argocd
spec:
  project: development
  source:
    repoURL: 'https://github.com/tai-nguyenvan/argo-cd.git'
    targetRevision: main
    path: modules/forgejo
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: development
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
YAML

# Optional: test install locally with Helm
# helm upgrade --install forgejo modules/forgejo \
#   --namespace development --create-namespace \
#   -f modules/forgejo/values.yaml

# Apply the ArgoCD Application (ArgoCD must be installed in cluster)
# kubectl apply -f argocd/forgejo-application.yaml -n argocd

curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.9.3/clusterctl-linux-amd64 -o clusterctl

sudo install -o root -g root -m 0755 clusterctl /usr/local/bin/clusterctl

sudo snap install k8s --classic --channel=1.34-classic/stable

sudo k8s bootstrap

sudo k8s status --wait-ready

mkdir -p ~/.kube/

sudo k8s config > ~/.kube/config

sudo k8s kubectl create namespace argocd

helm install argo-cd charts/argo-cd/ --namespace argocd

kubectl get pods -n argocd -w

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

# REDACTED_ARGOCD_INITIAL_ADMIN_PASSWORD

kubectl port-forward svc/argo-cd-argocd-server -n argocd 8080:443

# argocd login localhost:8080

sudo nano /etc/systemd/system/k8s-argo-forward.service

sudo systemctl daemon-reload

sudo systemctl enable k8s-argo-forward.service

sudo systemctl start k8s-argo-forward.service

sudo systemctl status k8s-argo-forward.service

sudo journalctl -u k8s-argo-forward.service -f

sudo reboot now

# REDACTED_GITHUB_TOKEN


kubectl apply -f modules/postgresql/values.yaml -n argocd


kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: forgejo
  namespace: argocd
spec:
  project: development
  source:
    repoURL: 'https://github.com/tai-nguyenvan/argo-cd'
    targetRevision: main
    path: manifest
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: development
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    sysOptions:
      CreateNamespace=true
EOF

