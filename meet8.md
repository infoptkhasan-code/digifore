

### **Artikel Praktikum: Hari ke-8 - Windows Registry Forensics dengan RegRipper**  

---

#### **Pengantar**  
Hari kedelapan kita menyelam ke **Windows Registry** - "otak" sistem Windows yang menyimpan konfigurasi, pengaturan, dan jejak aktivitas pengguna. Registry adalah sumber emas untuk:  
- **Mendeteksi malware persistence** (run keys, services)  
- **Melacak penggunaan perangkat** (USB history)  
- **Merekonstruksi aktivitas user** (browser history, dokumen terakhir dibuka)  

**Tujuan Hari Ini**:  
- Teori (20%): Struktur Registry, hive penting, dan artifact kunci.  
- Praktik (80%): Ekstrak dan analisis Registry dengan RegRipper.  

---

### **Bagian 1: Teori (20%) - Anatomi Windows Registry**  
#### **1. Struktur Registry**  
Registry adalah database hierarkis dengan:  
- **Keys**: Folder (e.g., `HKEY_LOCAL_MACHINE\SOFTWARE`).  
- **Values**: Data dalam key (e.g., `InstallPath = "C:\Program Files"`).  
- **Data Types**: REG_SZ (string), REG_DWORD (angka), REG_BINARY (biner).  

#### **2. Hive Registry Penting**  
| Hive | Lokasi di Disk | Fungsi |  
|------|----------------|--------|  
| **SYSTEM** | `\Windows\System32\config\SYSTEM` | Konfigurasi hardware, driver, service |  
| **SOFTWARE** | `\Windows\System32\config\SOFTWARE` | Aplikasi terinstall, pengaturan sistem |  
| **SAM** | `\Windows\System32\config\SAM` | User accounts, password hashes |  
| **NTUSER.DAT** | `\Users\<username>\NTUSER.DAT` | Pengaturan user-specific (desktop, browser) |  
| **USRCLASS.DAT** | `\Users\<username>\AppData\Local\Microsoft\Windows\UsrClass.dat` | Asosiasi file, shortcut |  

#### **3. Artifact Registry Kunci**  
| Artifact | Key Registry | Kegunaan Forensik |  
|----------|--------------|-------------------|  
| **Run Keys** | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | Malware persistence |  
| **USB History** | `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` | Device connection log |  
| **UserAssist** | `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` | Program execution count |  
| **LastVisitedPidlMRU** | `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU` | File/folder terakhir diakses |  
| **Network Adapter** | `HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces` | IP configuration history |  

---

