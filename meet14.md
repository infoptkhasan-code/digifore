

### **Artikel Praktikum: Hari ke-13 - iOS Forensics dengan iLEAPP dan iPhone Backup Analyzer**  

---

#### **Pengantar**  
Hari ketiga belas kita menjelajahi **iOS forensics** - tantangan unik karena sistem keamanan berlapis Apple (Secure Enclave, hardware encryption). Skill ini vital untuk:  
- **Mengungkap komunikasi terenkripsi** (iMessage, WhatsApp)  
- **Melacak pergerakan pengguna** (Significant Locations)  
- **Merekonstruksi aktivitas aplikasi** (social media, banking)  

**Tujuan Hari Ini**:  
- Teori (20%): Arsitektur iOS, security model, dan artifact kunci.  
- Praktik (80%): Ekstrak dan analisis iOS backup dengan iLEAPP.  

---

### **Bagian 1: Teori (20%) - Fundamentals iOS Forensics**  
#### **1. Arsitektur iOS**  
| Komponen | Deskripsi | Relevansi Forensik |  
|----------|-----------|-------------------|  
| **Secure Enclave** | Co-prosesor terpisah untuk enkripsi | Menyimpan encryption keys, biometric data |  
| **APFS** | Apple File System (encrypted) | Struktur file dengan snapshot, clone |  
| **Keychain** | Database password dan sertifikat | Menyimpan password aplikasi, Wi-Fi, VPN |  
| **Sandbox** | Isolasi aplikasi | Membatasi akses antar aplikasi |  

#### **2. Metode Akuisisi iOS**  
| Metode | Deskripsi | Keterbatasan |  
|--------|-----------|--------------|  
| **Logical** | Backup via iTunes/Finder | Hanya data user terenkripsi |  
| **File System** | Mounting filesystem (jailbreak required) | Butuh jailbreak (jarang di iOS modern) |  
| **Physical** | Chip-off (ekstrak chip NAND) | Destructive, butuh alat khusus |  

#### **3. Artifact Kunci di iOS**  
| Artifact | Lokasi di Backup | Isi |  
|----------|------------------|-----|  
| **SMS/iMessage** | `HomeDomain/Library/SMS/sms.db` | Pesan teks, attachment |  
| **Call History** | `HomeDomain/Library/CallDB/call_history.db` | Riwayat panggilan |  
| **Contacts** | `HomeDomain/Library/AddressBook/AddressBook.sqlitedb` | Daftar kontak |  
| **WhatsApp** | `AppDomain-com.whatsapp/Library/Application Support/ChatStorage.sqlite` | Chat, media |  
| **Location History** | `HomeDomain/Library/Caches/locationd/consolidated.db` | GPS tracking |  
| **Photos** | `MediaDomain/PhotoData/Photos.sqlite` | Metadata foto/video |  

---

