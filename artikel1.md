

### **Artikel Praktikum: Hari ke-1 - Membangun Lab Digital Forensic dengan Kali Linux & VirtualBox**  

---

#### **Pengantar**  
Hari pertama perjalanan 30 hari belajar Digital Forensic dimulai dengan fondasi terpenting: **membangun lingkungan lab virtual yang aman dan terisolasi**. Lab ini akan menjadi "ruang operasi" Anda untuk semua praktik forensik, mulai dari analisis disk hingga penyelidikan malware. Tujuan hari ini:  
- Memahami konsep dasar Digital Forensic (teori 20%).  
- Menginstal dan mengonfigurasi lab virtual dengan **Kali Linux** dan **VirtualBox** (praktik 80%).  

---

### **Bagian 1: Teori (20%) - Konsep Dasar Digital Forensic**  
Sebelum praktik, pahami 4 pilar fundamental Digital Forensic:  

#### **1. Siklus Hidup Forensik (Forensic Lifecycle)**  
Proses standar dalam investigasi digital:  
1. **Identifikasi**: Menentukan sumber bukti (perangkat, jaringan, cloud).  
2. **Preservasi**: Mengamankan bukti dengan *write-blocker* dan hashing (MD5/SHA).  
3. **Analisis**: Mengekstrak data menggunakan tools forensik.  
4. **Pelaporan**: Mendokumentasikan temuan secara legal.  

#### **2. Legalitas & Etika**  
- **Legalitas**: Investigasi harus mematuhi hukum (e.g., UU ITE di Indonesia).  
- **Etika**:  
  - Jangan mengubah data asli (*integrity*).  
  - Gunakan *chain of custody* untuk dokumentasi.  
  - Hindari *conflict of interest* (e.g., menyelidiki data pribadi tanpa izin).  

#### **3. Mengapa Lab Virtual?**  
- **Keamanan**: Isolasi dari sistem utama (mencegah kontaminasi data).  
- **Reproduksibilitas**: Simulasi kasus berulang kali dengan snapshot.  
- **Kostumisasi**: Install tools forensik tanpa risiko kerusakan OS.  

---

### **Bagian 2: Praktik (80%) - Instalasi Lab Virtual**  
#### **Tools yang Dibutuhkan**  
1. **VirtualBox** (Virtualisasi gratis): [Download di sini](https://www.virtualbox.org/wiki/Downloads).  
2. **Kali Linux** (OS forensik): [Download ISO terbaru](https://www.kali.org/get-kali/).  
3. **Spesifikasi Minimum**:  
   - RAM: 4GB (rekomendasi 8GB).  
   - Storage: 25GB (untuk install Kali + tools).  

---

#### **Langkah demi Langkah Instalasi**  
##### **Step 1: Install VirtualBox**  
1. Download VirtualBox versi terbaru untuk OS Anda (Windows/macOS/Linux).  
2. Jalankan installer → ikuti wizard (klik *Next* hingga selesai).  
3. Buka VirtualBox → klik *New* untuk membuat VM baru.  

##### **Step 2: Konfigurasi VM untuk Kali Linux**  
1. **Nama & OS**:  
   - Nama: `Kali-Forensic-Lab`  
   - Type: `Linux`  
   - Version: `Debian (64-bit)`  
2. **Memory Size**:  
   - Set minimal **4096 MB** (jika RAM PC ≥ 8GB).  
3. **Hard Disk**:  
   - Pilih *Create a virtual hard disk now*.  
   - Type: `VDI (VirtualBox Disk Image)`.  
   - Storage: **25 GB** (dynamically allocated).  

##### **Step 3: Install Kali Linux di VM**  
1. Pilih VM `Kali-Forensic-Lab` → klik *Settings* → *Storage*.  
2. Klik ikon disk (Controller: IDE) → *Choose a disk file* → pilih file ISO Kali Linux.  
3. Start VM → Kali akan boot dari ISO.  
4. Di menu install:  
   - Pilih *Graphical Install*.  
   - Language: `English` (rekomendasi untuk kompatibilitas tools).  
   - Keyboard: `American English`.  
5. **Partition Disk**:  
   - Pilih *Guided - use entire disk*.  
   - Pilih disk virtual yang dibuat (e.g., `/dev/sda`).  
   - Pilih *All files in one partition*.  
6. **User & Password**:  
   - Hostname: `kali-forensic`  
   - Username: `forensik` (hindari `root` untuk keamanan).  
   - Password: Buat password kuat (minimal 12 karakter).  
7. Tunggu hingga install selesai → reboot.  

##### **Step 4: Konfigurasi Awal Kali Linux**  
1. **Update System**:  
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```  
2. **Install Tools Forensik Dasar**:  
   ```bash
   sudo apt install -y autopsy sleuthkit wireshark volatility-tools bulk-extractor
   ```  
3. **Buat Snapshot**:  
   - Matikan VM → klik *Snapshots* → *Take* → beri nama `Base Install`.  
   - *Snapshot* memungkinkan Anda mengembalikan VM ke kondisi awal jika rusak.  

##### **Step 5: Verifikasi Lab**  
1. Cek tools forensik:  
   ```bash
   autopsy --version  # Harus muncul versi Autopsy
   volatility --info  # Harus muncul daftar plugin
   ```  
2. Test jaringan:  
   ```bash
   ping google.com  # Pastikan terhubung internet
   ```  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
1. **Dokumentasi**:  
   - Buat laporan singkat (1 halaman) dengan template:  
     ```markdown
     ## Laporan Hari ke-1: Setup Lab Virtual  
     ### Teori  
     - Jelaskan siklus hidup forensik dalam 3 kalimat.  
     - Sebutkan 2 etika penting dalam Digital Forensic.  
     ### Praktik  
     - Screenshot tampilan VirtualBox + Kali Linux.  
     - Bukti instalasi tools (output `autopsy --version`).  
     ### Kesulitan & Solusi  
     - [Deskripsi masalah yang dihadapi + cara mengatasi]  
     ```  
2. **Eksperimen Tambahan**:  
   - Coba buat snapshot baru → install aplikasi tambahan (e.g., GIMP) → restore ke `Base Install`. Amati perubahan!  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| VM lambat | Tambah RAM di *Settings* → *System* → *Motherboard*. |  
| Kali tidak boot | Cek file ISO di *Settings* → *Storage*. Pastikan terdeteksi. |  
| Error saat update | Jalankan `sudo dpkg --configure -a` → ulangi `apt upgrade`. |  

---

### **Referensi**  
1. [Official Kali Linux Docs](https://www.kali.org/docs/)  
2. [VirtualBox User Manual](https://www.virtualbox.org/manual/UserManual.html)  
3. [NIST Guide to Digital Forensics](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-86.pdf)  

---

### **Kesimpulan**  
Hari pertama ini adalah **investasi krusial** untuk 29 hari ke depan. Dengan lab virtual yang siap, Anda bisa:  
- Berlatih analisis malware tanpa risiko infeksi.  
- Uji coba tools forensik secara bebas.  
- Simulasi kasus nyata dengan dataset.  

**Pesan Penting**:  
> *"Dalam Digital Forensic, integritas lab adalah segalanya. Jangan pernah skip langkah dokumentasi atau snapshot!"*  

Siap untuk Hari ke-2? Kita akan mempelajari struktur disk dan partisi dengan hands-on menggunakan tools CLI! 💻🔍