### **Bagian 2: Praktik (80%) - Analisis Registry dengan RegRipper**  
#### **Tools yang Digunakan**  
- **RegRipper**: CLI tool untuk ekstrak Registry (bagian dari [Kroll Artifact Parser](https://github.com/keydet89/RegRipper2.8)).  
- **Dataset**: [Windows 7 Image](https://digitalcorpora.org/corpora/drives/nps-2009-ubnist1/) (download `nps-2009-ubnist1-gen2.E01`).  

---

#### **Langkah 1: Install RegRipper**  
```bash
# Download RegRipper
git clone https://github.com/keydet89/RegRipper2.8.git
cd RegRipper2.8

# Install dependencies (Perl)
sudo apt install -y libparse-win32registry-perl libdatetime-perl

# Jadikan executable
chmod +x rip.pl
```  

---

#### **Langkah 2: Ekstrak Hive dari Disk Image**  
1. **Mount Image**:  
   ```bash
   # Konversi E01 ke DD (jika perlu)
   ewfexport -f dd -o nps.dd nps-2009-ubnist1-gen2.E01

   # Mount image
   sudo mkdir /mnt/windows_image
   sudo mount -o loop,ro,show_sys_files nps.dd /mnt/windows_image
   ```  

2. **Salin Hive ke Lokasi Aman**:  
   ```bash
   cp /mnt/windows_image/Windows/System32/config/SYSTEM ~/registry_hives/
   cp /mnt/windows_image/Windows/System32/config/SOFTWARE ~/registry_hives/
   cp /mnt/windows_image/Users/Administrator/NTUSER.DAT ~/registry_hives/
   ```  

---

#### **Langkah 3: Analisis Hive dengan RegRipper**  
1. **Analisis Hive SYSTEM**:  
   ```bash
   ./rip.pl -r ~/registry_hives/SYSTEM -p system > system_report.txt
   ```  
   **Output Penting**:  
   ```plaintext
   LastShutdownTime => 2009-01-15 14:30:00
   CurrentControlSet => ControlSet001
   ComputerName => WIN-ABC123
   ```  

2. **Analisis Hive SOFTWARE**:  
   ```bash
   ./rip.pl -r ~/registry_hives/SOFTWARE -p software > software_report.txt
   ```  
   **Fokus ke**:  
   - Aplikasi terinstall:  
     ```plaintext
     Installed Software:
     - Microsoft Office 2007 (C:\Program Files\Microsoft Office)
     - WinRAR (C:\Program Files\WinRAR)
     ```  
   - Run keys:  
     ```plaintext
     HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run:
     - "AdobeAAMUpdater" => "C:\Program Files\Common Files\Adobe\OOBE\PDApp\UWA\UpdaterStartupUtility.exe"
     ```  

3. **Analisis Hive NTUSER.DAT**:  
   ```bash
   ./rip.pl -r ~/registry_hives/NTUSER.DAT -p ntuser > ntuser_report.txt
   ```  
   **Temuan Kunci**:  
   - USB History:  
     ```plaintext
     USBSTOR Entries:
     - VID_0781&PID_5530 => SanDisk Cruzer (Serial: 4C530001090614118976)
     ```  
   - UserAssist:  
     ```plaintext
     UserAssist Counts:
     - cmd.exe: 15 executions
     - winword.exe: 42 executions
     ```  

---

#### **Langkah 4: Deteksi Persistence Mechanism**  
1. **Cari Run Keys Mencurigakan**:  
   ```bash
   grep -i "run" ntuser_report.txt
   ```  
   **Contoh Output**:  
   ```plaintext
   HKCU\Software\Microsoft\Windows\CurrentVersion\Run:
   - "backdoor" => "C:\Temp\svchost.exe"
   ```  

2. **Analisis Service Persistence**:  
   ```bash
   ./rip.pl -r ~/registry_hives/SYSTEM -p svc > services_report.txt
   ```  
   **Cari Service Mencurigakan**:  
   ```plaintext
   Service: "UpdateService"
   ImagePath: "C:\Windows\System32\update.dll"
   StartType: Auto Start
   ```  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Sebuah sistem Windows terinfeksi malware. Analisis Registry untuk:  
> 1. Temukan persistence mechanism (run keys, services).  
> 2. Identifikasi USB yang terhubung ke sistem.  
> 3. Rekonstruksi aktivitas user (program yang sering dijalankan)."*  

**Instruksi**:  
1. **Download Dataset**: [Windows 7 Image](https://digitalcorpora.org/corpora/drives/nps-2009-ubnist1/).  
2. **Analisis**:  
   - Ekstrak hive SYSTEM, SOFTWARE, dan NTUSER.DAT.  
   - Gunakan RegRipper untuk generate report.  
3. **Laporan**:  
   ```markdown
   ## Laporan Registry Forensics  
   ### Persistence Mechanism  
   - **Run Keys**:  
     - [Key]: [Value]  
     - [Key]: [Value]  
   - **Services**:  
     - [Service Name]: [ImagePath]  
   ### USB History  
   - **Device 1**: [Vendor/Model + Serial]  
   - **Device 2**: [Vendor/Model + Serial]  
   ### User Activity  
   - **Program Paling Sering**:  
     - [Program]: [Execution Count]  
   ### Kesimpulan  
   - Indikasi Kompromi: [Ya/Tidak + alasan]  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `Can't locate Parse::Win32Registry.pm` | Install modul Perl:  
  ```bash
  sudo cpan Parse::Win32Registry
  ``` |  
| Registry tidak terbaca | Cek integrity hive:  
  ```bash
  file ~/registry_hives/SYSTEM
  ``` |  
| Tidak ada output | Pastikan menggunakan plugin yang tepat (e.g., `-p system` untuk hive SYSTEM). |  

---

### **Referensi**  
1. [RegRipper Documentation](https://github.com/keydet89/RegRipper2.8/blob/master/README.md)  
2. [Windows Registry Forensics](https://www.sans.org/white-papers/36808/)  
3. [Registry Key Reference](https://forensicswiki.xyz/wiki/index.php?title=Windows_Registry_Key_Reference)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami struktur Windows Registry dan hive penting.  
�as Mengekstrak dan menganalisis Registry dengan RegRipper.  
✅ Mendeteksi persistence mechanism dan aktivitas user.  

**Pesan Penting**:  
> *"Registry adalah buku harian tersembunyi Windows. Setiap key dan value adalah petunjuk yang bisa mengungkap kebenaran!"*  

Siap untuk Hari ke-9? Kita akan mempelajari **Timeline Analysis Lanjutan** dengan Super Timeline! ⏱️🔍
