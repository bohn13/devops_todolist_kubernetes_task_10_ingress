# 🚀 1. Prerequisites

Before starting, ensure you have the following tools installed:

* **Docker**: To run the cluster nodes.
* **kind**: To spin up the Kubernetes cluster using the `cluster.yml` configuration.
* **kubectl**: To interact with the cluster.

## 🏗️ 2. Cluster Setup

To create the cluster with the necessary port mappings (Ports 80 and 443) for Ingress, run:

```bash
kind create cluster --config cluster.yml --name todoapp-cluster
```

> **Note**: The `cluster.yml` must include `extraPortMappings` for `hostPort: 80` to allow access via `http://localhost`.

## 📦 3. Automated Deployment

Run the bootstrap script to deploy the infrastructure, the application, and the NGINX Ingress Controller:

```bash
chmod +x bootstrap.sh
./bootstrap.sh
```

**What this script does:**

1. Creates the `todoapp` and `mysql` namespaces.
2. Installs the **NGINX Ingress Controller** using the official `kind` provider manifest.
3. Deploys the MySQL **StatefulSet** and the Django **Deployment**.
4. Applies the Ingress resource located in `./infrastructure/ingress/ingress.yml`.

---

## 🔍 4. Validation Steps

### Check Ingress Status

Verify that the Ingress controller is running and the Ingress resource is correctly configured:

```bash
# Check Ingress Controller pod
kubectl get pods -n ingress-nginx

# Check Ingress resource in the application namespace
kubectl get ingress -n todoapp
```

### Accessing the Application

Open your web browser and navigate to:
**`http://localhost`**

**Expected Result:** You should see the ToDo application interface running.

### Verifying Path Rules and 404 Status

To ensure the regex rule `/(/|$)(.*)` is capturing paths correctly and no resources are missing:

1. Open **Developer Tools** (F12) in your browser.
2. Go to the **Network** tab.
3. Refresh the page.
4. **Validation**: Confirm that all requests (CSS, JS, API calls) return a `200 OK` status. There should be **no 404 status codes** in the console.

---

## 🛠️ Ingress Configuration Details

The Ingress manifest is located at `./infrastructure/ingress/ingress.yml` and adheres to the following requirements:

* **HTTP Rule**: Defined under `spec.rules`.
* **Path-based Rule**: Uses a regex capture group `/(/|$)(.*)` to forward all traffic to the backend service.
* **Backend**: Routes traffic to `todoapp-service` on port 80.
* **Rewrite Annotation**: `nginx.ingress.kubernetes.io/rewrite-target: /$2` ensures the app receives the correct sub-paths.

---

## 🧹 5. Cleanup

To delete the cluster and all associated resources:

```bash
kind delete cluster --name todoapp-cluster
```