### **Bagian 2: Praktik (80%) - Analisis iOS Backup**  
#### **Tools yang Digunakan**  
- **iLEAPP**: iOS Logs, Events, And Properties Parser (open-source).  
- **iPhone Backup Analyzer**: GUI tool untuk analisis backup.  
- **Dataset**: [iOS 13 Backup](https://github.com/abrignoni/iLEAPP-sample-backups) (download `iOS13-Backup.zip`).  

---

#### **Langkah 1: Install iLEAPP**  
```bash
# Install Python 3.8+ (jika belum)
sudo apt update && sudo apt install python3.8 python3-pip -y

# Install iLEAPP
git clone https://github.com/abrignoni/iLEAPP.git
cd iLEAPP
pip3 install -r requirements.txt
```  

---

#### **Langkah 2: Ekstrak Backup iOS**  
1. **Download & Ekstrak Dataset**:  
   ```bash
   wget https://github.com/abrignoni/iLEAPP-sample-backups/raw/main/iOS13-Backup.zip
   unzip iOS13-Backup.zip
   ```  

2. **Struktur Folder Backup**:  
   ```plaintext
   iOS13-Backup/
   ├── 3d/           # 3D Touch data
   ├── AppDomain-    # Data aplikasi
   ├── HomeDomain/   # Data user
   ├── MediaDomain/  # Media files
   └── Manifest.db   # Database metadata backup
   ```  

---

#### **Langkah 3: Analisis dengan iLEAPP**  
1. **Jalankan iLEAPP**:  
   ```bash
   python3 ileapp.py -t all -o iOS13_Report/ iOS13-Backup/
   ```  
   - **Parameter**:  
     - `-t all`: Analisis semua artifact.  
     - `-o`: Folder output.  

2. **Hasil Otomatis**:  
   - Folder `iOS13_Report/` berisi:  
     ```plaintext
     index.html           # Laporan interaktif
     SMS_Reports/         # Detail SMS
     WhatsApp_Reports/    # Detail WhatsApp
     Location_Reports/    # Detail lokasi
     ```  

---

#### **Langkah 4: Analisis Manual Key Artifact**  
1. **SMS/iMessage**:  
   - Buka `iOS13_Report/SMS_Reports/sms_report.html`  
   - **Filter Pesan Penting**:  
     ```html
     <tr>
       <td>2023-01-15 14:30:00</td>
       <td>+628123456789</td>
       <td>Meet at location X at 3PM</td>
     </tr>
     ```  

2. **WhatsApp**:  
   - Buka `iOS13_Report/WhatsApp_Reports/whatsapp_report.html`  
   - **Query Chat Mencurigakan**:  
     ```sql
     SELECT ZWACHATSESSION.ZCONTACTNAME, ZWAMESSAGE.ZTEXT 
     FROM ZWAMESSAGE 
     JOIN ZWACHATSESSION ON ZWAMESSAGE.ZCHATSESSION = ZWACHATSESSION.Z_PK 
     WHERE ZTEXT LIKE '%bomb%' OR ZTEXT LIKE '%ransom%';
     ```  

3. **Location History**:  
   - Buka `iOS13_Report/Location_Reports/location_report.html`  
   - **Filter Lokasi**:  
     ```html
     <tr>
       <td>2023-01-15 14:00:00</td>
       <td>-6.2088, 106.8456</td>
       <td>Jakarta, Indonesia</td>
     </tr>
     ```  

---

#### **Langkah 5: Analisis Metadata Media**  
1. **Ekstrak Foto**:  
   ```bash
   # Salin foto dari backup
   cp iOS13-Backup/MediaDomain/PhotoData/Metadata/DCIM/100APPLE/*.jpg ~/ios_photos/
   ```  

2. **Analisis Metadata**:  
   ```bash
   # Install exiftool
   sudo apt install libimage-exiftool-perl -y

   # Ekstrak metadata
   exiftool ~/ios_photos/IMG_1234.JPG
   ```  
   **Output Penting**:  
   ```plaintext
   Create Date         : 2023:01:15 13:45:00
   GPS Position        : -6.2088, 106.8456
   Device Model        : iPhone11,8 (iPhone XR)
   Software           : iOS 13.3
   ```  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Sebuah iPhone XR ditemukan di TKP kasus terorisme. Analisis backup untuk:  
> 1. Temukan SMS/iMessage mencurigakan.  
> 2. Ekstrak chat WhatsApp tentang rencana serangan.  
> 3. Identifikasi lokasi terakhir pengguna."*  

**Instruksi**:  
1. **Download Dataset**: [iOS 13 Backup](https://github.com/abrignoni/iLEAPP-sample-backups).  
2. **Analisis**:  
   - Jalankan iLEAPP untuk generate laporan.  
   - Fokus ke SMS, WhatsApp, dan location.  
3. **Laporan**:  
   ```markdown
   ## iOS Forensics Report  
   ### SMS/iMessage  
   - **Nomor Mencurigakan**: [+628123456789](tel:+628123456789)  
   - **Isi Pesan**: "Meet at location X at 3PM"  
   ### WhatsApp  
   - **Kontak**: [Nama Kontak]  
   - **Isi Chat**: [quote pesan penting]  
   ### Lokasi Terakhir  
   - **Koordinat**: -6.2088, 106.8456  
   - **Lokasi**: Jakarta, Indonesia  
   - **Waktu**: 2023-01-15 14:00:00  
   ### Bukti  
   - Screenshot SMS report  
   - Screenshot WhatsApp chat  
   - Screenshot location map  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `ileapp.py: command not found` | Install ulang: `pip3 install -r requirements.txt` |  
| Backup terenkripsi | Cari password di keychain atau gunakan tools seperti `Elcomsoft iOS Forensic Toolkit` (berbayar) |  
| Tidak ada data WhatsApp | Pastikan backup termasuk aplikasi data (biasanya tidak default di iTunes backup) |  
| Lokasi kosong | iOS 13+ encrypt location data, butuh jailbreak untuk akses penuh |  

---

### **Referensi**  
1. [iLEAPP Documentation](https://github.com/abrignoni/iLEAPP)  
2. [iOS Security Guide](https://www.apple.com/business/site/docs/iOS_Security_Guide.pdf)  
3. [iPhone Backup Analyzer](https://github.com/chihiro444/iphone-backup-analyzer)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami arsitektur iOS dan tantangan forensiknya.  
�as Mengekstrak dan menganalisis iOS backup dengan iLEAPP.  
✅ Mendeteksi bukti krusial di SMS, WhatsApp, dan lokasi.  

**Pesan Penting**:  
> *"iOS adalah benteng digital, tapi setiap benteng punya celah. Tugas Anda adalah menemukan jejak digital yang terselip di antara lapisan enkripsi!"*  

Siap untuk Hari ke-14? Kita akan mempelajari **Cloud Forensics (AWS)**! ☁️🔍
