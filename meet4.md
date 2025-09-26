

### **Artikel Praktikum: Hari ke-4 - Chain of Custody & Dokumentasi Forensik**  

---

#### **Pengantar**  
Hari keempat kita fokus pada **aspek krusial yang sering diabaikan oleh praktisi pemula**: *Chain of Custody* (CoC) dan dokumentasi forensik. Tanpa CoC yang solid, bukti digital sekalipun yang dianalisis dengan sempurna bisa **ditolak di pengadilan**.  

**Tujuan Hari Ini**:  
- Teori (20%): Memahami konsep CoC, standar legal, dan elemen dokumentasi.  
- Praktik (80%): Simulasi proses penanganan bukti dari lokasi kejadian hingga lab.  

---

### **Bagian 1: Teori (20%) - Fundamentals Chain of Custody**  
#### **1. Apa itu Chain of Custody?**  
**Definisi**: Dokumentasi terstruktur yang melacak setiap perpindahan, akses, dan modifikasi bukti digital dari **saat ditemukan** hingga **disajikan di pengadilan**.  

**Mengapa Penting?**  
- **Integritas Bukti**: Mencegah tuduhan pemalsuan atau kontaminasi.  
- **Akuntabilitas**: Setiap orang yang menyentuh bukti tercatat.  
- **Legalitas**: Syarat wajib untuk bukti diterima di pengadilan (e.g., di Indonesia: Pasal 5 KUHAP).  

#### **2. Elemen Kunci dalam CoC**  
| Elemen              | Deskripsi                                                                 | Contoh                          |  
|---------------------|---------------------------------------------------------------------------|---------------------------------|  
| **Identifikasi**    | Label unik bukti (barcode/ID)                                            | `CASE-2024-001-USB-01`          |  
| **Pengumpulan**     | Siapa, kapan, di mana, dan bagaimana bukti dikumpulkan                    | Officer Budi, 1 Juni 2024, 10:00 WIB |  
| **Transfer**        | Setiap perpindahan bukti antar pihak (dengan tanda tangan)                | Dari TKP → Lab Forensik         |  
| **Penyimpanan**     | Kondisi dan lokasi penyimpanan (terkunci, suhu terkontrol)                | Brankas Lab, Nomor Locker: A-7  |  
| **Analisis**        | Siapa, kapan, dan tools yang digunakan untuk analisis                     | Analyst Siti, 2 Juni 2024, Autopsy |  
| **Pengembalian**    | Status akhir bukti (dikembalikan/disimpan/dihancurkan)                    | Dikembalikan ke pemilik         |  

#### **3. Standar Internasional**  
- **ISO/IEC 27037**: Pandangan penanganan bukti digital.  
- **NIST SP 800-86**: Template dokumentasi forensik.  
- **ACPO Good Practice Guide**: Standar investigasi cybercrime di Inggris.  

---

