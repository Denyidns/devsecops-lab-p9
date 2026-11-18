# devsecops-lab-p9
# 🛡️ DevSecOps Lab 9: Container Security & Runtime Monitoring

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Docker](https://img.shields.io/badge/Docker-24.0+-blue)
![Trivy](https://img.shields.io/badge/Trivy-Vulnerability--Scanner-blueviolet)
![Falco](https://img.shields.io/badge/Falco-Runtime--Security-orange)

Repositori ini berisi dokumentasi praktikum, konfigurasi *secure container*, skrip otomatisasi CI/CD security gating, serta pemantauan keamanan *runtime* untuk mata kuliah DevSecOps (STT Terpadu Nurul Fikri).

---

## 📌 Topik & Tugas Praktikum

1. **Analisis Vulnerability (Image Scan)**: Pemindaian kerentanan CVE pada berbagai *base image* (`python:3.4-alpine`, `python:3.12-alpine`, dan `nginx:latest`) menggunakan Trivy.
2. **Hardening Dockerfile**: Pembuatan `Dockerfile.secure` berbasis multi-stage build dan non-root execution.
3. **Automated CI/CD Pipeline**: Integrasi Trivy vulnerability scanner ke dalam GitHub Actions workflow.
4. **Runtime Security Monitoring**: Deteksi anomali dan ancaman *real-time* di tingkat sistem menggunakan Falco & Event Generator.
5. **Container Hardening Checklist**: Implementasi 10 *best practices* keamanan container.

---

## 🚀 Fitur Keamanan pada `Dockerfile.secure`

Image dikonfigurasi mengikuti standar keamanan industri untuk meminimalisir *attack surface*:
* 📦 **Minimal Base Image**: Menggunakan `python:3.12-alpine` untuk memperkecil ukuran dan mengurangi jumlah CVE bawaan OS.
* 👤 **Non-Root User Execution**: Menjalankan aplikasi sebagai `appuser` (UID 1001) untuk mencegah *root privilege escalation*.
* 🏗️ **Multi-Stage Build**: Memisahkan lingkungan *build* dan *runtime* agar perkakas kompilasi tidak terbawa ke image produksi.
* 🧹 **Clean Package Management**: Menggunakan `pip install --no-cache-dir` dan `apk add --no-cache` untuk menjaga kebersihan image.
* 🙈 **File Exclusion (`.dockerignore`)**: Mencegah kebocoran berkas rahasia (`.git`, `.env`, kredensial) ke dalam layer image.

---

## 🛠️ Cara Menjalankan Project

### 1. Build & Run Secure Image (Lokal)

```bash
# Build image terharden
docker build -t app-secure:v1 -f Dockerfile.secure .

# Menjalankan container dengan non-root user
docker run -d --name secure-app-container app-secure:v1

# Memeriksa user yang sedang berjalan di dalam container (harus appuser / UID 1001)
docker exec secure-app-container whoami

2. Pemindaian Keamanan Menggunakan Trivy

# Scan kerentanan kerentanan (CVE) pada image
trivy image --scanners vuln app-secure:v1

# Scan dengan memfilter celah yang sudah ada patch-nya saja
trivy image --ignore-unfixed app-secure:v1

3. Simulasi Runtime Security Monitoring (Falco)
# Menjalankan simulasi ancaman runtime di container
docker run --rm -it falcosecurity/falco-event-generator run "Read sensitive file untrusted"

# Memeriksa log deteksi alert Falco Engine
docker logs -f falco

CI/CD Security Pipeline (GitHub Actions)
Workflow diletakkan pada .github/workflows/security-scan.yml. Setiap terjadi push atau pull_request ke branch main, pipeline akan otomatis:

Melakukan checkout kode sumber.

Membangun Docker Image secara terisolasi.

Menjalankan Trivy Vulnerability Scanner (aquasecurity/trivy-action@v0.36.0).

Mengunci versi action ke tag rilis spesifik untuk mencegah Supply Chain Attack.

Hasil Analisis Ringkas (Vulnerability Scan)

Hasil pemindaian keamanan menggunakan Trivy menunjukkan bahwa image legacy python:3.4-alpine memiliki tingkat kerentanan sangat tinggi dengan total 37 CVE yang mencakup 4 severity CRITICAL dan 13 HIGH, sedangkan pembaruan ke base image python:3.12-alpine berhasil mengeliminasi seluruh kerentanan CRITICAL meskipun masih menyisakan 16 kerentanan HIGH akibat dependensi OS yang belum di-patch. Melalui penerapan hardening pada app-secure:v1 (seperti multi-stage build, pembaruan paket runtime, dan pengosongan cache), seluruh celah keamanan berhasil ditambal hingga mencapai 0 CVE (PASSED) sehingga aman untuk digunakan pada lingkungan produksi.

Container Hardening Checklist Summary
[x] Minimal Base Image (python:3.12-alpine)

[x] Non-Root User (USER appuser)

[x] Explicit WORKDIR (/app)

[x] Multi-Stage Build Strategy

[x] .dockerignore Enforced

[x] Disabled Package Cache

[x] Explicit Image Version Tagging (Avoid :latest)

[x] Read-Only Root Filesystem Ready

[x] Package Manager Index Cleaned

[x] Runtime Monitoring Verified (Falco)

Deni Maulana - Informatics Engineering (STT Terpadu Nurul Fikri)

GitHub: @Denyidns
