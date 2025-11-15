# Kubernetes Networking & DNS Troubleshooting Guide

This md file contains steps to debug **frontend connectivity issues** in Kubernetes — both internal service-to-service DNS resolution and external connectivity.

---

## Check Internal Service DNS Resolution

When the frontend cannot reach the backend, the problem is usually **DNS or Service selector mismatch**.

### Step 1: Check Services and Selectors

```bash
kubectl get svc
kubectl describe svc backend-service
kubectl get endpoints backend-service
```

* Ensure `backend-service` has the correct selector matching the backend pod labels.

* Check `kubectl get endpoints backend-service` shows the backend pod IP. If empty → selector mismatch.

### Step 2: Test DNS Resolution Inside Pod

Exec into the frontend pod:

```bash
kubectl exec -it frontend-7d8f445b8c-cxnw4 -- sh
```

Inside the pod, test DNS and connectivity:

```bash
# Test DNS
nslookup backend-service

# Test connectivity to backend
curl http://backend-service:5000
```

If `nslookup` resolves the backend service IP and `curl` succeeds → internal DNS and connectivity are fine.

---

## Check External Connectivity

If your pods need to connect to external APIs, databases, or the Internet:

### Step 1: Test outbound connectivity from pod

```bash
kubectl exec -it frontend-7d8f445b8c-cxnw4 -- sh
curl -v https://example.com
ping 8.8.8.8
```

* If `ping` works but DNS fails → likely DNS issue.
* If `curl` fails → check NetworkPolicy, firewall, or security groups.

### Step 2: Check Network Policies

```bash
kubectl get networkpolicy --all-namespaces
```

### Step 3: Confirm kube-dns / CoreDNS is working

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system coredns-66bc5c9577-8drlj
```
---

## Common Fixes

| Issue                      | Fix                                                                         |
| -------------------------- | --------------------------------------------------------------------------- |
| Service selector mismatch  | Update service selector to match pod labels (`kubectl apply -f <svc.yaml>`) |
| Pod cannot resolve service | Confirm kube-dns / CoreDNS is running and healthy                           |
| NetworkPolicy blocking     | Edit or create NetworkPolicy to allow egress to the target port/IP          |
| External DNS blocked       | Allow UDP/TCP port 53 in cluster firewall / VM NAT settings                 |
| External HTTP blocked      | Allow egress to port 80/443                                                 |

---

## Confirm Fix

After applying fixes, re-test connectivity:

```bash
kubectl exec -it frontend-7d8f445b8c-cxnw4 -- curl -v http://backend-service:5000
```

Expected output:

```
Backend response here
```

Check frontend in the browser:

```
Backend says: <response from backend>
```

If the frontend shows the backend output in the browser, internal DNS and network are working.

---

