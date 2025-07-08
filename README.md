The error you're seeing:

```
Error response from daemon: bridge is a pre-defined network and cannot be removed
```

means that `bridge` is one of Docker’s **default networks**, and **Docker does not allow removing default networks** like:

* `bridge`
* `host`
* `none`

These are **built-in** and are essential for Docker’s operation.

---

### 🔍 Default Docker Networks:

| Name     | Type   | Description                                     |
| -------- | ------ | ----------------------------------------------- |
| `bridge` | bridge | Default network for containers on a single host |
| `host`   | host   | Shares the host’s networking stack              |
| `none`   | null   | Disables networking for a container             |

You **can only remove user-defined networks**, like this one from your earlier output:

```
578e6511f261   node_project_node_mysql_net   bridge    local
```

---

### ✅ So in summary:

* You **cannot delete** the `bridge`, `host`, or `none` networks.
* You **can delete** custom networks (those you create with `docker network create`) **if they’re not in use**.

Let me know if you want to list containers using a network or clean up all unused networks.


--------------------------------------------
You're almost there! Here’s what’s happening and what you need to do:

---

### ❌ Problem:

You typed:

```bash
docke volume rm all
```

* `docke` is a typo → should be `docker`
* `all` is not a valid volume name

---

### ✅ Correct Ways to Remove Docker Volumes

#### 🔹 Option 1: Remove a **specific volume** by name:

```bash
docker volume rm <volume_name>
```

Example:

```bash
docker volume rm node_project_mysql_data
```

---

#### 🔹 Option 2: Remove **all unused volumes** (safe cleanup):

```bash
docker volume prune
```

This removes **volumes not used by any containers**.
You’ll be prompted to confirm.

---

#### 🔹 Option 3: Remove **all volumes forcefully** (⚠️ use with caution):

```bash
docker volume rm $(docker volume ls -q)
```

This command:

* `docker volume ls -q` → lists all volume names
* `docker volume rm ...` → deletes them all

⚠️ **Danger:** This will remove **all volumes**, even if they're in use, causing **data loss** for containers using those volumes.

---

### Let me know if you want:

* To back up volume data before deletion
* To list which containers are using which volumes
* To automate cleanup in a script

I'm here to help!
------------------------------------

Sure! Here's a **step-by-step guide** to test DNS resolution inside your Kubernetes cluster using a temporary pod:

---

### 🔍 **Goal**

You want to check if the DNS entry (FQDN) of a service like `mysql-svc.default.svc.cluster.local` can be resolved **inside the cluster**.

---

## ✅ Step-by-Step DNS Resolution Test in Kubernetes

---

### **Step 1: Create a temporary pod with DNS tools**

We'll use `busybox` or `dnsutils` image which includes `nslookup` or `dig`.

Run this command:

```bash
kubectl run dnsutils --rm -it --image=busybox:1.28 --restart=Never -- sh
```

> ⏳ Wait a moment — this will start a pod named `dnsutils` and drop you into its shell (`sh`).

---

### **Step 2: Inside the pod, run DNS resolution**

Now you're inside the pod shell. Run this command:

```bash
nslookup mysql-svc.default.svc.cluster.local
```

You should get an output like:

```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      mysql-svc.default.svc.cluster.local
Address 1: 10.96.241.43
```

✅ This confirms that your service `mysql-svc` is **resolvable** via DNS inside the cluster.

---

### **Step 3: Exit the temporary pod**

Just type:

```bash
exit
```

This will remove the temporary pod (because of `--rm` flag).

---

## 🧪 Optional: Use `dig` command instead of `nslookup`

If you use an image with `dig` (e.g., `infoblox/dig`), you can test it like this:

```bash
kubectl run digpod --rm -it --image=infoblox/dig --restart=Never -- dig mysql-svc.default.svc.cluster.local
```

---

### 🔁 Summary:

| Step | Action                                       |
| ---- | -------------------------------------------- |
| 1    | Run a temporary pod with shell using busybox |
| 2    | Run `nslookup <service-fqdn>` inside pod     |
| 3    | Confirm output shows the Cluster IP          |
| 4    | Type `exit` to close the pod                 |

Let me know if you'd like to **test DNS for another service** or troubleshoot if resolution fails.
Great! Below is the **updated single YAML file** (`phpmyadmin-deploy.yaml`) that includes:

1. `ConfigMap` for phpMyAdmin
2. `phpMyAdmin` Pod with `seccompProfile` fix
3. `phpMyAdmin` NodePort Service
4. ✅ A **temporary test Pod (`dnsutils`)** to verify DNS resolution inside the cluster

---

### ✅ File: `phpmyadmin-deploy.yaml`

```yaml
# ---------------------------------------------------
# 📘 Full Deployment: phpMyAdmin + DNS Test Pod
# ---------------------------------------------------

# 📌 1. ConfigMap for phpMyAdmin environment variables
apiVersion: v1
kind: ConfigMap
metadata:
  name: phpadmin-config
data:
  PMA_HOST: "mysql-svc.default.svc.cluster.local"
  PMA_PORT: "3306"

---

# 📌 2. phpMyAdmin Pod (Fixed: added seccompProfile)
apiVersion: v1
kind: Pod
metadata:
  name: phpadmin-pod
  labels:
    run: phpadmin-pod
spec:
  containers:
    - name: phpadmin-pod
      image: phpmyadmin
      envFrom:
        - configMapRef:
            name: phpadmin-config
      securityContext:
        seccompProfile:
          type: Unconfined  # ✅ Fixes "Operation not permitted" error
  dnsPolicy: ClusterFirst
  restartPolicy: Always

---

# 📌 3. phpMyAdmin Service (NodePort to access from browser)
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-svc
  labels:
    app: phpmyadmin-svc
spec:
  type: NodePort
  selector:
    run: phpadmin-pod
  ports:
    - name: phpmyadmin-port
      protocol: TCP
      port: 8099
      targetPort: 80
      nodePort: 30248

---

# 📌 4. (Optional) DNS test Pod to validate service FQDN resolution
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
spec:
  containers:
    - name: dnsutils
      image: busybox:1.28
      command:
        - sh
        - -c
        - "sleep 3600"
  restartPolicy: Never
```

