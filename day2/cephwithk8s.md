# Complete Guide: Deploy Ceph CSI with Helm & Configure Ceph Cluster

This guide will walk you through:
- ✅ Deploying Ceph CSI using Helm
- ✅ Configuring Ceph Cluster to get credentials
- ✅ Creating Kubernetes ConfigMap & Secret with Ceph credentials
- ✅ Setting up a StorageClass & PVC to use Ceph storage

## 🎯 Step 1: Install Ceph CSI using Helm

First, deploy the Ceph CSI plugin in Kubernetes using Helm.

### 1️⃣ Add the Ceph CSI Helm Repo

```bash
helm repo add ceph-csi https://ceph.github.io/csi-charts
helm repo update
```

### 2️⃣ Install the Ceph CSI RBD Driver

```bash
helm install ceph-csi-rbd ceph-csi/ceph-csi-rbd -n ceph-csi-rbd --create-namespace
```

👉 This will install the Ceph CSI RBD plugin in the `ceph-csi-rbd` namespace.

## 🎯 Step 2: Set Up Ceph Cluster & Get Required Credentials

You need to get monitor IPs, Ceph pool name, Ceph user, and key from your Ceph cluster.

### 1️⃣ Get the Monitor IPs

Run this on your Ceph cluster:

```bash
ceph mon dump | grep mon.
```

🔹 Example output:

```swift
mon.a v2:10.218.0.5:6789/0 mon.b v2:10.218.0.6:6789/0 mon.c v2:10.218.0.7:6789/0
```

👉 Monitor IPs: `10.218.0.5,10.218.0.6,10.218.0.7`

### 2️⃣ Get Available Ceph Pools

List storage pools:

```bash
ceph osd pool ls
```

🔹 Example output:

```nginx
ashu_pool
rbd
```

👉 Use `ashu_pool` (or any preferred pool).

### 3️⃣ Create a Ceph User for CSI

```bash
ceph auth get-or-create client.csi-user mon 'profile rbd' osd 'profile rbd pool=ashu_pool' mgr 'profile rbd'
```

🔹 Example output:

```ini
[client.csi-user]
    key = AQClvKdcVlsZCBAA1/zmr5yBNqnw80ddL4yVEw==
```

👉 User: `csi-user`  
👉 Key: `AQClvKdcVlsZCBAA1/zmr5yBNqnw80ddL4yVEw==`

## 🎯 Step 3: Create ConfigMap & Secret in Kubernetes

Now, use the credentials obtained from Step 2 in Kubernetes.

### 1️⃣ Create the Ceph ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ceph-csi-config
  namespace: ceph-csi-rbd
data:
  config.json: |
    [
      {
        "clusterID": "459e067e-0280-4c22-ac34-ce34a90544dd",
        "monitors": ["10.218.0.5:6789", "10.218.0.6:6789", "10.218.0.7:6789"]
      }
    ]
```

Apply:

```bash
kubectl apply -f ceph-csi-config.yaml
```

### 2️⃣ Create the Ceph Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ceph-csi-secret
  namespace: ceph-csi-rbd
type: Opaque
data:
  userID: $(echo -n "csi-user" | base64)
  userKey: $(echo -n "AQClvKdcVlsZCBAA1/zmr5yBNqnw80ddL4yVEw==" | base64)
```

Apply:

```bash
kubectl apply -f ceph-csi-secret.yaml
```

## 🎯 Step 4: Configure StorageClass in Kubernetes

Now, create a StorageClass to allow Kubernetes to provision volumes dynamically.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-rbd
provisioner: rbd.csi.ceph.com
parameters:
  clusterID: "459e067e-0280-4c22-ac34-ce34a90544dd"
  pool: "ashu_pool"
  csi.storage.k8s.io/provisioner-secret-name: ceph-csi-secret
  csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi-rbd
  csi.storage.k8s.io/controller-expand-secret-name: ceph-csi-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: ceph-csi-rbd
  csi.storage.k8s.io/node-stage-secret-name: ceph-csi-secret
  csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi-rbd
  csi.storage.k8s.io/fstype: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
```

Apply:

```bash
kubectl apply -f ceph-storage-class.yaml
```

## 🎯 Step 5: Test with a PersistentVolumeClaim (PVC)

Now, create a PVC using the Ceph-backed StorageClass.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: ceph-rbd
```

Apply:

```bash
kubectl apply -f ceph-pvc.yaml
```

Verify PVC status:

```bash
kubectl get pvc
```

It should show `Bound` if successful.

## 🎯 Step 6: Verify Everything

### 1️⃣ Check Ceph CSI Pods

```bash
kubectl get pods -n ceph-csi-rbd
```

Expected output:

```sql
ceph-csi-rbd-provisioner-0   Running
ceph-csi-rbd-nodeplugin-0    Running
```

### 2️⃣ Check StorageClass

```bash
kubectl get storageclass
```

Expected output:

```nginx
NAME       PROVISIONER          AGE
ceph-rbd   rbd.csi.ceph.com     5m
```

### 3️⃣ Check PVC

```bash
kubectl get pvc
```

Expected output:

```pgsql
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
ceph-pvc   Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   10Gi       RWO            ceph-rbd       1m
```

## 🎯 Summary

- ✅ Installed Ceph CSI using Helm
- ✅ Retrieved Ceph cluster details (monitors, pools, user, key)
- ✅ Created Kubernetes ConfigMap & Secret
- ✅ Configured a StorageClass for dynamic Ceph-backed PVCs
- ✅ Tested PVC creation