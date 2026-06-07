## DESC\_Kubernates\_ConfigMap\_and\_Secret\_Demo

===

This repository contains a basic Kubernetes/K3s demo showing how to use **ConfigMaps** and **Secrets** inside a Pod.

The demo covers two common usage patterns:

1. Injecting ConfigMap and Secret values as **environment variables**
2. Mounting ConfigMap and Secret values as **files inside a container**

This demo works on:

* K3s
* Kubernetes
* Minikube
* kubeadm-based clusters
* EKS, AKS, and GKE

No K3s-specific feature is required for this demo.

\---

## Repository Files

```text
config-secret-demo/
├── 01-namespace.yaml
├── 02-configmap.yaml
├── 03-secret.yaml
└── 04-pod.yaml
```

|File|Purpose|
|-|-|
|`01-namespace.yaml`|Creates a separate namespace for the demo|
|`02-configmap.yaml`|Creates a ConfigMap for non-sensitive application configuration|
|`03-secret.yaml`|Creates a Secret for sensitive values|
|`04-pod.yaml`|Creates a Pod that consumes the ConfigMap and Secret|

\---

## Demo Architecture

```text
Kubernetes Cluster
│
├── Namespace: config-secret-demo
│
├── ConfigMap: app-config
│   ├── APP\_NAME
│   ├── APP\_ENV
│   ├── WELCOME\_MESSAGE
│   └── app.properties
│
├── Secret: app-secret
│   ├── DB\_USERNAME
│   ├── DB\_PASSWORD
│   └── credentials.txt
│
└── Pod: config-secret-pod
    ├── Uses ConfigMap values as environment variables
    ├── Uses Secret values as environment variables
    ├── Mounts ConfigMap as files under /etc/config
    └── Mounts Secret as files under /etc/secret
```

\---

## Concept Overview

### ConfigMap

A **ConfigMap** stores non-sensitive configuration data as key-value pairs.

Typical examples:

* application name
* environment name
* URLs
* feature flags
* configuration files

Example values used in this demo:

```text
APP\_NAME=student-portal
APP\_ENV=training
WELCOME\_MESSAGE=Hello from Kubernetes ConfigMap
```

\---

### Secret

A **Secret** stores sensitive values.

Typical examples:

* passwords
* tokens
* certificates
* API keys
* database credentials

Example values used in this demo:

```text
DB\_USERNAME=admin
DB\_PASSWORD=admin123
```

> Note: Kubernetes Secrets are base64-encoded by default. Base64 is not encryption. For stronger protection, use RBAC, limited permissions, and encryption at rest.

\---

## Prerequisites

Before running the demo, make sure:

* A Kubernetes or K3s cluster is running
* `kubectl` is installed and configured
* The current user has permission to create namespaces, ConfigMaps, Secrets, and Pods

Check cluster status:

```bash
kubectl get nodes
```

Expected example:

```text
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   10m   v1.xx.x+k3s1
```

\---

## Step 1: Create the Namespace

Apply the namespace manifest:

```bash
kubectl apply -f 01-namespace.yaml
```

### Command Meaning

|Command Part|Meaning|
|-|-|
|`kubectl`|Kubernetes command-line tool|
|`apply`|Creates or updates a resource|
|`-f`|Specifies the YAML file|
|`01-namespace.yaml`|Namespace manifest file|

Verify:

```bash
kubectl get ns
```

Expected output:

```text
NAME                 STATUS   AGE
config-secret-demo   Active   10s
```

\---

## Step 2: Create the ConfigMap

Apply the ConfigMap manifest:

```bash
kubectl apply -f 02-configmap.yaml
```

### Command Meaning

|Command Part|Meaning|
|-|-|
|`kubectl apply`|Creates or updates the resource|
|`-f 02-configmap.yaml`|Uses the ConfigMap YAML file|

Verify:

```bash
kubectl get configmap -n config-secret-demo
```

Expected output:

```text
NAME         DATA   AGE
app-config   4      10s
```

View details:

```bash
kubectl describe configmap app-config -n config-secret-demo
```

This shows the ConfigMap keys and values.

\---

## Step 3: Create the Secret

Apply the Secret manifest:

```bash
kubectl apply -f 03-secret.yaml
```

### Command Meaning

|Command Part|Meaning|
|-|-|
|`kubectl apply`|Creates or updates the resource|
|`-f 03-secret.yaml`|Uses the Secret YAML file|

Verify:

```bash
kubectl get secret -n config-secret-demo
```

Expected output:

```text
NAME         TYPE     DATA   AGE
app-secret   Opaque   3      10s
```

View metadata:

```bash
kubectl describe secret app-secret -n config-secret-demo
```

This shows Secret metadata, but not the plain text values.

\---

## Step 4: Create the Pod

Apply the Pod manifest:

```bash
kubectl apply -f 04-pod.yaml
```

### Command Meaning

|Command Part|Meaning|
|-|-|
|`kubectl apply`|Creates or updates the resource|
|`-f 04-pod.yaml`|Uses the Pod YAML file|

Verify:

```bash
kubectl get pods -n config-secret-demo
```

Expected output:

```text
NAME                READY   STATUS    RESTARTS   AGE
config-secret-pod   1/1     Running   0          10s
```

\---

## Step 5: Verify Environment Variables

Open a shell inside the Pod:

```bash
kubectl exec -it config-secret-pod -n config-secret-demo -- sh
```

### Command Meaning

|Command Part|Meaning|
|-|-|
|`kubectl exec`|Runs a command inside a container|
|`-it`|Opens an interactive terminal|
|`config-secret-pod`|Target Pod name|
|`-n config-secret-demo`|Target namespace|
|`-- sh`|Starts a shell inside the container|

