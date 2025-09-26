

### **Artikel Praktikum: Hari ke-10 - Network Forensics dengan Wireshark**  

---

#### **Pengantar**  
Hari kesepuluh kita memasuki ranah **network forensics** - analisis lalu lintas jaringan untuk mengungkap aktivitas mencurigakan, serangan siber, dan kebocoran data. Wireshark sebagai "swiss army knife" network forensik memungkinkan kita:  
- **Mendeteksi malware communication** (C2 server)  
- **Melacak data exfiltration**  
- **Menganalisis serangan DDoS, port scanning, dan man-in-the-middle**  

**Tujuan Hari Ini**:  
- Teori (20%): Konsep network forensics, TCP/IP model, dan serangan jaringan.  
- Praktik (80%): Analisis packet capture (pcap) dengan Wireshark.  

---

### **Bagian 1: Teori (20%) - Fundamentals Network Forensics**  
#### **1. Apa itu Network Forensics?**  
**Definisi**: Proses menangkap, merekam, dan menganalisis lalu lintas jaringan untuk mengumpulkan bukti digital.  

**Mengapa Penting?**  
- **Real-time Detection**: Identifikasi serangan saat terjadi.  
- **Post-incident Analysis**: Rekonstruksi serangan setelah kejadian.  
- **Evidence Collection**: Paket jaringan sebagai bukti hukum (RFC 3227).  

#### **2. TCP/IP Model & Packet Structure**  
| Layer | Protokol | Fungsi Forensik |  
|-------|----------|-----------------|  
| **Application** | HTTP, FTP, DNS | Konten data, file transfer |  
| **Transport** | TCP, UDP | Port, connection state |  
| **Internet** | IP, ICMP | Source/destination IP, routing |  
| **Link** | Ethernet, ARP | MAC address, local network |  

**Struktur Packet**:  
```plaintext
[Ethernet Header][IP Header][TCP Header][Payload]
```

#### **3. Jenis Serangan Jaringan**  
| Serangan | Ciri-ciri di Wireshark |  
|----------|------------------------|  
| **Port Scanning** | Banyak SYN packet tanpa ACK |  
| **DDoS** | Flood packet ke target IP |  
| **Malware C2** | Koneksi rutin ke IP/Port mencurigakan |  
| **Data Exfiltration** | Large outbound transfer ke unknown IP |  

---

