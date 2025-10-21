# Synapsis Case 1: Automated Kubernetes Engine Setup

## Deskripsi
Repository ini dibuat untuk menyelesaikan **Case 1** dari assessment DevSecOps Engineer PT Synapsis Sinergi Digital.
Tujuan:
- Install K3s secara otomatis di master dan worker node
- Upgrade cluster ke versi tertentu tanpa drift
- Mengambil dan membagikan kubeconfig secara aman untuk tim developer

Cluster setup:
- Master: `192.168.10.201` (`k3s-master`)
- Worker 1: `192.168.10.202` (`k3s-worker1`)
- Worker 2: `192.168.10.203` (`k3s-worker2`)
- OS: Ubuntu Server 22.04
- Network: 192.168.10.0/24

---

## Struktur Repository
synapsis-case1/
├── inventory.ini
├── kubeconfig
│   └── k3s.yaml
├── playbook.yml
├── README.md
└── roles
    └── k3s
        ├── files
        │   └── master_token.txt
        └── tasks
            ├── install-master.yml
            ├── join-worker.yml
            └── upgrade.yml

- **inventory.ini** → daftar IP master & worker
- **playbook.yml** → playbook utama yang memanggil semua roles
- **roles/k3s/tasks/** → task modular untuk install, join, upgrade, dan fetch kubeconfig
- **roles/k3s/files/master_token.txt** → token master sementara untuk join worker
- **kubeconfig/k3s.yaml** → hasil decript kubeconfig dari master

---

## Cara Jalankan Playbook

1. Pastikan Ansible sudah terinstall:
```sudo apt update && sudo apt install ansible -y```

2. Pastikan semua node sudah bisa login SSH tanpa password:
```ssh ubuntu@192.168.10.202
   ssh ubuntu@192.168.10.203```

3. Pastikan user ubuntu punya sudo tanpa password:
```ubuntu ALL=(ALL) NOPASSWD:ALL```

4. Jalankan playbook utama:
```ansible-playbook -i inventory.ini playbook.yml```

   - Playbook ini otomatis:
     a. Install K3s master
     b. Generate token
     c. Install K3s worker dan join ke master
     d. Upgrade cluster ke versi v1.33.5+k3s1
     e. Ambil kubeconfig dari master dan encrypt untuk distribusi

---
# Verifikasi Cluster
Setelah playbook selesai, login ke master dan jalankan:
```sudo kubectl get nodes
   sudo kubectl get pods -A```

## Hasil yang diharapkan (contoh):
```NAME          STATUS   ROLES                  AGE   VERSION
   k3s-master    Ready    control-plane,master   10m   v1.33.5+k3s1
   k3s-worker1   Ready    <none>                 8m    v1.33.5+k3s1
   k3s-worker2   Ready    <none>                 8m    v1.33.5+k3s1```

---

## Distribusi Kubeconfig

- File kubeconfig terenkripsi ada di folder ./kubeconfig/k3s.yaml
- Untuk developer mendekripsi:

```ansible-vault decrypt ./kubeconfig/k3s.yaml```

- File ini bisa dipakai untuk mengakses cluster:

```export KUBECONFIG=./kubeconfig/k3s.yaml
   kubectl get nodes```

---

## Catatan

- Struktur ini modular dengan roles, memudahkan upgrade dan maintenance cluster di masa depan.
- Bisa langsung diadaptasi untuk cluster tambahan, cukup ubah inventory.ini.
- Pastikan urutan upgrade: master → worker1 → worker2 untuk menghindari versi tidak sinkron.
