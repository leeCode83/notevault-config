# 🚀 Universal CI/CD Mastery Guide

Selamat datang di panduan lengkap membangun pipeline CI/CD modern menggunakan **GitOps**. Panduan ini dirancang agar kamu bisa menerapkannya di proyek apa pun (Universal).

---

## 🏗️ Strategi Repositori
Gunakan dua repositori terpisah:
1.  **Repo Aplikasi**: Berisi kode sumber (`src/`, `test/`) dan CI workflow.
2.  **Repo Config**: Berisi Manifest Kubernetes (`yaml`) dan konfigurasi CD.

---

## 🛠️ Modul 1: CI Pipeline Fundamentals (Quality Control)

**Langkah demi Langkah:**
1.  **Persiapan Lokal**: Pastikan aplikasimu punya script linting dan testing (misal: `flake8`, `pytest`).
2.  **Buat Folder Workflow**: Di Repo Aplikasi, buat struktur folder `.github/workflows/`.
3.  **Definisikan CI (`ci.yaml`)**:
    -   Pilih OS runner (misal: `ubuntu-latest`).
    -   Gunakan action `actions/checkout@v4`.
    -   Setup bahasa (Python/Node/Go).
    -   Install dependencies & jalankan linting.
    -   Jalankan unit tests.
4.  **Uji Coba**: Push kode ke GitHub, cek tab **Actions**. Jika hijau, lanjut ke modul berikutnya.

---

## 📦 Modul 2: Containerization & Registry (Building Artifacts)

**Langkah demi Langkah:**
1.  **Tulis Dockerfile**: Pastikan dockerfile bisa berjalan di lokal (`docker build`).
2.  **Login ke Registry**: Tambahkan langkah di `ci.yaml` menggunakan `docker/login-action`. Gunakan `secrets.GITHUB_TOKEN` untuk GHCR.
3.  **Gunakan Tag Dinamis**: Gunakan `docker/metadata-action` untuk membuat tag `latest` dan `sha-<git-commit-hash>`. Tag SHA sangat penting bagi GitOps untuk mendeteksi perubahan.
4.  **Build & Push**: Gunakan `docker/build-push-action`.
5.  **Penting**: Jika menggunakan **Private Registry**, pastikan kamu mengubah setting visibilitas package menjadi **Public** atau membuat `imagePullSecret` di Kubernetes agar gambar bisa ditarik.

---

## ☸️ Modul 3: GitOps Foundation with ArgoCD (Deployment)

**Langkah demi Langkah:**
1.  **Setup Cluster**: Jalankan Minikube atau cluster K8s lainnya.
2.  **Install ArgoCD**:
    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```
3.  **Tulis Manifest (Repo Config)**:
    -   Buat folder `environments/dev/` dan `environments/prod/`.
    -   Buat file `deployment.yaml`, `service.yaml`, dan `configmap.yaml`.
4.  **Keamanan Manual**: Jangan simpan secret di Git. Jalankan `kubectl create secret generic nama-secret --from-literal=KEY=VALUE -n namespace-kamu`.
5.  **Daftarkan App ke ArgoCD**:
    -   Buat file `argocd/app.yaml` yang berisi `source` (alamat Repo Config) dan `destination` (cluster & namespace).
    -   Jalankan `kubectl apply -f argocd/app.yaml`.

---

## 🤖 Modul 4: The Automation Bridge (Connecting CI to CD)

**Langkah demi Langkah:**
1.  **Buat Token (PAT)**: Buat Personal Access Token di GitHub dengan izin `repo`.
2.  **Pasang Secret**: Masukkan PAT tadi ke **Repo Aplikasi** (tab Settings > Secrets) dengan nama `CONFIG_REPO_TOKEN`.
3.  **Update CI Workflow**: Tambahkan job di akhir `ci.yaml`:
    -   Clone Repo Config.
    -   Update file `deployment.yaml` menggunakan perintah `sed` untuk mengganti tag image lama dengan tag SHA baru.
    -   Commit dan Push kembali ke Repo Config.
4.  **Hasil**: Saat kamu push kode aplikasi, file di repo config akan berubah otomatis, dan ArgoCD akan langsung mendeteksinya.

---

## 🏥 Modul 5: Operations & Safety (Rollback & Observaility)

**Langkah demi Langkah:**
1.  **Promosi ke Prod**: Salin nilai tag image yang stabil dari folder `dev` ke folder `production` di Repo Config secara manual/PR.
2.  **Simulasi Fail**: Coba push kode yang menghasilkan image yang salah. ArgoCD akan menunjukkan status merah.
3.  **Rollback Cepat**:
    -   Di dashboard ArgoCD, klik **History and Rollback**.
    -   Klik versi lama yang sehat, klik **Rollback**. Kubernetes akan "menghidupkan kembali" pod lama dalam hitungan detik.
4.  **Cek Kesehatan (Monitoring)**:
    -   `kubectl get pods -n <namespace>`
    -   `kubectl logs -f <pod-name> -n <namespace>`

---

## 🧹 Cleanup (Membersihkan Semuanya)

1.  Hapus App ArgoCD: `kubectl delete -f argocd/app.yaml`
2.  Hapus Namespace: `kubectl delete namespace <nama-namespace>`
3.  Matikan Minikube: `minikube stop` atau `minikube delete` (untuk hapus total).

**Selamat! Kamu sekarang memiliki workflow kelas industri yang bisa kamu bawa ke proyek mana saja!** 🚀