### **Bagian 2: Praktik (80%) - Analisis Packet dengan Wireshark**  
#### **Tools yang Digunakan**  
- **Wireshark**: Network protocol analyzer.  
- **Dataset**: [Malware PCAP](https://www.malware-traffic-analysis.net/2023/01/2023-01-11-traffic-analysis-exercise.html) (download `2023-01-11-traffic-analysis-exercise.pcap`).  

---

#### **Langkah 1: Install & Konfigurasi Wireshark**  
```bash
# Install Wireshark
sudo apt update && sudo apt install wireshark -y

# Tambahkan user ke grup wireshark (untuk capture tanpa root)
sudo usermod -aG wireshark $USER
newgrp wireshark  # Reload group
```  

---

#### **Langkah 2: Analisis Traffic Normal**  
1. **Buka PCAP di Wireshark**:  
   ```bash
   wireshark 2023-01-11-traffic-analysis-exercise.pcap
   ```  

2. **Filter HTTP Traffic**:  
   - Di display filter: `http`  
   - **Analisis**:  
     - Klik kanan packet > **Follow > TCP Stream**  
     - Amati request/response antara client dan server.  

3. **Identifikasi File Download**:  
   - Filter: `http.content_type == "application/octet-stream"`  
   - Ekstrak file:  
     - Klik kanan packet > **Export Packet Bytes** → Simpan sebagai `malware_sample.bin`  

---

#### **Langkah 3: Deteksi Malware Communication**  
1. **Cari DNS Query Mencurigakan**:  
   - Filter: `dns`  
   - Perhatikan domain dengan:  
     - Nama acak (e.g., `xj3k2l9.com`)  
     - Panjang tidak biasa  

2. **Analisis Koneksi ke C2 Server**:  
   - Filter: `tcp.dstport == 4444` (port umum C2)  
   - **Temuan**:  
     ```plaintext
     No.     Time           Source                Destination           Protocol Length Info
     123     10:15:32.000   192.168.1.100        203.0.113.10         TCP      60     54321 → 4444 [SYN]
     ```  

3. **Ekstrak Payload C2**:  
   - Klik kanan packet C2 > **Follow > TCP Stream**  
   - Cari command atau encoded data di payload.  

---

#### **Langkah 4: Deteksi Data Exfiltration**  
1. **Filter Large Outbound Transfer**:  
   - Filter: `tcp.len > 1000 && ip.src == 192.168.1.100`  
   - **Analisis**:  
     - Cari transfer ke IP eksternal (bukan local network).  
     - Perhatikan port non-standard (bukan 80/443).  

2. **Statistik Transfer**:  
   - Menu **Statistics > Endpoints > IPv4**  
   - Sort by **Tx Bytes** untuk cari IP dengan transfer besar.  

---

#### **Langkah 5: Rekonstruksi Serangan**  
1. **Buat Timeline**:  
   - Export packet list: **File > Export Packet Dissections > As Plain Text**  
   - Filter penting:  
     ```plaintext
     No.     Time           Source                Destination           Protocol Info
     45      10:15:30.000   192.168.1.100        8.8.8.8              DNS     Standard query A xj3k2l9.com
     123     10:15:32.000   192.168.1.100        203.0.113.10         TCP      54321 → 4444 [SYN]
     456     10:15:45.000   192.168.1.100        203.0.113.10         TCP      [HTTP POST] /upload
     ```  

2. **Kesimpulan Serangan**:  
   - **10:15:30**: Malware resolve C2 domain via DNS.  
   - **10:15:32**: Koneksi TCP ke C2 server (port 4444).  
   - **10:15:45**: Data exfiltration via HTTP POST.  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Sebuah workstation terinfeksi malware yang mencuri data sensitif. Analisis PCAP untuk:  
> 1. Identifikasi C2 server (IP/domain).  
> 2. Temukan bukti data exfiltration.  
> 3. Tentukan jenis data yang dicuri."*  

**Instruksi**:  
1. **Download Dataset**: [Malware PCAP](https://www.malware-traffic-analysis.net/2023/01/2023-01-11-traffic-analysis-exercise.html).  
2. **Analisis**:  
   - Gunakan Wireshark untuk buka PCAP.  
   - Terapkan filter untuk deteksi C2 dan exfiltration.  
3. **Laporan**:  
   ```markdown
   ## Network Forensics Report  
   ### C2 Server  
   - **IP Address**: [IP]  
   - **Domain**: [domain]  
   - **Port**: [port]  
   ### Data Exfiltration  
   - **Time**: [timestamp]  
   - **Destination**: [IP/port]  
   - **Data Size**: [bytes]  
   - **File Type**: [PDF/DOC/exe]  
   ### Bukti  
   - **DNS Query**: [screenshot]  
   - **TCP Stream**: [screenshot payload]  
   ### Kesimpulan  
   - **Modus Operandi**: [deskripsi]  
   - **Rekomendasi**: [block IP, isolasi host]  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| Wireshark tidak bisa capture | Jalankan sebagai root: `sudo wireshark` atau tambahkan user ke grup wireshark. |  
| Tidak bisa buka PCAP besar | Gunakan `tshark` (CLI): `tshark -r file.pcap -Y "http" -w filtered.pcap` |  
| Traffic terenkripsi (HTTPS) | Gunakan `ssl` filter dan import private key (jika ada). |  
| Terlalu banyak noise | Gunakan display filter: `tcp.port == 4444 or dns` |  

---

### **Referensi**  
1. [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)  
2. [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/)  
3. [SANS Network Forensics Cheat Sheet](https://www.sans.org/posters/network-forensics/)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami konsep network forensics dan serangan jaringan.  
�as Menganalisis packet capture untuk deteksi malware dan data exfiltration.  
✅ Merekonstruksi serangan dari lalu lintas jaringan.  

**Pesan Penting**:  
> *"Setiap packet menceritakan kisah. Tugas Anda adalah mendengarkan dengan teliti dan menghubungkan titik-titik yang tersembunyi."*  

Siap untuk Hari ke-11? Kita akan mempelajari **Memory Forensics dengan Volatility**! 💾🔍
