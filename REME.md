


JhYN3qgafGMXZD5l


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

TkIrSRSAwYYqQYyf

kubectl port-forward svc/argo-cd-argocd-server -n argocd 8080:443

argocd login localhost:8080

sudo nano /etc/systemd/system/k8s-argo-forward.service

sudo systemctl daemon-reload

sudo systemctl enable k8s-argo-forward.service

sudo systemctl start k8s-argo-forward.service

sudo systemctl status k8s-argo-forward.service

sudo journalctl -u k8s-argo-forward.service -f

sudo reboot now

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