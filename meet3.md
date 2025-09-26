

### **Artikel Praktikum: Hari ke-3 - Disk Imaging dengan `dd` dan FTK Imager**  

---

#### **Pengantar**  
Hari ketiga kita mempelajari **disk imaging** - proses krusial dalam Digital Forensic untuk membuat salinan identik (bit-by-bit) dari media penyimpanan. Ini adalah langkah wajib sebelum analisis untuk:  
- **Preservasi bukti asli** (tanpa modifikasi)  
- **Menciptakan duplikat forensik** yang dapat dianalisis berulang kali  
- **Memenuhi standar legal** (chain of custody)  

**Tujuan Hari Ini**:  
- Teori (20%): Memahami metode imaging, write-blocker, dan hashing.  
- Praktik (80%): Membuat disk image dengan `dd` (CLI) dan FTK Imager (GUI).  

---

### **Bagian 1: Teori (20%) - Konsep Disk Imaging**  
#### **1. Metode Imaging**  
| Metode       | Deskripsi                          | Use Case               |  
|--------------|------------------------------------|------------------------|  
| **Dead Imaging** | Dilakukan saat device mati (offline) | Hard drive, USB, SSD    |  
| **Live Imaging** | Dilakukan saat sistem aktif        | Server, RAM, cloud     |  

#### **2. Write-Blocker**  
- **Fungsi**: Mencegah penulisan ke media asli selama imaging.  
- **Jenis**:  
  - Hardware: Tab khusus (e.g., Tableau) - wajib untuk kasus legal.  
  - Software: `dd` dengan opsi `iflag=direct` (simulasi).  

#### **3. Hashing (Verifikasi Integritas)**  
- **Algoritma**: MD5, SHA-1, SHA-256.  
- **Proses**:  
  ```mermaid
  graph LR
    A[Disk Asli] -->|Hitung Hash| B[Hash Value A]
    C[Disk Image] -->|Hitung Hash| D[Hash Value B]
    B -->|Bandingkan| D
    D -->|Sama?| E[Image Valid]
    D -->|Beda?| F[Gagal Imaging]
  ```  

---

### **Bagian 2: Praktik (80%) - Membuat Disk Image**  
#### **Tools yang Digunakan**  
- **`dd`**: Tool CLI Unix untuk imaging (built-in di Kali Linux).  
- **FTK Imager**: Tool GUI profesional (gratis untuk versi lite).  
- **Target**: USB drive (minimal 4GB) atau file virtual disk.  

---

#### **Langkah 1: Persiapan Media Target**  
1. **Identifikasi Device ID**:  
   ```bash
   sudo fdisk -l
   ```  
   Contoh output untuk USB:  
   ```plaintext
   Disk /dev/sdb: 7.5 GiB, 8053063680 bytes, 15728640 sectors
   ```  
   ⚠️ **PENTING**: Pastikan `/dev/sdb` adalah device yang benar (jangan salah pilih disk utama!).  

2. **Unmount Partisi**:  
   ```bash
   sudo umount /dev/sdb*
   ```  

---

#### **Langkah 2: Imaging dengan `dd` (CLI)**  
1. **Buat Image dengan Hashing**:  
   ```bash
   sudo dd if=/dev/sdb of=~/forensic_images/usb_image.dd bs=4M conv=noerror,sync status=progress iflag=direct
   ```  
   - **Parameter**:  
     - `if`: Input file (device asli).  
     - `of`: Output file (image).  
     - `bs=4M`: Block size 4MB (lebih cepat).  
     - `conv=noerror,sync`: Lanjutkan meski ada error.  
     - `iflag=direct`: Akses langsung tanpa cache (simulasi write-blocker).  

2. **Hitung Hash Image**:  
   ```bash
   sha256sum ~/forensic_images/usb_image.dd > ~/forensic_images/usb_image.sha256
   cat ~/forensic_images/usb_image.sha256
   ```  

