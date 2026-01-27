## SystemdCgroup
seperti sebelumnya jangan lupa mengaktifkan drivers SystemdCgroup untuk container runtimenya
```
SystemdCgroup = true
```

>[!WARNING]
>failed to run Kubelet: misconfiguration:
>kubelet cgroup driver: systemd
>container runtime cgroup driver: cgroupfs

### Reset Kube
**Kamu bisa pakai langsung `kubeadm reset` seperti di dokumentasi resmi**, itu memang cara yang disarankan buat _"balikin node ke keadaan sebelum `kubeadm init`/`join`"_ tanpa drama terselubung.

Hentikan service kubelet agar tidak membuat pod secara auto
```
systemctl stop kubelet
```

reset kubeadm
```
kubeadm reset -f
```

hapus folder/file yang bersangkutan
```
sudo rm -rf \
  /etc/kubernetes \
  /var/lib/etcd \
  /var/lib/kubelet \
  /etc/cni \
  /opt/cni \
  /var/lib/cni \
  ~/.kube

```

reset rule iptables
```
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X
```

>[!NOTE]
> **Opsional**
> jika menggunakan ipvadm 
> ```
> sudo ipvsadm --clear 2>/dev/null || true
> ```

hidupkan kembali service kubelet
```
sudo systemctl restart kubelet
```

aktifkan SystemdCgroupnya
```
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

cek apakah nilai sudah berubah atau belum
```
sed -n '/systemd/Ip' /etc/containerd/config.toml
```

ulangi membuat cluster lagi atau join cluster

>[!NOTE]
>**ISSUE**
>karena sebelumnya saat ingin init kubernetes tidak mau dan karena images yang dibutuhkan oleh kubernetes ternyata tidak mau terpull, karenanya dilakukan untuk metrigger atau memancing agar images nya mau dipull dahulu 
>#### Tool
>```
>sudo apt install -y cri-tools
>crictl pull registry.k8s.io/pause:3.9
>crictl pull busybox:latest
>systemctl restart containerd
>systemctl restart kubelet
>```



