

### **Artikel Praktikum: Hari ke-6 - Pemulihan Data dengan Autopsy dan PhotoRec**  

---

#### **Pengantar**  
Hari keenam kita mempelajari **teknik pemulihan data** - salah satu skill paling krusial dalam Digital Forensic. Ketika file dihapus, sebenarnya data fisiknya masih ada di disk hingga ditimpa data baru. Hari ini kita akan:  
- **Memulihkan file terhapus** dari disk image  
- **Membandingkan 2 pendekatan**: automated (Autopsy) vs manual carving (PhotoRec)  
- **Memahami batasan recovery** (fragmentasi, overwriting)  

**Tujuan Hari Ini**:  
- Teori (20%): Konsep unallocated space, file carving, dan mekanisme penghapusan file.  
- Praktik (80%): Recovery file terhapus menggunakan Autopsy dan PhotoRec.  

---

### **Bagian 1: Teori (20%) - Mekanisme Penghapusan File**  
#### **1. Apa yang Terjadi Saat File Dihapus?**  
| File System | Proses Penghapusan                                                                 |  
|-------------|------------------------------------------------------------------------------------|  
| **NTFS**    | - MFT entry ditandai "unused"<br>- $Bitmap membebaskan clusters<br>- Data tetap ada hingga ditimpa |  
| **FAT32**   | - Byte pertama nama file diubah jadi `0xE5`<br>- FAT chain dilepas<br>- Data tetap ada           |  
| **ext4**    | - inode ditandai "free"<br>- Block pointer dilepas<br>- Data tetap ada                      |  

#### **2. Unallocated Space**  
- **Definisi**: Area disk yang tidak teralokasi ke file aktif.  
- **Kandungan**:  
  - File terhapus  
  - Slack space (sisa data di akhir cluster)  
  - Fragmentasi file  

#### **3. File Carving**  
- **Konsep**: Recovery file berdasarkan **header/footer signature**, bukan metadata.  
- **Contoh Signature**:  
  | Tipe File | Header (Hex) | Footer (Hex) |  
  |-----------|--------------|--------------|  
  | JPEG      | `FF D8 FF`   | `FF D9`      |  
  | PDF       | `25 50 44 46`| `0A 25 25 45`|  
  | ZIP       | `50 4B 03 04`| `50 4B 05 06`|  

---

