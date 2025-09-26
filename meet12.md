

### **Artikel Praktikum: Hari ke-12 - Mobile Forensics (Android) dengan ADB dan Autopsy**  

---

#### **Pengantar**  
Hari kedua belas kita memasuki ranah **mobile forensics** - analisis perangkat Android yang kini menyimpan 80% data pribadi pengguna. Dengan 3.5 miliar pengguna Android globally, skill ini krusial untuk:  
- **Mengungkap komunikasi tersembunyi** (WhatsApp, Telegram)  
- **Melacak lokasi pengguna** (GPS history)  
- **Merekonstruksi aktivitas aplikasi** (social media, banking)  

**Tujuan Hari Ini**:  
- Teori (20%): Arsitektur Android, partisi, dan artifact kunci.  
- Praktik (80%): Ekstrak data Android dengan ADB dan analisis di Autopsy.  

---

### **Bagian 1: Teori (20%) - Fundamentals Android Forensics**  
#### **1. Arsitektur Android**  
| Layer | Komponen | Relevansi Forensik |  
|-------|----------|-------------------|  
| **Kernel** | Linux kernel, drivers | Log sistem, hardware info |  
| **HAL** | Hardware Abstraction Layer | Interface hardware-software |  
| **Libraries** | SQLite, SSL, Media | Database, encryption |  
| **Framework** | Activity Manager, Content Providers | Manajemen aplikasi, data sharing |  
| **Apps** | System & User Apps | Data spesifik aplikasi |  

#### **2. Partisi Android**  
| Partisi | Mount Point | Isi |  
|---------|-------------|-----|  
| **/system** | `/system` | OS, system apps (read-only) |  
| **/data** | `/data` | User data, apps (terenkripsi) |  
| **/cache** | `/cache` | Temporary files |  
| **/recovery** | `/recovery` | Recovery environment |  

#### **3. Artifact Kunci di Android**  
| Artifact | Lokasi | Isi |  
|----------|--------|-----|  
| **Call Logs** | `/data/data/com.android.providers.contacts/databases/calllog.db` | Riwayat panggilan |  
| **SMS** | `/data/data/com.android.providers.telephony/databases/mmssms.db` | Pesan teks |  
| **WhatsApp** | `/data/data/com.whatsapp/databases/msgstore.db` | Chat, media |  
| **Browser History** | `/data/data/com.android.chrome/app_chrome/Default/History` | Riwayat browsing |  
| **Location History** | `/data/data/com.google.android.gms/files/databases/gmm_location.db` | GPS tracking |  

---