---

### ✅ After Applying

```bash
kubectl apply -f phpmyadmin-deploy.yaml
```

---

### 🔍 Test DNS Resolution from `dnsutils` Pod

```bash
kubectl exec -it dnsutils -- nslookup mysql-svc.default.svc.cluster.local
kubectl exec -it dnsutils -- nc -zv mysql-svc.default.svc.cluster.local 3306
```

---

### 🌐 Access phpMyAdmin

```
http://<EC2_PUBLIC_IP>:30248
```

Use:

* Username: `root`
* Password: (from your MySQL secret or Dockerfile)
* Server: `mysql-svc.default.svc.cluster.local`

---

Let me know if you want me to include the MySQL Pod/StatefulSet + volume config in the same file too.
------------------------------------Thanks for sharing the current configuration. The issue is now **clearly identified**:

---

## ❌ Root Cause

Your **`mysql-svc.yaml` has a label selector mismatch.**

```yaml
selector:
  app: mysql-sts   ❌ (Incorrect)
```

But in your **`db-pod.yaml`**, the pod is labeled as:

```yaml
metadata:
  labels:
    run: mysql-pod ✅
```

So the `mysql-svc` cannot find the MySQL pod, and **phpMyAdmin cannot connect**, hence the `mysqli::real_connect(): Operation not permitted` error.

---

## ✅ Solution

### 🔧 Update `mysql-svc.yaml` with the correct selector:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    run: mysql-pod     # ✅ This must match db-pod.yaml
  type: ClusterIP
```

---

## 🔁 Apply the fix

```bash
kubectl delete -f mysql-svc.yaml
kubectl apply -f mysql-svc.yaml
```

Then verify:

```bash
kubectl get endpoints mysql-svc
```

You should see a valid pod IP listed (e.g., `100.67.x.x:3306`). If it's `<none>`, the selector is still wrong.

---

## ✅ Next Step

After the fix, visit your phpMyAdmin:

```bash
http://<EC2_PUBLIC_IP>:30248
```

Login:

* Username: `root`
* Password: `Passw0rd`

If you still face issues, I can verify your remaining YAML files again.

Let me know if you'd like me to regenerate the final fixed `mysql-svc.yaml`.
--------------------------------------------------------------------------------
Great! Let's secure your application with **TLS (HTTPS)** using **Let's Encrypt** and **cert-manager** in your Kubernetes cluster. Here's a **step-by-step guide** to set it up properly on your `apps-ingress`.

---

## 🔐 TLS with Let's Encrypt on Kubernetes (via cert-manager)

---

### ✅ Prerequisites

1. ✅ You’re using **Ingress NGINX** (`ingress-nginx-controller`) with a **public LoadBalancer** (confirmed).
2. ✅ Your domains `loginapp.gangadev.site` and `dbmanage.gangadev.site` already point to the ELB (confirmed).
3. ⚠️ Port **80 and 443** must be open in your AWS security group for the LoadBalancer.

---

### 📦 Step 1: Install `cert-manager`

Apply the cert-manager manifest:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml
```

Wait 1–2 minutes and verify:

```bash
kubectl get pods --namespace cert-manager
```

You should see:

```
cert-manager-xxxxx          Running
cert-manager-cainjector     Running
cert-manager-webhook        Running
```

---

### 🆔 Step 2: Create Let's Encrypt Issuer (or ClusterIssuer)

We'll use a **ClusterIssuer** for simplicity:

```yaml
# cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        ingress:
          class: nginx
```

Replace `your-email@example.com` with your actual email. Then apply:

```bash
kubectl apply -f cluster-issuer.yaml
```

---

### 🔐 Step 3: Modify Your Ingress to Use TLS

Update your `application-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - loginapp.gangadev.site
        - dbmanage.gangadev.site
      secretName: gangadev-tls
  rules:
    - host: loginapp.gangadev.site
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-svc
                port:
                  number: 5000
    - host: dbmanage.gangadev.site
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: phpmyadmin-svc
                port:
                  number: 8099
```

Then apply it:

```bash
kubectl apply -f application-ingress.yaml
```

---

### 🔍 Step 4: Verify Certificate Request

Check for `Certificate` and `Order` resources:

```bash
kubectl get certificate
kubectl describe certificate gangadev-tls
```

You should see it being issued.

---

### ✅ Step 5: Access Over HTTPS

After a minute or two, visit:

* [https://loginapp.gangadev.site](https://loginapp.gangadev.site)
* [https://dbmanage.gangadev.site](https://dbmanage.gangadev.site)

✅ You should get a **valid green padlock** with no warning.

---

## 💬 Let Me Know

* ✅ Once you finish applying the steps
* 🚨 If you'd prefer a `staging` (test) issuer first
* 👇 If you want me to generate these YAMLs for you automatically

Would you like me to generate and give you the exact YAML files ready to apply?




