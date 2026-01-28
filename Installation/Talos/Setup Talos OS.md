![[talos.png]]
# Installation Talos OS in Proxmox

## Preparation 

### Section 1: Setup on Proxmox
Download Talos iso on github official of siderolabs [Talos Release](https://github.com/siderolabs/talos/releases) metal-{amd/arm}64.iso
, after downloading the ISO from GitHub Siderolabs, upload the ISO to **Proxmox**.

Create a new VM on Proxmox
1. Create VM: General
   ![[1.1.png]]
2. Create VM: OS
   ![[1.2.png]]
3. Create VM: System
   ![[1.3.png]]
4. Create VM: Disks
   ![[1.4.png]]
5. Create VM: CPU
   ![[1.5.png]]
6. Create VM: Memory
   ![[1.6.png]]
7. Create VM: Network
   ![[1.7.png]]
8. Create VM: Confirm
   ![[1.8.png]]
 
>[!NOTE]
>Adjust the Proxmox VM resources as recommended by Talos. [Proxmox Recomendation](https://docs.siderolabs.com/talos/v1.12/platform-specific-installations/virtualized-platforms/proxmox)
>VM hardware content display
>![[hardware-talos-proxmox(1.1).png]]

### Section 2: Setup Kube on Talos OS

>[!NOTE]
>Start the Talos VM that was created earlier for **control-plane** and **worker**.
> ![[consol-talos-proxmox(1.3).png]]

Performing static network configuration settings in Talos console
![[set-talos-static-net(1.2).png]]

Install talosctl according to the remote device (Windows/Mac/Linux) [talosctl download](https://github.com/siderolabs/talos/releases)
After installing talosctl, you can generate files that will be used in Talos.
```
talosctl gen config fajar-cluster https://172.23.7.111:6443
```

The command will generate three files: talosconfig, controlplane.yaml, and worker.yaml. These will be used to set up the cluster and determine the **control-plane** or **worker** nodes.

#### Initialize for control-plane
Run the following command to apply the control plane manifest to the target IP (172.23.7.111).
```
talosctl apply-config --insecure --nodes 172.23.7.111 --file controlplane.yaml
```

When booting for the first time to initialize the node, activate etcd by doing the following:
![[boot-masih-ke-iso(1.4).png]]

When booting for the first time to initialize the node, activate etcd by doing the following:
```
talosctl bootstrap --talosconfig .\talosconfig --endpoints 172.23.7.111 --nodes 172.23.7.111
```
And wait until the node status becomes ready
![[after-set-bootstrap-first-node(1.5).png]]

Generate kubeconfig for accessing kubectl
```
talosctl kubeconfig . --talosconfig ./talosconfig --endpoints 172.23.7.111
```
And check using kubectl
```
kubectl --kubeconfig .\kubeconfig get nodes
```

#### Initialize for worker
Before initializing the worker node, you can configure the static network settings and hostname by editing the *worker.yaml* file, which will be executed during the initialization of the worker node.
![[set-network&hostname-in-manifest-worker(1.6.1).png]]
![[set-network&hostname-in-manifest-worker(1.6.2).png]]

Then run the worker node initialization
```
talosctl apply-config --insecure --nodes 172.23.7.114 --file worker.yaml
```

