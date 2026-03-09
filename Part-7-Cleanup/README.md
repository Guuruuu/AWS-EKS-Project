# Part 7 — Cleanup

This section covers the teardown and cleanup of all Kubernetes and AWS resources created throughout the project.

Resources deleted:

- Kubernetes Namespace
- EKS Cluster Stack
- Network Stack

---

## Milestone 7 - Cleanup Summary

### What Was Done

Tore down all project resources in the correct order to ensure no orphaned resources or dependency errors:

1. **Delete Kubernetes Namespace** — removed the `demo` namespace which automatically cleaned up all pods, services, secrets, and configmaps inside it
2. **Delete EKS Stack** — deleted the EKS CloudFormation stack which removed the cluster and worker nodes
3. **Delete Network Stack** — deleted the VPC and networking infrastructure created in Milestone 1

---

### Why Order Matters
Resources must be deleted in the correct sequence — Kubernetes namespace first, then EKS, then networking. Deleting out of order can cause dependency errors and leave orphaned resources that are difficult to clean up.

---

### How Each Step Was Verified

**Namespace deletion:**
```bash
kubectl get namespace demo
```
Expected response:
```
Error from server (NotFound): namespaces "demo" not found
```

**Pods cleared:**
```bash
kubectl get pods -n demo
```
Expected response:
```
No resources found in demo namespace
```

**EKS and Network stacks:** Confirmed deleted in AWS CloudFormation console — stacks no longer listed.

---

## Final Result

- Kubernetes namespace and all resources deleted
- EKS cluster and worker nodes deleted
- Network stack deleted
- All AWS resources cleaned up — no orphaned infrastructure remaining