### **Bagian 2: Praktik (80%) - Ekstrak Data Android**  
#### **Tools yang Digunakan**  
- **ADB (Android Debug Bridge)**: CLI untuk komunikasi dengan Android.  
- **Autopsy**: Analisis artifact Android.  
- **Dataset**: [Android Emulator Image](https://drive.google.com/file/d/1abc123def456/view) (download `android_emulator.img`).  

---

#### **Langkah 1: Setup ADB di Kali Linux**  
```bash
# Install ADB
sudo apt update && sudo apt install android-tools-adb -y

# Verifikasi instalasi
adb version
```  
**Output yang Diharapkan**:  
```plaintext
Android Debug Bridge version 1.0.41
```

---

#### **Langkah 2: Ekstrak Data dari Android Emulator**  
1. **Start Emulator** (jika menggunakan physical device, aktifkan USB Debugging):  
   ```bash
   # Jalankan emulator (gunakan image yang didownload)
   emulator -avd android_emulator -writable-system
   ```  

2. **Verifikasi Koneksi ADB**:  
   ```bash
   adb devices
   ```  
   **Output**:  
   ```plaintext
   List of devices attached
   emulator-5554   device
   ```  

3. **Ekstrak Artifact Penting**:  
   ```bash
   # Buat folder untuk hasil ekstraksi
   mkdir ~/android_forensics
   cd ~/android_forensics

   # Ekstrak call logs
   adb pull /data/data/com.android.providers.contacts/databases/calllog.db

   # Ekstrak SMS
   adb pull /data/data/com.android.providers.telephony/databases/mmssms.db

   # Ekstrak WhatsApp (jika terinstall)
   adb pull /data/data/com.whatsapp/databases/msgstore.db

   # Ekstrak browser history
   adb pull /data/data/com.android.chrome/app_chrome/Default/History
   ```  

---

#### **Langkah 3: Analisis Database SQLite**  
1. **Install SQLite Browser**:  
   ```bash
   sudo apt install sqlitebrowser -y
   ```  

2. **Analisis Call Logs**:  
   ```bash
   sqlitebrowser calllog.db
   ```  
   **Query Penting**:  
   ```sql
   SELECT number, date, duration, type FROM calls;
   ```  
   - **type**: 1 (incoming), 2 (outgoing), 3 (missed).  
   - **date**: Timestamp dalam milidetik (konversi ke human-readable).  

3. **Analisis SMS**:  
   ```bash
   sqlitebrowser mmssms.db
   ```  
   **Query**:  
   ```sql
   SELECT address, body, date FROM sms;
   ```  

---

#### **Langkah 4: Analisis di Autopsy**  
1. **Buat Kasus Baru**:  
   ```bash
   autopsy  # Buka Autopsy di browser
   ```  
   - **Case Name**: `Day12_Android_Forensics`  
   - **Data Source**: Tambah folder `~/android_forensics`  

2. **Konfigurasi Ingest Modules**:  
   - Centang:  
     - ✅ **Android Artifacts**  
     - ✅ **SQLite Database Parser**  
     - ✅ **Recent Activity**  

3. **Analisis Hasil**:  
   - Navigasi ke **File Analysis** → Filter:  
     - `File Type: SQLite Database`  
     - `Path: */databases/*`  
   - Contoh temuan:  
     ```plaintext
     /databases/msgstore.db (WhatsApp)
     /databases/calllog.db (Call Logs)
     ```  

---

#### **Langkah 5: Ekstrak Media & Metadata**  
1. **Ekstrak Media WhatsApp**:  
   ```bash
   adb pull /sdcard/WhatsApp/Media
   ```  

2. **Analisis Metadata Foto**:  
   ```bash
   # Install exiftool
   sudo apt install libimage-exiftool-perl -y

   # Ekstrak metadata foto
   exiftool ~/android_forensics/Media/WhatsApp/Images/IMG_20230101.jpg
   ```  
   **Output Penting**:  
   ```plaintext
   Create Date         : 2023:01:01 10:30:45
   GPS Position        : 6.2088, 106.8456 (Jakarta)
   Device Model        : SM-G998B (Samsung Galaxy S21)
   ```  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Sebuah Android device ditemukan di TKP kasus penculikan. Analisis untuk:  
> 1. Temukan nomor telepon yang sering dihubungi.  
> 2. Ekstrak chat WhatsApp mencurigakan.  
> 3. Identifikasi lokasi terakhir pengguna."*  

**Instruksi**:  
1. **Download Dataset**: [Android Emulator Image](https://drive.google.com/file/d/1abc123def456/view).  
2. **Ekstrak Data**:  
   - Gunakan ADB untuk pull call logs, SMS, dan WhatsApp.  
   - Ekstrak media WhatsApp.  
3. **Analisis**:  
   - Gunakan SQLite Browser untuk query database.  
   - Gunakan exiftool untuk analisis metadata foto.  
4. **Laporan**:  
   ```markdown
   ## Android Forensics Report  
   ### Call Logs  
   - **Nomor Sering Dihubungi**: [nomor]  
   - **Total Panggilan**: [jumlah]  
   ### WhatsApp Chat  
   - **Kontak Mencurigakan**: [nomor/nama]  
   - **Isi Chat**: [quote pesan penting]  
   ### Lokasi Terakhir  
   - **Koordinat**: [lat, long]  
   - **Lokasi**: [alamat]  
   - **Waktu**: [timestamp]  
   ### Bukti  
   - Screenshot query SQLite  
   - Screenshot metadata foto  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `adb: command not found` | Install ulang: `sudo apt install android-tools-adb` |  
| `device unauthorized` | Di device, centang "Always allow from this computer" saat prompt debugging |  
| `Permission denied` | Jalankan ADB sebagai root: `adb root` (untuk emulator) |  
| Database terenkripsi | Gunakan tool khusus seperti Andriller (free version) untuk dekripsi |  

---

### **Referensi**  
1. [ADB Documentation](https://developer.android.com/studio/command-line/adb)  
2. [Android Forensics with Autopsy](https://www.autopsy.com/docs/android-forensics)  
3. [SQLite Database Analysis](https://www.sqlite.org/cli.html)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami arsitektur Android dan artifact forensik.  
�as Mengekstrak data Android dengan ADB.  
✅ Menganalisis database SQLite dan metadata media.  

**Pesan Penting**:  
> *"Setiap tap, swipe, dan chat di Android meninggalkan jejak digital. Tugas Anda adalah menjadi detektif digital yang mengungkap cerita di balik layar!"*  

Siap untuk Hari ke-13? Kita akan mempelajari **iOS Forensics**! 🍎🔍
