# Talos Linux on VirtualBox - Step-by-Step Guide

This guide explains how to deploy a **Talos Linux Kubernetes cluster on VirtualBox** using `talosctl`.  
It includes all commands, explanations, and small adjustments so anyone can reproduce the setup easily.

---

## 1. Prerequisites

Before starting, make sure you have the following installed:

- VirtualBox
- A virtual machine running **Talos Linux ISO**
- `curl`
- `kubectl` (optional but recommended)

You also need the **IP address of the Talos VM**. In this example, we will use:

```
192.168.0.109
```

---

## 2. Install talosctl

First, install `talosctl`, which is the CLI tool used to manage Talos nodes.

```bash
curl -sL https://talos.dev/install | sh
```

This command downloads and installs the latest version of `talosctl` automatically.

---

## 3. Check Available Disks on the Node

Before generating the cluster configuration, you need to know which disk Talos will install to.

```bash
talosctl get disks --insecure --nodes 192.168.0.109
```

Explanation:

- `--insecure` → used because the node is not configured yet
- `--nodes` → IP address of the Talos VM

Look for something like:

```
sda
```

---

## 4. Define Environment Variables

These variables make the commands easier to reuse.

```bash
export CONTROL_PLANE_IP=192.168.0.109
export CLUSTER_NAME=k8s-talos
export DISK_NAME=sda
```

Explanation:

- `CONTROL_PLANE_IP` → IP of the Talos node
- `CLUSTER_NAME` → name of your Kubernetes cluster
- `DISK_NAME` → disk where Talos will be installed

---

## 5. Generate the Talos Cluster Configuration

Now generate the configuration files.

```bash
talosctl gen config $CLUSTER_NAME https://$CONTROL_PLANE_IP:6443 --install-disk /dev/$DISK_NAME
```

This command generates the following files:

- `controlplane.yaml`
- `worker.yaml`
- `talosconfig`

Explanation:

- `gen config` → generates the Kubernetes + Talos configuration
- `https://$CONTROL_PLANE_IP:6443` → Kubernetes API endpoint
- `--install-disk` → disk where Talos will be installed

---

## 6. Apply the Configuration to the Node

Now apply the control plane configuration to the Talos VM.

```bash
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file controlplane.yaml
```

Explanation:

- `apply-config` → sends the configuration to the node
- `--insecure` → still required because the node is not fully configured

---

## 7. Configure talosctl to Use the Node

```bash
talosctl --talosconfig=./talosconfig config endpoints $CONTROL_PLANE_IP
```

This command tells `talosctl` which node it should talk to.

---

## 8. Bootstrap the Kubernetes Cluster

Now bootstrap the cluster:

```bash
talosctl bootstrap --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig
```

This step initializes the Kubernetes control plane.

---

## 9. Set the Talos Configuration Path

```bash
export TALOSCONFIG=/home/luisgustavo/talosconfig
```

This makes `talosctl` automatically use the correct configuration file.

---

## 10. Check Cluster Health

```bash
talosctl health -n $CONTROL_PLANE_IP
```

This command verifies:

- Kubernetes API
- etcd
- control plane components

---

## 11. Generate kubeconfig

Now generate a kubeconfig file so you can use `kubectl`.

```bash
talosctl kubeconfig alternative-kubeconfig --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig
```

Then export it:

```bash
export KUBECONFIG=/home/luisgustavo/alternative-kubeconfig
```

Now test it:

```bash
kubectl get nodes
```

---

## 12. Allow Workloads on the Control Plane (Optional - for Labs)

If you are using **only one VM**, you must allow workloads to run on the control plane.

Edit the file:

```bash
vim controlplane.yaml
```

Add this configuration:

```yaml
cluster:
  allowSchedulingOnControlPlanes: true
```

Apply the configuration again:

```bash
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file controlplane.yaml
```

---

## 13. Reboot the Node

```bash
talosctl reboot -n $CONTROL_PLANE_IP
```

---

## 14. Final Test

After the reboot:

```bash
kubectl get nodes
kubectl get pods -A
```

If everything worked correctly, your Talos Kubernetes cluster is ready 🚀

---

## 15. What You Achieved

With this guide you:

- Installed Talos Linux on VirtualBox
- Created a Kubernetes control plane
- Generated kubeconfig
- Enabled scheduling on a single-node cluster
- Successfully deployed a working Kubernetes cluster

---

## 16. Next Steps (Recommended for GitHub README)

You can extend this lab with:

- Installing NGINX Ingress
- Installing metrics-server
- Deploying a test application
- Creating a second worker node

---
