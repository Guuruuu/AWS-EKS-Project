# Kubernetes Milestone 5

This folder contains Kubernetes manifests used to deploy an application in an EKS cluster.

Resources created:

- Namespace
- ConfigMap
- Secret
- Deployment
- LoadBalancer Service

---

## Milestone 5 - Kubernetes App Deployment Summary

### What Was Done
Deployed a full Kubernetes application onto the EKS cluster by applying 5 manifest files in order:

1. **Namespace** — created an isolated environment called `demo`
2. **Secret** — securely stored app credentials
3. **ConfigMap** — stored app configuration settings
4. **Deployment** — launched the app across 2 running pods
5. **Load Balancer** — exposed the app publicly to the internet

---

### Obstacles Encountered

### Obstacle 1: Git Clone Permission Denied

**Problem:** When attempting to clone the project repository, the following error appeared:
```
fatal: could not create work tree dir 'AWS-EKS-Project': Permission denied
```
The SSM user did not have write permission in the current directory.

**Solution:** Navigated to the home directory first where the SSM user has full permissions, then re-ran the clone:
```bash
cd ~
git clone https://github.com/Guuruuu/AWS-EKS-Project.git
```

---

### Obstacle 2: "Already Exists" Error on Second Clone Attempt

**Problem:** After fixing the permission issue, the clone command was run again resulting in:
```
fatal: destination path 'AWS-EKS-Project' already exists and is not an empty directory
```

**Solution:** Recognized that the first clone had actually succeeded. Skipped the clone and moved directly to the next step.

---

### Obstacle 3: `kubectl apply -f .` Failing

**Problem:** Running the command in the project folder failed because `README.md` was in the same folder and kubectl can't read it — only `.yaml/.yml/.json` files are accepted.

**Solution:** Navigated into the specific Part 5 folder and applied each YAML file individually in the correct order:
```bash
cd ~/AWS-EKS-Project/part\ 5\ -\ Kubernetes\ App\ Deployment
kubectl apply -f namespace_manifest.yaml
kubectl apply -f Secret_manifest.yaml
kubectl apply -f configmap_manifest.yaml
kubectl apply -f deployment_mainfest.yaml
kubectl apply -f loadbalancer_manifest.yaml
```

---

### Recurring Warning: `error reading [.]`

**What it was:** A recurring but harmless warning that appeared multiple times throughout the session when running `aws eks update-kubeconfig`.

**Reality:** This never blocked progress — the EKS kubeconfig updated successfully each time despite the warning appearing.
