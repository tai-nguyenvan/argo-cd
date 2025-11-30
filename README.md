Purpose

This repository contains Argo CD applications and Kubernetes manifests used for a local demo cluster.

Note: I created this README from the project `REAME.md` and removed any inline commands that contained secrets or sensitive tokens. Do NOT commit tokens or passwords into the repo.

Why you might not be able to push

Common reasons you cannot push to GitHub:
- Local network/DNS cannot reach github.com ("Could not resolve host" or timeouts).
- HTTPS authentication requires a Personal Access Token (PAT) instead of a password.
- SSH authentication missing or not configured ("Permission denied (publickey)").
- Remote URL is incorrect or you don't have write access to the target repo.

Quick checklist (run these on your machine)

1) Local repo status and remotes

```bash
cd /home/nvtai/workspaces/argo-cd
git status --porcelain --branch
git remote -v
```

2) Network checks (diagnose DNS / HTTP reachability)

```bash
# DNS
dig +short github.com || nslookup github.com

# HTTP
curl -I --max-time 10 https://github.com || curl -I --max-time 10 https://api.github.com
```

If these fail with DNS/timeouts, try switching networks (mobile tether) or temporarily change DNS to 8.8.8.8 in /etc/resolv.conf to verify.

3) Verbose debug push (captures the exact error)

- For HTTPS remotes:

```bash
GIT_TRACE=1 GIT_CURL_VERBOSE=1 git push origin HEAD
```

- For SSH remotes:

```bash
GIT_SSH_COMMAND="ssh -vvv" GIT_TRACE=1 git push origin HEAD
```

Paste the output here if you need help interpreting it.

How to fix common failures

A) "Could not resolve host" or timeout
- Fix local DNS or network first (change DNS to 8.8.8.8 or use a different network).
- If corporate proxy/firewall blocks outbound 443/53, either use an allowed network or configure the proxy.

B) HTTPS Authentication (HTTP 403 / authentication failed)
- GitHub removed password auth for HTTPS. Use a Personal Access Token (PAT) with repo scope.
- Create a PAT at https://github.com/settings/tokens (classic tokens or fine-grained tokens) and use it when prompted.
- Example (don’t store token in shell history):

```bash
# temporary one-off push (do not keep token in URL long-term)
git remote set-url origin https://<USERNAME>:<PAT>@github.com/<USER>/<REPO>.git
git push origin HEAD
# then reset remote url back to avoid token leakage
git remote set-url origin https://github.com/<USER>/<REPO>.git
```

Or configure credential helper and enter the PAT when prompted:

```bash
git config --global credential.helper store
git push origin HEAD
# enter USERNAME and PAT at prompt
```

C) SSH Authentication (Permission denied / no publickey)
- Generate SSH key and add to GitHub:

```bash
ssh-keygen -t ed25519 -C "nguyen.van.tai@outlook.com"
cat ~/.ssh/id_ed25519.pub  # copy contents
# then add at https://github.com/settings/ssh/new
ssh -T git@github.com
```
```bash
git remote set-url origin git@github.com:<USER>/<REPO>.git
git push origin HEAD
```

D) Repository not found / permission denied
- Ensure the remote URL is correct and that your account has write permission on the target repository.
- If pushing to an org repo, verify you are a collaborator or team member with push permissions.

If DNS/Network errors persist for GitHub specifically
- Try running the git push from a different network to rule out a local firewall or ISP block.
- Use SSH as a fallback if HTTPS is blocked by a proxy but SSH port is allowed.

Emergency troubleshooting commands to paste here

If you're still stuck, run these and paste the outputs:

```bash
# repo status + remotes
git status --porcelain --branch
git remote -v

# network
dig +short github.com
curl -I --max-time 10 https://github.com

# verbose push (HTTPS)
GIT_TRACE=1 GIT_CURL_VERBOSE=1 git push origin HEAD

# or verbose push (SSH)
GIT_SSH_COMMAND="ssh -vvv" GIT_TRACE=1 git push origin HEAD
```

What I changed in this workspace
- Created `README.md` (cleaned up and removed secrets). The original `REAME.md` (typo) remains unchanged.

Next steps I can do for you
- Help interpret the debug push output if you paste it here.
- Walk you through creating a PAT and performing a secure HTTPS push.
- Walk you through generating and installing SSH keys and switching remotes to SSH.
- If the problem is cluster-side (Argo CD can't fetch your repo), I can continue with the kube-proxy install to fix Service IP routing inside the cluster. (I previously created a temporary DaemonSet to restore masquerade.)

If you want me to proceed with one specific fix now, say one of:
- "Help interpret debug output" (paste output) 
- "Create PAT and push guide" 
- "Help configure SSH keys"
- "Apply kube-proxy fix in cluster"

Thank you — paste the failing output or tell me which action you want next.

