# Integrating OpenStack Object Storage (Swift) with Ceph RADOS Gateway (RGW) in Kubernetes

This guide will help you set up Ceph RADOS Gateway (RGW) as an S3-compatible object storage backend for OpenStack Swift, and integrate it with Kubernetes using CSI (Container Storage Interface).

## 🎯 Step 1: Setup Ceph RADOS Gateway (RGW)

Ceph RGW provides an S3-compatible API that can be used as a backend for OpenStack Swift.

### 1️⃣ Deploy Ceph RADOS Gateway (RGW)

If RGW is not already installed in your Ceph cluster, deploy it:

```bash
ceph orch apply rgw my-rgw --placement="1"
```

Verify:

```bash
ceph orch ls | grep rgw
```

👉 This will deploy an RGW instance in your Ceph cluster.

## 🎯 Step 2: Create a Ceph User for Swift Integration

We need to create a Ceph user with S3 & Swift capabilities.

### 1️⃣ Create a Ceph User

```bash
radosgw-admin user create --uid="swift-user" --display-name="Swift User" --caps="buckets=*;users=*;usage=*;metadata=*;zone=*"
```

🔹 Example output:

```json
{
    "user_id": "swift-user",
    "keys": [
        {
            "user": "swift-user",
            "access_key": "AKIAXXXX",
            "secret_key": "SECRETKEYXXXX"
        }
    ]
}
```

👉 Save the access_key and secret_key.

## 🎯 Step 3: Configure OpenStack Swift to Use Ceph

Now, configure OpenStack Swift to use Ceph RGW.

### 1️⃣ Edit Swift Proxy Configuration

On the OpenStack Swift proxy node, edit `/etc/swift/proxy-server.conf`:

```ini
[DEFAULT]
bind_port = 8080
bind_ip = 0.0.0.0
workers = 5

[pipeline:main]
pipeline = catch_errors cache swift3 s3token proxy-server

[filter:swift3]
use = egg:swift3#swift3

[filter:s3token]
paste.filter_factory = keystonemiddleware.s3_token:filter_factory

[filter:rgw]
use = egg:swift#rgw
rgw_storage_url = http://10.218.0.5:8080/
rgw_access_key = AKIAXXXX
rgw_secret_key = SECRETKEYXXXX
```

👉 Replace `10.218.0.5` with your Ceph RGW endpoint.
👉 Replace `rgw_access_key` & `rgw_secret_key` with the Ceph user credentials created earlier.

## 🎯 Step 4: Restart Swift Proxy Services

Restart Swift services to apply the configuration:

```bash
systemctl restart swift-proxy
```

Verify if the service is running:

```bash
systemctl status swift-proxy
```

## 🎯 Step 5: Verify OpenStack Swift with Ceph RGW

### 1️⃣ Check Swift Container Creation

Use OpenStack CLI to create a test container:

```bash
openstack container create test-container
```

Verify:

```bash
openstack container list
```

Expected output:

```pgsql
+------------------+
| Name             |
+------------------+
| test-container   |
+------------------+
```

👉 If the container is created successfully, OpenStack Swift is using Ceph RGW as storage.