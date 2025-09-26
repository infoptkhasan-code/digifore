

### **Artikel Praktikum: Hari ke-11 - Memory Forensics dengan Volatility**  

---

#### **Pengantar**  
Hari kesebelas kita menyelam ke **memory forensics** - analisis data volatile (RAM) yang mengandung "jejak digital" yang hilang saat sistem dimatikan. Memory forensik adalah kunci untuk:  
- **Menangkap malware tanpa file** (fileless malware)  
- **Mengungkap koneksi aktif** (C2 server, data exfiltration)  
- **Merekonstruksi aktivitas real-time** (proses, password, encryption keys)  

**Tujuan Hari Ini**:  
- Teori (20%): Konsep memory acquisition, struktur RAM, dan Volatility framework.  
- Praktik (80%): Analisis memory dump dengan Volatility untuk deteksi malware.  

---

### **Bagian 1: Teori (20%) - Fundamentals Memory Forensics**  
#### **1. Apa itu Memory Forensik?**  
**Definisi**: Proses menangkap dan menganalisis konten RAM untuk mengekstrak informasi tentang keadaan sistem saat runtime.  

**Mengapa Penting?**  
- **Volatile Evidence**: 90% data penting (password, koneksi, proses) hanya ada di RAM.  
- **Anti-Forensik Detection**: Rootkit dan malware sering menyembunyikan diri di RAM.  
- **Real-time Context**: Menangkap keadaan sistem saat serangan terjadi.  

#### **2. Struktur RAM**  
| Komponen | Deskripsi | Relevansi Forensik |  
|----------|-----------|-------------------|  
| **Kernel Space** | Area OS dan driver | Konfigurasi sistem, driver terload |  
| **User Space** | Area aplikasi | Proses, thread, handle |  
| **Page Tables** | Pemetaan virtual → physical address | Menemukan data tersembunyi |  
| **Slack Space** | Sisa data di akhir halaman | Fragmentasi data |  

#### **3. Volatility Framework**  
- **Konsep**: Extract artifacts dari memory dump menggunakan **profil OS**.  
- **Komponen Kunci**:  
  - **Plugins**: Modul untuk ekstrak artifacts (e.g., `pslist`, `netscan`).  
  - **Profiles**: Database struktur OS (e.g., `Win10x64_19041`).  
- **Alur Kerja**:  
  ```mermaid
  graph LR
    A[Memory Dump] --> B[Identify OS Profile]
    B --> C[Jalankan Plugin]
    C --> D[Analisis Output]
  ```

---