Check ConfigMap values:

```bash
echo $APP\_NAME
```

Expected output:

```text
student-portal
```

```bash
echo $APP\_ENV
```

Expected output:

```text
training
```

Check Secret values:

```bash
echo $DB\_USERNAME
```

Expected output:

```text
admin
```

```bash
echo $DB\_PASSWORD
```

Expected output:

```text
admin123
```

\---

## Step 6: Verify ConfigMap Mounted as Files

Inside the Pod, list the mounted ConfigMap directory:

```bash
ls -l /etc/config
```

Expected files:

```text
APP\_ENV
APP\_NAME
WELCOME\_MESSAGE
app.properties
```

Read a ConfigMap key mounted as a file:

```bash
cat /etc/config/APP\_NAME
```

Expected output:

```text
student-portal
```

Read the mounted configuration file:

```bash
cat /etc/config/app.properties
```

Expected output:

```text
app.name=student-portal
app.environment=training
app.owner=devops-class
```

### Key Point

When a ConfigMap is mounted as a volume, each key becomes a file.

\---

## Step 7: Verify Secret Mounted as Files

Inside the Pod, list the mounted Secret directory:

```bash
ls -l /etc/secret
```

Expected files:

```text
DB\_PASSWORD
DB\_USERNAME
credentials.txt
```

Read a Secret key mounted as a file:

```bash
cat /etc/secret/DB\_USERNAME
```

Expected output:

```text
admin
```

Read the mounted credentials file:

```bash
cat /etc/secret/credentials.txt
```

Expected output:

```text
username=admin
password=admin123
```

Exit the container:

```bash
exit
```

### Key Point

Secrets can be mounted as files. This approach is commonly used for credentials, tokens, certificates, and keys.

\---

## Step 8: View Encoded Secret Data

Display the Secret in YAML format:

```bash
kubectl get secret app-secret -n config-secret-demo -o yaml
```

Example output:

```yaml
data:
  DB\_PASSWORD: YWRtaW4xMjM=
  DB\_USERNAME: YWRtaW4=
```

Decode an example value:

```bash
echo YWRtaW4xMjM= | base64 -d
```

Expected output:

```text
admin123
```

### Important Note

Base64 encoding is not encryption. Anyone with permission to read the Secret can decode its values.

\---

## Full Command Sequence

```bash
# Check cluster status
kubectl get nodes

# Create namespace
kubectl apply -f 01-namespace.yaml

# Verify namespace
kubectl get ns

# Create ConfigMap
kubectl apply -f 02-configmap.yaml

# Verify ConfigMap
kubectl get configmap -n config-secret-demo
kubectl describe configmap app-config -n config-secret-demo

# Create Secret
kubectl apply -f 03-secret.yaml

# Verify Secret
kubectl get secret -n config-secret-demo
kubectl describe secret app-secret -n config-secret-demo

# Create Pod
kubectl apply -f 04-pod.yaml

# Verify Pod
kubectl get pods -n config-secret-demo

# Enter Pod
kubectl exec -it config-secret-pod -n config-secret-demo -- sh
```

Inside the Pod:

```bash
# Check ConfigMap environment variables
echo $APP\_NAME
echo $APP\_ENV

# Check Secret environment variables
echo $DB\_USERNAME
echo $DB\_PASSWORD

# Check mounted ConfigMap files
ls -l /etc/config
cat /etc/config/APP\_NAME
cat /etc/config/app.properties

# Check mounted Secret files
ls -l /etc/secret
cat /etc/secret/DB\_USERNAME
cat /etc/secret/credentials.txt

# Exit container
exit
```

Outside the Pod:

```bash
# Show Secret in YAML format
kubectl get secret app-secret -n config-secret-demo -o yaml

# Decode sample Secret value
echo YWRtaW4xMjM= | base64 -d
```

\---

## Troubleshooting

### Pod Is Not Running

Check Pod status:

```bash
kubectl get pods -n config-secret-demo
```

Describe the Pod:

```bash
kubectl describe pod config-secret-pod -n config-secret-demo
```

Check the `Events` section at the bottom.

Common causes:

|Issue|Possible Cause|
|-|-|
|`CreateContainerConfigError`|ConfigMap or Secret name/key is wrong|
|`ImagePullBackOff`|Image cannot be pulled|
|`Pending`|Node is not ready or scheduling issue|
|Environment variable is empty|Wrong key name in ConfigMap or Secret|

\---

### ConfigMap Key Not Found

Check ConfigMap contents:

```bash
kubectl describe configmap app-config -n config-secret-demo
```

Make sure the key used in the Pod exists in the ConfigMap.

\---

### Secret Key Not Found

Check Secret metadata:

```bash
kubectl describe secret app-secret -n config-secret-demo
```

Make sure the key used in the Pod exists in the Secret.

\---

## Cleanup

Delete the complete demo namespace:

```bash
kubectl delete namespace config-secret-demo
```

This removes:

* Namespace
* ConfigMap
* Secret
* Pod

Verify:

```bash
kubectl get ns
```

\---

## Summary

ConfigMaps and Secrets separate configuration from container images.

Use **ConfigMaps** for normal, non-sensitive configuration:

```text
APP\_NAME
APP\_ENV
WELCOME\_MESSAGE
application.properties
```

Use **Secrets** for sensitive values:

```text
DB\_USERNAME
DB\_PASSWORD
tokens
certificates
keys
```

Both can be consumed in two common ways:

```text
1. Environment variables
2. Mounted files
```

Final reminder:

> Do not hardcode configuration or passwords inside container images. Use ConfigMaps for normal configuration and Secrets for sensitive values.

