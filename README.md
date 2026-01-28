<div align="center">

# ☸️ Kubernetes Ops Notes

![Kubernetes Version](https://img.shields.io/badge/Kubernetes-v1.34-326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Talos Version](https://img.shields.io/badge/Talos_Linux-Ready-ff5722?style=for-the-badge&logo=linux&logoColor=white)
![Maintained](https://img.shields.io/badge/Maintained%3F-Yes-00C7B7?style=for-the-badge&logo=git&logoColor=white)

<p align="center">
  <b>Dokumentasi perjalanan, konfigurasi, dan "contekan" setup Cluster Kubernetes & Talos OS.</b><br>
  <i>Ditulis menggunakan Obsidian, disimpan untuk keabadian.</i>
</p>

<img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&pause=1000&color=326CE5&center=true&vCenter=true&width=435&lines=Kubernetes+Administration;Talos+Linux+Bootstrapping;Proxmox+Virtualization;Cloud+Native+Networking" alt="Typing SVG" />

</div>

---

## 🛠️ Tech Stack & Tools

Kumpulan teknologi yang dibahas atau digunakan dalam repositori ini:

<div align="center">

<a href="https://kubernetes.io/">
  <img src="https://skillicons.dev/icons?i=kubernetes,docker,linux,bash" />
</a>
<br/>
<a href="https://proxmox.com/">
  <img src="https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white" />
</a>
<a href="https://www.talos.dev/">
  <img src="https://img.shields.io/badge/Talos_OS-000000?style=for-the-badge&logo=linux&logoColor=white" />
</a>
<a href="https://obsidian.md/">
  <img src="https://img.shields.io/badge/Obsidian-7C3AED?style=for-the-badge&logo=obsidian&logoColor=white" />
</a>

</div>

---

## 🗺️ Navigation Hub

Langsung loncat ke topik yang kamu butuhkan. Gak perlu *scrolling* lama-lama.

### 🏗️ Installation & Setup

| Topik | Deskripsi | Link |
| :--- | :--- | :---: |
| **K8s Native** | Panduan instalasi manual Kubernetes v1.34 (Kubeadm) | [➡️ Baca](./Installation/Installation%20K8s%20Native%20(v1.34).md) |
| **Talos Linux** | Setup modern OS immutable (Talos) di atas Proxmox | [➡️ Baca](./Installation/Talos/Setup%20Talos%20OS.md) |

### 🚑 Operations & Troubleshooting

> *"If it breaks, we fix it."*

* [❌ Troubleshoot Log](./Installation/Troubleshoot.md) - Kumpulan error, solusi, dan drama saat instalasi.

---

## 📸 Sneak Peek

Berikut adalah tampilan topologi atau proses yang didokumentasikan:

<div align="center">
  <img src="./Installation/Talos/images/talos.png" width="45%" alt="Talos Architecture" />
  <img src="./Installation/images/installation.kube-install.png" width="45%" alt="K8s Installation" />
</div>

---

## ⚡ Quick Cheatsheet

Beberapa command sakti yang sering lupa:

```bash
# Cek Nodes di Talos
talosctl -n <IP> dashboard

# Generate Kubeconfig (Talos)
talosctl kubeconfig . --talosconfig ./talosconfig --endpoints <IP-MASTER>

# Reset Cluster (Kubeadm)
kubeadm reset -f && iptables -F && iptables -t nat -F