3. **Verifikasi Integritas**:  
   ```bash
   sudo sha256sum /dev/sdb
   ```  
   Bandingkan hasilnya dengan hash di `usb_image.sha256`.  

---

#### **Langkah 3: Imaging dengan FTK Imager (GUI)**  
1. **Install FTK Imager**:  
   ```bash
   wget https://download.accessdata.com/ftkimager_4.7.1_amd64.deb
   sudo dpkg -i ftkimager_4.7.1_amd64.deb
   sudo apt install -f  # Fix dependencies
   ```  

2. **Buat Image**:  
   - Buka FTK Imager → `File` → `Create Disk Image`.  
   - **Source**: Pilih `Physical Drive` → `/dev/sdb`.  
   - **Image Format**: Pilih `DD` (raw image).  
   - **Destination**:  
     - Browse folder `~/forensic_images/`.  
     - Nama file: `usb_image_ftk.dd`.  
   - **Verify**: Centang `Verify image after creation`.  
   - Klik `Start` → tunggu proses selesai.  

3. **Cek Hash di FTK Imager**:  
   - Setelah imaging, FTK akan menampilkan hash otomatis.  
   - Bandingkan dengan hasil dari `dd`.  

---

#### **Langkah 4: Simulasi Write-Blocker dengan `dd`**  
Untuk memahami risiko tanpa write-blocker:  
```bash
# Coba tulis ke disk asli (HANYA UNTUK PEMBELAJARAN!)
echo "TEST" | sudo dd of=/dev/sdb bs=1 count=4 conv=notrunc
```  
**Apa yang Terjadi?**:  
- Data asli di disk terkorupsi!  
- Ini sebabnya **write-blocker hardware wajib** di kasus nyata.  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Anda menerima USB drive dari saksi mata. Buat forensic image dan verifikasi integritasnya. Dokumentasikan proses sesuai standar legal."*  

#### **Langkah Pengerjaan**:  
1. **Imaging**:  
   - Buat 2 image: 1 dengan `dd`, 1 dengan FTK Imager.  
   - Hitung hash untuk masing-masing image.  

2. **Verifikasi**:  
   - Bandingkan hash antara image dan disk asli.  
   - Jika berbeda, ulangi proses!  

3. **Laporan**:  
   ```markdown
   ## Laporan Hari ke-3: Disk Imaging  
   ### Metadata  
   - Tanggal: [tanggal]  
   - Operator: [nama]  
   - Target: /dev/sdb (USB 8GB)  
   ### Proses Imaging  
   1. **dd Command**:  
      ```bash
      [paste command dd]
      ```  
   2. **FTK Imager**:  
      - Screenshot proses imaging.  
   ### Verifikasi  
   - Hash Disk Asli: [hash]  
   - Hash Image (dd): [hash]  
   - Hash Image (FTK): [hash]  
   ### Kesimpulan  
   - Status: [Valid/Invalid]  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `dd: Permission denied` | Gunakan `sudo` dan pastikan device tidak ter-mount. |  
| Hash tidak cocok | Cek koneksi USB (gunakan port langsung, bukan hub). |  
| FTK Imager tidak detect device | Jalankan sebagai root: `sudo ftkimager`. |  
| Proses `dd` lambat | Tambah `bs=64M` untuk mempercepat. |  

---

### **Referensi**  
1. [NIST Guide to Disk Imaging](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-86.pdf)  
2. [FTK Imager Documentation](https://www.exterro.com/ftk-imager-user-guide)  
3. [Linux `dd` Man Page](https://man7.org/linux/man-pages/man1/dd.1.html)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Membuat forensic image dengan 2 metode (CLI dan GUI).  
✅ Memverifikasi integritas image menggunakan hashing.  
✅ Memahami risiko tanpa write-blocker.  

**Pesan Penting**:  
> *"Dalam Digital Forensic, satu byte yang berubah bisa membuat bukti tidak sah di pengadilan. Selalu verifikasi hash!"*  

Siap untuk Hari ke-4? Kita akan mempelajari **Chain of Custody** dan dokumentasi forensik! 📜🔒
