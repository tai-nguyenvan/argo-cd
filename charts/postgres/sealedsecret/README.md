This folder contains a template for a SealedSecret and instructions to create a real SealedSecret locally.

Why use SealedSecrets
- Keeps secrets encrypted in Git.
- Controller in-cluster (installed) will decrypt SealedSecret into a Kubernetes Secret.

Prerequisites (on your workstation)
- kubectl configured to your cluster (same cluster where Sealed Secrets controller is installed).
- kubeseal CLI installed. Install instructions: https://github.com/bitnami-labs/sealed-secrets#installation

Steps (copy-paste)
1) Create a plain secret locally (dry-run) and pipe to kubeseal to produce a SealedSecret manifest:

```bash
kubectl create secret generic postgres-creds \
  --dry-run=client \
  --from-literal=POSTGRES_USER=admin \
  --from-literal=POSTGRES_PASSWORD='pass@123' \
  --from-literal=POSTGRES_DB=postgres \
  -o yaml \
  | kubeseal --format yaml > charts/postgres/sealedsecret/postgres-sealedsecret.yaml
```

Notes:
- The `kubeseal` command will fetch the SealedSecrets controller public key from the cluster and use it to encrypt values.
- The output file `charts/postgres/sealedsecret/postgres-sealedsecret.yaml` is safe to commit to Git (it is encrypted).

2) Inspect the generated sealed secret (ensure metadata.name=postgres-creds and namespace=development):

```bash
sed -n '1,120p' charts/postgres/sealedsecret/postgres-sealedsecret.yaml
```

3) Commit & push to your repo (SSH recommended):

```bash
# if remote is HTTPS and you prefer SSH, switch remote URL (1-time)
git remote set-url origin git@github.com:tai-nguyenvan/argo-cd.git

git add charts/postgres/sealedsecret/postgres-sealedsecret.yaml
git commit -m "chore(secrets): add sealedsecret for postgres creds"
git push origin main
```

4) Enable your Argo CD Application to reference the sealed secret (example for `postgres-blue`):
- Edit `charts/postgres/postgres-application-blue.yaml` in the repo to set Helm values enabling secret usage, e.g. add under `spec.source.helm.values`:

```yaml
fullnameOverride: postgres-blue
credentials:
  secret:
    enabled: true
    name: postgres-creds
```

Then commit & push the change. Argo CD will sync and the SealedSecret will be unsealed into a Secret by the controller.

5) Force Argo CD to refresh the parent app (optional immediate):

```bash
kubectl -n argocd annotate application postgres argocd.argoproj.io/refresh=hard --overwrite
kubectl -n argocd get applications -o custom-columns=NAME:.metadata.name,HEALTH:.status.health.status,SYNC:.status.sync.status
```

Verify in-cluster Secret (created by controller):

```bash
kubectl -n development get secret postgres-creds -o yaml
kubectl -n development get secret postgres-creds -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 --decode && echo
```

Cleanup guidance
- If you ever need to remove the sealed secret from Git, simply `git rm` and push. The controller will not remove existing unsealed Secrets automatically.

If you prefer, I can prepare the sealedsecret file for you (it must be generated with kubeseal on a machine with access to the controller) — I will provide the exact command above for you to run locally and then paste the generated file here or push to the repo. After you push, tell me and I'll refresh the Argo CD parent app and verify everything is Synced.

