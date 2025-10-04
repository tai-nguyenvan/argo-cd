kubectl create namespace argocd
helm install argo-cd charts/argo-cd/ --namespace argocd 
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
HCPKe0aJ8cV96mS1clear