### **Bagian 2: Praktik (80%) - Pemulihan Data**  
#### **Tools yang Digunakan**  
- **Autopsy**: GUI tool untuk automated recovery (dengan TSK backend).  
- **PhotoRec**: CLI tool untuk file carving (bagian dari TestDisk).  
- **Dataset**: [Deleted Files Image](https://digitalcorpora.org/corpora/drives/nps-2009-ubnist1/) (download `nps-2009-ubnist1-gen2.E01`).  

---

#### **Langkah 1: Pemulihan Otomatis dengan Autopsy**  
1. **Buat Kasus Baru**:  
   ```bash
   autopsy  # Buka Autopsy di browser (http://localhost:9999)
   ```  
   - **Case Name**: `Day6_Recovery`  
   - **Data Source**: Tambah file `nps-2009-ubnist1-gen2.E01`  

2. **Konfigurasi Ingest Modules**:  
   - Centang:  
     - ✅ **File Type Identification** (untuk deteksi header)  
     - ✅ **Hash Lookup** (untuk filter file known good/bad)  
     - ✅ **Deleted Files** (fokus hari ini)  

3. **Analisis Hasil**:  
   - Navigasi ke **File Analysis** → Filter:  
     - `Deleted Files: Yes`  
     - `File Type: Documents`  
   - Contoh file yang ditemukan:  
     ```plaintext
     /Documents/secret_plan.doc (Deleted)
     /Pictures/vacation.jpg (Deleted)
     ```  

4. **Ekstrak File**:  
   - Klik kanan file → **Extract Files** → Simpan di folder `recovered_autopsy`.  

---

#### **Langkah 2: Pemulihan Manual dengan PhotoRec**  
1. **Install PhotoRec**:  
   ```bash
   sudo apt install testdisk -y  # PhotoRec termasuk dalam paket TestDisk
   ```  

2. **Jalankan PhotoRec**:  
   ```bash
   photorec nps-2009-ubnist1-gen2.E01
   ```  

3. **Konfigurasi Interaktif**:  
   | Prompt | Pilihan | Keterangan |  
   |--------|---------|------------|  
   | `Proceed` | `[Continue]` | Lanjutkan meski ada error |  
   | `Partition` | `[Whole disk]` | Scan seluruh disk |  
   | `Filesystem` | `[Other]` | Non-filesystem mode (raw carving) |  
   | `File types` | `[doc,jpg,pdf]` | Pilih tipe file target |  
   | `Destination` | `/home/user/recovered_photorec` | Folder output |  

4. **Proses Recovery**:  
   - PhotoRec akan scan dan recover file berdasarkan signature.  
   - File hasil recovery akan dinamai:  
     ```plaintext
     f123456.doc  (file DOC)
     f123457.jpg  (file JPEG)
     ```  

---

#### **Langkah 3: Verifikasi & Perbandingan**  
1. **Cek Integritas File**:  
   ```bash
   # Verifikasi file JPEG
   file recovered_autopsy/vacation.jpg
   file recovered_photorec/f123457.jpg

   # Hitung hash untuk perbandingan
   sha256sum recovered_autopsy/secret_plan.doc
   sha256sum recovered_photorec/f123456.doc
   ```  

2. **Bandingkan Hasil**:  
   | Tool | Jumlah File Ditemukan | Kelebihan | Kekurangan |  
   |------|----------------------|-----------|------------|  
   | Autopsy | 15 file | - Integrasi metadata<br>- Filter otomatis | - Lambat untuk image besar<br>- Bergantung pada file system |  
   | PhotoRec | 23 file | - Cepat<br>- Bisa recovery dari file system rusak | - Tidak ada metadata<br>- Banyak false positive |  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Seorang pegawai menghapus file kontrak rahasia sebelum resign. Anda menerima disk image `employee_disk.E01`. Pulihkan file kontrak dan dokumen terkait."*  

**Instruksi**:  
1. **Download Dataset**: [Employee Disk Image](https://digitalcorpora.org/corpora/drives/nps-2009-gen2/) (gunakan `nps-2009-ubnist1-gen2.E01`).  
2. **Recovery**:  
   - Gunakan Autopsy untuk cari file `.doc` dan `.pdf` terhapus.  
   - Gunakan PhotoRec untuk recovery file dengan signature yang sama.  
3. **Laporan**:  
   ```markdown
   ## Laporan Pemulihan Data  
   ### Dataset  
   - Image: employee_disk.E01  
   - Ukuran: 2GB  
   ### Hasil Recovery  
   **Autopsy**:  
   - File ditemukan: [nama file]  
   - Status: [utuh/rusak]  
   **PhotoRec**:  
   - File ditemukan: [nama file]  
   - Status: [utuh/rusak]  
   ### Analisis  
   - Alasan perbedaan hasil: [jelaskan]  
   - File paling penting: [nama + hash]  
   ### Bukti  
   - Screenshot Autopsy (daftar file terhapus)  
   - Screenshot PhotoRec (proses scanning)  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| Autopsy lambat | Kurangi ingest modules (non-aktifkan "Email Parsing" jika tidak diperlukan). |  
| PhotoRec tidak detect image | Konversi E01 ke DD dengan `ewfexport`:  
  ```bash
  ewfexport -f dd -o employee_disk.dd employee_disk.E01
  ``` |  
| File hasil recovery rusak | Coba carving dengan `foremost` (alternatif PhotoRec):  
  ```bash
  foremost -t doc,jpg,pdf -i employee_disk.dd -o foremost_output
  ``` |  

---

### **Referensi**  
1. [Autopsy Documentation](https://www.autopsy.com/docs/)  
2. [PhotoRec Step-by-Step](https://www.cgsecurity.org/wiki/PhotoRec_Step_By_Step)  
3. [File Signature Database](https://www.filesignatures.net/index.php?page=search)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami mekanisme penghapusan file di berbagai file system.  
✅ Memulihkan file terhapus dengan 2 metode berbeda.  
✅ Mengevaluasi kelebihan/kekurangan setiap tool.  

**Pesan Penting**:  
> *"Data yang 'terhapus' tidak benar-benar hilang - ia hanya menunggu ditemukan oleh analis yang tepat. Tapi ingat: semakin lama Anda menunggu, semakin besar risiko data ditimpa!"*  

Siap untuk Hari ke-7? Kita akan mempelajari **Timeline Analysis** dengan Timesketch! ⏱️🕵️‍♂️
