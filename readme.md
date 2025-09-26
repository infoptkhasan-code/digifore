

Berikut adalah **Content Plan 30 Hari Belajar Digital Forensic** dengan fokus **80% praktik** dan **20% teori**, dirancang untuk pembelajaran mandiri. Materi disusun secara bertahap dari dasar hingga lanjutan, dengan tools open-source dan dataset latihan yang dapat diakses gratis.

---

### **Prinsip Pembelajaran**
- **Teori (20%)**: Konsep fundamental, metodologi, dan hukum.  
- **Praktik (80%)**: Hands-on dengan tools nyata, dataset latihan, dan simulasi kasus.  
- **Tools Utama**: Autopsy, The Sleuth Kit (TSK), Volatility, Wireshark, FTK Imager, Bulk Extractor.  
- **Dataset Latihan**: Gunakan dataset dari [DFIR Science](https://dfir.science/), [CyberDefenders](https://cyberdefenders.org/), atau [Google Drive Forensics](https://github.com/google/grr).  

---

### **Struktur Harian**
Setiap hari terdiri dari:  
1. **Teori (1-2 jam)**: Baca konsep + tonton video pendek (max 30 menit).  
2. **Praktik (4-6 jam)**: Eksperimen dengan tools + analisis dataset.  
3. **Tugas Harian**: Dokumentasi hasil dalam laporan singkat (format template disediakan).  

---

### **Content Plan 30 Hari**

#### **Minggu 1: Fondasi Digital Forensic & Disk Imaging**
| Hari | Teori (20%)                          | Praktik (80%)                                                                 | Tools & Resources                                                                 |
|------|--------------------------------------|-------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| 1    | Pengantar Digital Forensic: Siklus hidup forensik, legalitas, etika. | Instalasi lab virtual (VirtualBox + Kali Linux). Buat forensic workstation. | [Kali Linux](https://www.kali.org/), [VirtualBox](https://www.virtualbox.org/)    |
| 2    | Konsep Disk & Partisi: MBR, GPT, file system (FAT, NTFS, ext4). | Analisis struktur disk dengan `fdisk`, `gparted`, dan `hexedit`.               | [TestDisk](https://www.cgsecurity.org/wiki/TestDisk)                             |
| 3    | Disk Imaging: Metode (dead/live), write-blocker, hashing (MD5/SHA). | Buat disk image USB drive menggunakan `dd` dan `FTK Imager`. Verifikasi hash. | [FTK Imager](https://www.exterro.com/ftk-imager)                                  |
| 4    | Chain of Custody: Dokumentasi proses forensik. | Simulasi kasus: Dokumentasikan proses imaging dengan template chain of custody. | [Template NIST](https://www.nist.gov/itl/small-business-security-publications)   |
| 5    | File System Metadata: Inode, $MFT, timestamps (MACB). | Ekstrak metadata dari file dengan `tsk_gettimes` dan `istat` (The Sleuth Kit). | [The Sleuth Kit](https://www.sleuthkit.org/)                                     |
| 6    | Data Recovery: Konsep deleted file, slack space. | Pulihkan file terhapus dari disk image menggunakan `autopsy` dan `photorec`.  | [Autopsy](https://www.autopsy.com/), [PhotoRec](https://www.cgsecurity.org/wiki/PhotoRec) |
| 7    | **Review**: Uji coba latihan CyberDefenders "Disk Image Forensics". | Selesaikan challenge [CyberDefenders: Disk01](https://cyberdefenders.org/blueteam-ctf-challenges/#disk01). | [CyberDefenders](https://cyberdefenders.org/)                                    |

---

#### **Minggu 2: File System Analysis & Windows Forensics**
| Hari | Teori (20%)                          | Praktik (80%)                                                                 | Tools & Resources                                                                 |
|------|--------------------------------------|-------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| 8    | Windows Artifacts: Registry, Prefetch, $LogFile, $UsnJrnl. | Analisis registry hive (SAM, SYSTEM, SOFTWARE) dengan `RegRipper`.           | [RegRipper](https://github.com/keydet89/RegRipper)                                |
| 9    | Timeline Analysis: Membuat timeline aktivitas sistem. | Generate timeline dari disk image menggunakan `log2timeline` (Timesketch).    | [Timesketch](https://github.com/google/timesketch)                                |
| 10   | File Carving: Header/footer, deep carving. | Ekstrak file dari unallocated space dengan `bulk_extractor` dan `foremost`.   | [Bulk Extractor](https://github.com/simsong/bulk_extractor), [Foremost](https://forensics.cert.org/foremost/) |
| 11   | Browser Forensics: History, cache, cookies (Chrome/Firefox). | Analisis database SQLite browser dengan `sqlite3` dan `Browser History Forensics`. | [SQLite Browser](https://sqlitebrowser.org/)                                     |
| 12   | Email Forensics: PST/OST, header analysis. | Ekstrak email dari file PST menggunakan `readpst` dan analisis header.        | [readpst](http://www.five-ten-sg.com/libpst/)                                    |
| 13   | Linux Forensics: Log files, bash history, cron jobs. | Analisis image Linux (ext4) untuk jejak aktivitas user dan malware.            | Dataset: [Linux Memory Dump](https://downloads.volatilityfoundation.org/)         |
| 14   | **Review**: Selesaikan CTF "Windows Artifacts" di CyberDefenders. | Challenge: [CyberDefenders: Win-Registry](https://cyberdefenders.org/blueteam-ctf-challenges/#win-registry). | [CyberDefenders](https://cyberdefenders.org/)                                    |

---

#### **Minggu 3: Memory & Network Forensics**
| Hari | Teori (20%)                          | Praktik (80%)                                                                 | Tools & Resources                                                                 |
|------|--------------------------------------|-------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| 15   | Memory Forensics: Volatile data, RAM acquisition. | Capture RAM dari live system menggunakan `FTK Imager Lite` atau `Belkasoft RAM Capturer`. | [Belkasoft RAM Capturer](https://belkasoft.com/ram-capturer.html)                |
| 16   | Volatility Framework: Plugin, profil OS. | Analisis memory dump untuk proses, network connections, dan malware dengan `volatility`. | [Volatility](https://www.volatilityfoundation.org/)                              |
| 17   | Malware Analysis: Persistence, rootkit. | Deteksi malware di memory dump (e.g., `malfind`, `psxview`).                  | Contoh dataset: [Cridex](https://memory.analysis.blog/2020/05/12/cridex-memory-forensics-with-volatility-3/) |
| 18   | Network Forensics: TCP/IP, packet capture. | Analisis traffic dengan Wireshark: filter HTTP/DNS, ekstrak file.             | [Wireshark](https://www.wireshark.org/)                                          |
| 19   | Intrusion Detection: Log analysis (Apache, Firewall). | Analisis log serangan web dengan `grep`, `awk`, dan `Splunk` (versi free).     | Dataset: [Apache Logs](https://github.com/elastic/examples/blob/master/Common%20Data%20Formats/apache_logs/apache_logs) |
| 20   | VPN/Proxy Forensics: Tracing anonymized traffic. | Analisis VPN logs dan korelasikan dengan network traffic.                      | Dataset: [VPN Traffic](https://www.netresec.com/?page=PCAP)                       |
| 21   | **Review**: Selesaikan CTF "Memory Forensics" di CyberDefenders. | Challenge: [CyberDefenders: Memory-Inspection](https://cyberdefenders.org/blueteam-ctf-challenges/#memory-inspection). | [CyberDefenders](https://cyberdefenders.org/)                                    |

---

#### **Minggu 4: Mobile, Cloud & Reporting**
| Hari | Teori (20%)                          | Praktik (80%)                                                                 | Tools & Resources                                                                 |
|------|--------------------------------------|-------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| 22   | Mobile Forensics: Android vs iOS, acquisition methods. | Ekstrak data Android (SMS, call logs) dari backup menggunakan `adb` dan `Autopsy`. | [ADB](https://developer.android.com/studio/command-line/adb), [Autopsy](https://www.autopsy.com/) |
| 23   | iOS Forensics: Encrypted backup, keychain. | Analisis iPhone backup dengan `iLEAPP` atau `iPhone Backup Analyzer`.        | [iLEAPP](https://github.com/abrignoni/iLEAPP)                                     |
| 24   | Cloud Forensics: AWS, Azure, Google Drive. | Analisis cloud logs (AWS CloudTrail) dan sinkronisasi file.                   | Dataset: [AWS CloudTrail Logs](https://aws.amazon.com/cloudtrail/)                |
| 25   | Anti-Forensics: Steganography, encryption, wiping. | Deteksi steganografi dengan `zsteg` dan coba dekripsi file terenkripsi.       | [zsteg](https://github.com/zed-0xff/zsteg), [John the Ripper](https://www.openwall.com/john/) |
| 26   | Reporting: Menulis laporan forensik yang valid di pengadilan. | Buat laporan lengkap dari kasus sebelumnya (template disediakan).             | Template: [NIST SP 800-86](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-86.pdf) |
| 27   | Court Testimony: Teknik presentasi bukti digital. | Simulasi courtroom: Presentasikan temuan kasus malware.                       | Video: [Digital Forensics in Court](https://www.youtube.com/watch?v=K5eZ5v7Qk9A)  |
| 28   | Emerging Trends: IoT forensics, AI in forensics. | Analisis log perangkat IoT (smart camera) dengan `Wireshark`.                 | Dataset: [IoT Traffic](https://www.stratosphereips.org/datasets-iot)              |
| 29   | **Capstone Project**: Kasus komprehensif (disk + memory + network). | Selesaikan challenge [DFIR Science: Case 001](https://dfir.science/case-001). | [DFIR Science](https://dfir.science/)                                             |
| 30   | **Final Review & Roadmap**: Evaluasi progress + rencana belajar lanjutan. | Buat resume keseluruhan materi + sertifikasi (misal: GCFA, CDFE).              | [SANS DFIR](https://www.sans.org/cyber-security-courses/digital-forensics-and-incident-response/) |

---

### **Tips Sukses**
1. **Dokumentasi**: Gunakan template laporan harian (contoh: [NIST Template](https://www.nist.gov/document/template-digital-forensic-report)).  
2. **Komunitas**: Bergabung dengan forum seperti [Reddit r/computerforensics](https://www.reddit.com/r/computerforensics/) untuk bertanya.  
3. **Sertifikasi**: Setelah 30 hari, siapkan diri untuk sertifikasi dasar (e.g., [CDFE](https://mile2.com/certified-digital-forensics-examiner-cdfe.html)).  
4. **Waktu**: Fokus 4-6 jam/hari. Jika terlewat, gabungkan dengan hari berikutnya.  

### **Resources Tambahan**
- **Buku**: *Digital Forensics with Open Source Tools* (Cory Altheide).  
- **YouTube**: [DFIR Training](https://www.youtube.com/c/DFIRTraining), [OSDFIR](https://www.youtube.com/c/OSDFIRConferences).  
- **CTF Platforms**: [Hack The Box](https://www.hackthebox.com/), [TryHackMe](https://tryhackme.com/).  

Semoga plan ini membantu Anda menjadi ahli digital forensic! Jika butuh bantuan teknis selama praktik, jangan ragu bertanya. 🚀