### **Bagian 2: Praktik (80%) - Simulasi Chain of Custody**  
#### **Tools yang Digunakan**  
- **Template CoC**: [NIST Chain of Custody Form](https://www.nist.gov/document/template-digital-forensic-report)  
- **Software**: PDF Editor (e.g., LibreOffice Draw) atau platform seperti [ChainCustody](https://www.chaincustody.com/).  
- **Simulasi**: USB drive sebagai "bukti digital".  

---

#### **Langkah 1: Simulasi Penemuan Bukti di TKP**  
**Skenario**:  
> *"Sebuah USB drive ditemukan di meja kerja tersangka kasus kebocoran data. Anda adalah officer pertama yang menanganinya."*  

**Proses**:  
1. **Identifikasi Bukti**:  
   - Tempelkan label pada USB: `CASE-2024-001-USB-01`.  
   - Foto USB dari 5 sudut (depan, belakang, sisi kiri/kanan, port USB).  

2. **Isi Form CoC (Bagian Pengumpulan)**:  
   ```markdown
   ## CHAIN OF CUSTODY - CASE-2024-001
   **Bukti**: USB Flashdisk (Merah, 32GB)  
   **ID Bukti**: CASE-2024-001-USB-01  
   **Dikumpulkan Oleh**: Officer John Doe (Badge: 123)  
   **Tanggal/Waktu**: 1 Juni 2024, 10:15 WIB  
   **Lokasi**: Ruang Kerja Tersangka, Lantai 5, Gedung X  
   **Kondisi Awal**:  
   - Fisik: Tidak ada kerusakan, terpasang di laptop.  
   - Digital: LED menyala (terdeteksi oleh OS).  
   ```  

---

#### **Langkah 2: Transfer ke Lab Forensik**  
**Prosedur**:  
1. **Bungkus Bukti**:  
   - Masukkan USB ke *anti-static bag* → segel dengan *evidence tape*.  
   - Tulis ID bukti di segel: `CASE-2024-001-USB-01`.  

2. **Isi Form CoC (Bagian Transfer)**:  
   ```markdown
   ### TRANSFER #1  
   **Dari**: Officer John Doe  
   **Ke**: Lab Forensik (Penerima: Analyst Siti)  
   **Tanggal/Waktu**: 1 Juni 2024, 14:30 WIB  
   **Alasan**: Untuk imaging dan analisis  
   **Metode Pengiriman**: Diantar langsung (dengan kendaraan dinas)  
   **Kondisi Saat Transfer**:  
   - Segel utuh, tidak ada tanda kerusakan.  
   - Tanda Tangan Penerima: [_____________________]  
   ```  

---

#### **Langkah 3: Proses di Lab Forensik**  
**Aktivitas**:  
1. **Imaging**:  
   - Buat forensic image dengan `dd` (seperti Hari ke-3).  
   - Catat di CoC:  
     ```markdown
     ### ANALISIS  
     **Tanggal/Waktu**: 2 Juni 2024, 09:00-11:00 WIB  
     **Analyst**: Siti (ID: LAB-456)  
     **Tools**: FTK Imager v4.7.1, dd (Linux)  
     **Hash Asli**: SHA256: a1b2c3...  
     **Hash Image**: SHA256: a1b2c3... (MATCH)  
     ```  

2. **Penyimpanan**:  
   - Simpan USB asli di brankas (Locker A-7).  
   - Catat di CoC:  
     ```markdown
     ### PENYIMPANAN  
     **Lokasi**: Brankas Lab Forensik, Locker A-7  
     **Kondisi**:  
     - Suhu: 18-22°C  
     - Kelembaban: 40-60%  
     - Akses: Hanya authorized personnel  
     ```  

---

#### **Langkah 4: Pengembalian Bukti**  
**Prosedur**:  
1. **Selesai Analisis**:  
   - Bukti siap dikembalikan ke penyidik.  
   - Isi bagian akhir CoC:  
     ```markdown
     ### PENGEMBALIAN  
     **Dikembalikan Kepada**: Officer John Doe  
     **Tanggal/Waktu**: 5 Juni 2024, 10:00 WIB  
     **Alasan**: Analisis selesai, bukti tidak diperlukan lagi  
     **Kondisi Akhir**:  
     - Segel utuh, tidak ada modifikasi fisik/digital.  
     - Tanda Tangan Penerima: [_____________________]  
     ```  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Anda menerima laptop dari TKP kasus pencurian data. Lakukan simulasi CoC lengkap dari penemuan hingga pengembalian."*  

**Instruksi**:  
1. **Download Template**: [NIST CoC Form](https://www.nist.gov/document/template-digital-forensic-report) (modifikasi sesuai kebutuhan).  
2. **Simulasi**:  
   - Gunakan laptop Anda sebagai "bukti".  
   - Ambil foto laptop dari 5 sudut.  
   - Isi semua bagian CoC (pengumpulan → transfer → analisis → pengembalian).  
3. **Buat Laporan**:  
   - Export ke PDF dengan nama: `CoC_Day4_Report.pdf`.  
   - Struktur:  
     ```markdown
     ## Laporan Chain of Custody  
     ### Identitas Kasus  
     - Nomor Kasus: [buat sendiri, e.g., CASE-2024-002]  
     - Jenis Bukti: Laptop [merk/tipe]  
     ### Dokumentasi CoC  
     [Lampirkan form CoC yang sudah diisi]  
     ### Bukti Foto  
     [Lampirkan 5 foto laptop]  
     ### Verifikasi  
     - Tanda tangan digital (e.g., dengan [DocuSign](https://www.docusign.com/) atau tanda tangan basah).  
     ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| Lupa mencatat timestamp | Gunakan aplikasi timestamp (e.g., [Timestamp Camera](https://play.google.com/store/apps/details?id=com.jeyluta.timestampcamera)) saat pengambilan foto. |  
| Tidak ada saksi saat transfer | Libatkan minimal 2 orang untuk setiap transfer (1 pengirim + 1 penerima). |  
| Bukti rusak saat penyimpanan | Gunakan *anti-static bag* dan kontrol lingkungan (suhu/kelembaban). |  

---

### **Referensi**  
1. [NIST SP 800-86: Guide to Integrating Forensic Techniques](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-86.pdf)  
2. [ISO/IEC 27037:2012 Guidelines for Digital Evidence](https://www.iso.org/standard/44381.html)  
3. [ACPO Good Practice Guide for Digital Evidence](https://www.digital-detective.net/digital-forensics-documents/ACPO_Good_Practice_Guide_for_Digital_Evidence_v4.pdf)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami pentingnya Chain of Custody dalam investigasi digital.  
✅ Membuat dokumentasi forensik yang memenuhi standar legal.  
✅ Mensimulasikan proses penanganan bukti dari TKP hingga lab.  

**Pesan Penting**:  
> *"Dalam Digital Forensic, bukti tanpa dokumentasi yang valid hanyalah sekumpulan data. Chain of Custody adalah jembatan antara teknis dan legal."*  

Siap untuk Hari ke-5? Kita akan mempelajari **File System Metadata Analysis** dengan The Sleuth Kit! 🕵️‍♂️📁
