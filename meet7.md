

### **Artikel Praktikum: Hari ke-7 - Timeline Analysis dengan Timesketch**  

---

#### **Pengantar**  
Hari ketujuh kita mempelajari **timeline analysis** - teknik powerful untuk merekonstruksi aktivitas digital secara kronologis. Timeline mengubah "jejak digital" yang terpisah-pisah menjadi narasi investigasi yang koheren, memungkinkan kita:  
- **Menghubungkan kejadian tersembunyi** (e.g., malware installation → file deletion → network connection)  
- **Mendeteksi anomali** (aktivitas di luar jam kerja, akses file mencurigakan)  
- **Memvisualisasikan pola perilaku pengguna**  

**Tujuan Hari Ini**:  
- Teori (20%): Konsep timeline, jenis event, dan metode rekonstruksi.  
- Praktik (80%): Membuat timeline dari disk image menggunakan Timesketch.  

---

### **Bagian 1: Teori (20%) - Fundamentals Timeline Analysis**  
#### **1. Apa itu Timeline Forensik?**  
**Definisi**: Representasi kronologis semua aktivitas digital yang tercatat di sistem, disusun berdasarkan timestamp.  

**Mengapa Penting?**  
- **Rekonstruksi Kejadian**: Menghubungkan titik-titik terpisah menjadi cerita lengkap.  
- **Deteksi Anomali**: Menemukan aktivitas di luar pola normal (e.g., akses file jam 3 pagi).  
- **Validasi Alibi**: Membuktikan/membantah klaim pengguna.  

#### **2. Jenis Timeline**  
| Jenis Timeline | Sumber Data                          | Contoh Event                     |  
|---------------|--------------------------------------|----------------------------------|  
| **Filesystem** | $MFT, Inode, USN Journal             | File creation, deletion, rename  |  
| **Application** | Browser history, logs, registry      | Website visit, software install |  
| **Network**    | Firewall logs, packet capture       | Connection, data transfer       |  
| **Memory**     | Process list, network connections    | Process execution, DLL load      |  

#### **3. Struktur Timeline Data**  
Setiap event dalam timeline memiliki:  
```plaintext
[Timestamp] [Event Type] [Source] [Description] [Extra Data]
```  
- **Timestamp**: Waktu presisi (UTC preferred).  
- **Event Type**: Kategori (e.g., `file_created`, `reg_modified`).  
- **Source**: Aplikasi/file system yang mencatat event.  
- **Description**: Ringkasan aktivitas.  
- **Extra Data**: Metadata tambahan (e.g., file path, user, hash).  

---