### **Bagian 2: Praktik (80%) - Analisis Memory dengan Volatility**  
#### **Tools yang Digunakan**  
- **Volatility**: Memory forensics framework (install di Kali Linux).  
- **Dataset**: [Memory Dump](https://downloads.volatilityfoundation.org/volatility/samples/nps-2009-gen2/nps-2009-gen2-mem.vmem) (Windows 7, 2.5GB).  

---

#### **Langkah 1: Install & Verifikasi Volatility**  
```bash
# Install Volatility
sudo apt update && sudo apt install volatility -y

# Verifikasi instalasi
vol --version
```  
**Output yang Diharapkan**:  
```plaintext
Volatility Foundation Volatility Framework 2.6
```

---

#### **Langkah 2: Identifikasi OS Profile**  
```bash
# Cek profil OS dari memory dump
vol -f nps-2009-gen2-mem.vmem imageinfo
```  
**Output Penting**:  
```plaintext
Suggested Profile(s) : Win7SP1x64, Win7SP0x64
AS Layer1 : WindowsAMD64PagedMemory
```  
- **Gunakan Profil**: `Win7SP1x64` (Windows 7 SP1 64-bit).  

---

#### **Langkah 3: Analisis Proses yang Berjalan**  
```bash
# List semua proses aktif
vol -f nps-2009-gen2-mem.vmem --profile=Win7SP1x64 pslist
```  
**Fokus ke Proses Mencurigakan**:  
```plaintext
Offset(V)  Name                    PID   PPID   Thds   Handles
---------- ---------------------- ----- ------ ------ --------
0x822f9d30 svchost.exe            780    680    15      200
0x8234a030 chrome.exe             1234   3456   25      500
0x8235b030 backdoor.exe           5678   1234   5       50
```  
- **Perhatikan**: `backdoor.exe` (PID 5678) dengan thread rendah (indikasi malware).  

---

#### **Langkah 4: Deteksi Malware dengan `malfind`**  
```bash
# Cari proses dengan injeksi kode
vol -f nps-2009-gen2-mem.vmem --profile=Win7SP1x64 malfind -p 5678
```  
**Output Kunci**:  
```plaintext
Process: backdoor.exe Pid: 5678
Address: 0x0000000000400000
Protection: PAGE_EXECUTE_READWRITE
Dumped memory to: 5678.dmp
```  
- **PAGE_EXECUTE_READWRITE**: Indikasi injeksi kode malware.  
- **File Dump**: `5678.dmp` (untuk analisis lebih lanjut).  

---

#### **Langkah 5: Analisis Koneksi Jaringan**  
```bash
# List koneksi TCP/UDP aktif
vol -f nps-2009-gen2-mem.vmem --profile=Win7SP1x64 netscan
```  
**Filter Koneksi Mencurigakan**:  
```plaintext
Offset(P)  Proto Local Address          Foreign Address         State
---------- ----- ---------------------- ---------------------- ---------
0x822f9d30 TCP   192.168.1.100:54321    203.0.113.10:4444      ESTABLISHED
```  
- **Koneksi ke 203.0.113.10:4444**: C2 server malware.  

---

#### **Langkah 6: Ekstrak Password dan Hash**  
```bash
# Dump password hash
vol -f nps-2009-gen2-mem.vmem --profile=Win7SP1x64 hashdump
```  
**Output**:  
```plaintext
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
```  
- **NTLM Hash**: `8846f7eaee8fb117ad06bdd830b7586c` (bisa di-crack dengan John the Ripper).  

---

#### **Langkah 7: Deteksi Rootkit dengan `psxview`**  
```bash
# Bandingkan berbagai metode enumerasi proses
vol -f nps-2009-gen2-mem.vmem --profile=Win7SP1x64 psxview
```  
**Cari Proses Tersembunyi**:  
```plaintext
PID  pslist  psscan  thrdproc  psdspcid  csrss  session  exit_time  exit_reason
5678 True    True    False    True      True   True     False      -
```  
- **`thrdproc: False`**: Indikasi rootkit menyembunyikan proses.  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Sebuah server Windows 7 terinfeksi malware. Analisis memory dump untuk:  
> 1. Temukan proses mencurigakan.  
> 2. Identifikasi C2 server.  
> 3. Ekstrak password administrator."*  

**Instruksi**:  
1. **Download Dataset**: [Memory Dump](https://downloads.volatilityfoundation.org/volatility/samples/nps-2009-gen2/nps-2009-gen2-mem.vmem).  
2. **Analisis**:  
   - Gunakan `imageinfo` untuk identifikasi profil.  
   - Jalankan `pslist`, `malfind`, `netscan`, dan `hashdump`.  
3. **Laporan**:  
   ```markdown
   ## Memory Forensics Report  
   ### Informasi Sistem  
   - **OS**: Windows 7 SP1 x64  
   - **Profile**: Win7SP1x64  
   ### Proses Mencurigakan  
   - **Nama**: backdoor.exe  
   - **PID**: 5678  
   - **Path**: C:\Temp\backdoor.exe  
   - **Indikasi**: Injeksi kode (PAGE_EXECUTE_READWRITE)  
   ### C2 Server  
   - **IP**: 203.0.113.10  
   - **Port**: 4444  
   - **Proses**: backdoor.exe  
   ### Password Administrator  
   - **Username**: Administrator  
   - **NTLM Hash**: 8846f7eaee8fb117ad06bdd830b7586c  
   ### Kesimpulan  
   - **Status Terinfeksi**: Ya  
   - **Bukti Kunci**: Proses backdoor.exe + koneksi ke 203.0.113.10:4444  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `imageinfo` gagal | Cek integrity file: `sha256sum nps-2009-gen2-mem.vmem` |  
| Profile tidak cocok | Coba semua profil yang disarankan (e.g., `Win7SP0x64`). |  
| `malfind` tidak menemukan apa-apa | Jalankan `psxview` untuk deteksi rootkit. |  
| `netscan` kosong | Gunakan `connscan` untuk koneksi yang sudah tertutup. |  

---

### **Referensi**  
1. [Volatility Documentation](https://github.com/volatilityfoundation/volatility/wiki)  
2. [Memory Forensics Cheat Sheet](https://www.sans.org/posters/memory-forensics-cheat-sheet/)  
3. [Windows Memory Analysis](https://www.forensicswiki.xyz/wiki/index.php?title=Windows_Memory_Analysis)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami konsep memory forensics dan struktur RAM.  
✅ Menggunakan Volatility untuk analisis memory dump.  
✅ Mendeteksi malware, C2 server, dan ekstrak password.  

**Pesan Penting**:  
> *"RAM adalah saksi bisu dari setiap aktivitas di sistem. Bahkan malware yang paling licin pun meninggalkan jejak di memori!"*  

Siap untuk Hari ke-12? Kita akan mempelajari **Mobile Forensics (Android)**! 📱🔍