### **Bagian 2: Praktik (80%) - Membuat Timeline dengan Timesketch**  
#### **Tools yang Digunakan**  
- **Timesketch**: Open-source timeline analysis tool (dengan GUI web).  
- **Dataset**: [NPS Corporation Image](https://digitalcorpora.org/corpora/drives/nps-2009-ubnist1/) (download `nps-2009-ubnist1-gen2.E01`).  

---

#### **Langkah 1: Install Timesketch**  
```bash
# Install Docker (jika belum)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Timesketch via Docker
docker pull us.gcr.io/osdfir/timesketch:latest
docker run -p 5000:5000 --name timesketch -d us.gcr.io/osdfir/timesketch:latest

# Akses Timesketch di browser
http://localhost:5000
```  
**Default Login**:  
- Username: `admin`  
- Password: `password`  

---

#### **Langkah 2: Buat Timeline dari Disk Image**  
1. **Konversi E01 ke DD**:  
   ```bash
   docker exec -it timesketch bash
   apt update && apt install -y ewf-tools
   ewfexport -f dd -o /tmp/nps.dd /mnt/nps-2009-ubnist1-gen2.E01
   exit
   ```  

2. **Generate Timeline dengan `log2timeline`**:  
   ```bash
   docker exec -it timesketch log2timeline.py \
     --partition_offset=2048 \
     --vss_stores=all \
     --temporary_directory=/tmp \
     /tmp/timeline.plaso \
     /tmp/nps.dd
   ```  
   - **Parameter Penting**:  
     - `--partition_offset=2048`: Offset partisi NTFS.  
     - `--vss_stores=all`: Include Volume Shadow Copies (untuk file history).  

3. **Import Timeline ke Timesketch**:  
   - Di web UI Timesketch:  
     - Klik **Create New Sketch** → Beri nama `Day7_Timeline`.  
     - Pilih **Add Timeline** → Upload file `timeline.plaso`.  

---

#### **Langkah 3: Analisis Timeline**  
1. **Filter Event Penting**:  
   - Di **Search Bar**, gunakan query:  
     ```sql
     event_type:"file_created" OR event_type:"file_deleted" OR event_type:"reg_modified"
     ```  
   - Tambahkan filter waktu:  
     ```sql
     date:"2009-01*"  # Fokus bulan Januari 2009
     ```  

2. **Visualisasi Timeline**:  
   - Pilih tab **Graph** → Pilih **Timeline**.  
   - Zoom ke periode dengan event padat (indikasi aktivitas mencurigakan).  

3. **Analisis Pola**:  
   - Cari event mencurigakan:  
     ```sql
     filename:"*.exe" AND event_type:"file_created"
     ```  
   - Contoh temuan:  
     ```plaintext
     2009-01-15 14:23:45 | file_created | C:\Temp\backdoor.exe | User: Administrator
     ```  

---

#### **Langkah 4: Rekonstruksi Aktivitas**  
1. **Buat Storyline**:  
   - Di Timesketch, gunakan **Sketch** untuk membuat narasi:  
     ```markdown
     ## Aktivitas Mencurigakan  
     ### 2009-01-15 14:00-15:00  
     - 14:23:45: File `backdoor.exe` dibuat di C:\Temp  
     - 14:25:10: Registry key `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` dimodifikasi  
     - 14:26:33: Koneksi network ke IP 192.168.1.100:4444  
     ```  

2. **Korelasikan Event**:  
   - Gunakan **Graph Analysis** untuk hubungkan event:  
     ```sql
     source:"NTFS" AND data_type:"fs:stat" AND timestamp:"2009-01-15T14:23*"
     ```  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Sebuah sistem disusupi malware. Analisis timeline untuk:  
> 1. Kapan malware pertama kali masuk?  
> 2. File apa yang dimodifikasi/dihapus?  
> 3. Apakah ada aktivitas network mencurigakan?"*  

**Instruksi**:  
1. **Download Dataset**: [NPS Image](https://digitalcorpora.org/corpora/drives/nps-2009-ubnist1/).  
2. **Analisis**:  
   - Buat timeline dengan Timesketch.  
   - Fokus ke periode 15-20 Januari 2009.  
3. **Laporan**:  
   ```markdown
   ## Laporan Timeline Analysis  
   ### Timeline Utama  
   - **Malware Entry**: [tanggal + waktu]  
     - Event: [deskripsi]  
   - **File Modification**: [tanggal + waktu]  
     - File: [nama file]  
   - **Network Activity**: [tanggal + waktu]  
     - Destination: [IP/port]  
   ### Kesimpulan  
   - Modus Operandi: [deskripsi]  
   - Bukti Kunci: [event ID + screenshot]  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `log2timeline` gagal | Cek offset partisi dengan `mmls`:  
  ```bash
  docker exec -it timesketch mmls /tmp/nps.dd
  ``` |  
| Timesketch lambat | Kurangi jumlah event dengan filter lebih spesifik. |  
| Tidak ada event | Pastikan dataset mengandung artifact (coba dengan image lain: [CFT Image](https://digitalcorpora.org/corpora/scenarios/corpora-scenarios/)). |  

---

### **Referensi**  
1. [Timesketch Documentation](https://timesketch.org/guides/)  
2. [log2timeline Usage](https://plaso.readthedocs.io/en/latest/sources/user/Timeline-generation.html)  
3. [Digital Forensics Timeline Analysis](https://www.sans.org/white-papers/36872/)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Membuat timeline forensik dari disk image.  
✅ Menganalisis event kronologis untuk rekonstruksi kejadian.  
✅ Menggunakan Timesketch untuk visualisasi dan storytelling.  

**Pesan Penting**:  
> *"Timeline adalah jembatan antara data mentah dan narasi investigasi. Satu event yang salah tempat bisa mengubah seluruh kesimpulan!"*  

Siap untuk Hari ke-8? Kita akan mempelajari **Windows Registry Forensics** dengan RegRipper! 🪟🔍
