

### **Artikel Praktikum: Hari ke-5 - File System Metadata Analysis dengan The Sleuth Kit**  

---

#### **Pengantar**  
Hari kelima kita menyelami **level metadata file system** - lapisan tersembunyi yang menyimpan jejak digital paling krusial. Metadata seperti timestamps, permissions, dan lokasi fisik file adalah "jejak kaki" yang digunakan untuk:  
- **Membuktikan kapan file dibuat/diakses/dihapus**  
- **Mendeteksi pemalsuan bukti** (e.g., memodifikasi timestamp)  
- **Merekonstruksi aktivitas pengguna**  

**Tujuan Hari Ini**:  
- Teori (20%): Memahami konsep inode, $MFT, dan MACB timestamps.  
- Praktik (80%): Ekstrak metadata dari disk image menggunakan The Sleuth Kit (TSK).  

---

### **Bagian 1: Teori (20%) - Anatomi Metadata File System**  
#### **1. Inode (Linux/ext4)**  
- **Definisi**: Struktur data yang menyimpan metadata file (kecuali nama file).  
- **Komponen Inode**:  
  ```plaintext
  [Mode] [UID] [GID] [Size] [Atime] [Mtime] [Ctime] [Block Pointers]
  ```  
- **Lokasi**: Di *inode table* (setiap partisi ext4 punya inode table sendiri).  

#### **2. $MFT (NTFS)**  
- **Definisi**: *Master File Table* - database yang mencatat semua file/folder di NTFS.  
- **Struktur $MFT Entry**:  
  ```plaintext
  [Header] [Standard Information] [File Name] [Data Attribute] ... 
  ```  
- **Timestamp di $MFT**:  
  - **SI (Standard Information)**: MACB timestamps (modifikasi, akses, pembuatan, perubahan metadata).  
  - **FN (File Name)**: Timestamp alternatif (bisa dimanipulasi).  

#### **3. MACB Timestamps**  
| Tipe | Nama Lengkap         | Deskripsi                                  | Contoh Penggunaan              |  
|------|----------------------|--------------------------------------------|--------------------------------|  
| **M** | Modification Time   | Waktu konten file terakhir diubah          | Edit dokumen, tambah data       |  
| **A** | Access Time         | Waktu file terakhir dibuka/dibaca          | View file, cat file            |  
| **C** | Creation Time       | Waktu file dibuat (atau dipindahkan)       | Copy file, download             |  
| **B** | Birth Time (NTFS)   | Waktu asli pembuatan (jarang digunakan)    | File asli dari kamera          |  

---

### **Bagian 2: Praktik (80%) - Analisis Metadata dengan The Sleuth Kit**  
#### **Tools yang Digunakan**  
- **The Sleuth Kit (TSK)**: Kumpulan CLI tools untuk analisis file system.  
- **Dataset**: [NTFS Image](https://github.com/ReFirmLabs/binwalk/raw/master/tests/images/ntfs.dd.gz) (download & ekstrak).  

---

#### **Langkah 1: Install & Verifikasi TSK**  
```bash
# Install TSK (sudah termasuk di Kali Linux)
sudo apt update && sudo apt install sleuthkit -y

# Verifikasi instalasi
fls -V
```  
**Output yang Diharapkan**:  
```plaintext
The Sleuth Kit version 4.11.1
... (detail versi)
```  

---

#### **Langkah 2: Analisis Struktur File System**  
1. **Identifikasi Partisi**:  
   ```bash
   # List partition di image
   mmls ntfs.dd
   ```  
   **Output**:  
   ```plaintext
   DOS Partition Table
   Offset Sector: 0
   Units are in 512-byte sectors

        Slot      Start        End          Length       Description
   000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
   001:  -------   0000000001   0000002047   0000002047   Unallocated
   002:  000:000   0000002048   0002047999   0002045952   NTFS (0x07)
   ```  

2. **Ekstrak Metadata Partisi NTFS**:  
   ```bash
   # Gunakan offset partisi (2048 * 512 = 1048576 bytes)
   fls -r -o 1048576 -m / ntfs.dd > timeline.txt
   ```  
   - **Parameter**:  
     - `-r`: Rekursif (semua file/folder).  
     - `-o 1048576`: Offset partisi.  
     - `-m /`: Mount point virtual.  

---

#### **Langkah 3: Analisis Inode/MFT Entry**  
1. **Cari File Menarik**:  
   ```bash
   # Cari file .doc/.pdf di timeline
   grep -E "\.doc|\.pdf" timeline.txt
   ```  
   **Contoh Output**:  
   ```plaintext
   /Users/Admin/Documents/secret.doc  128-128-0: 128 (r/rrwxr-xr-x): 0
   ```  
   - `128`: Nomor inode/MFT entry.  

2. **Dump Metadata dengan `istat`**:  
   ```bash
   # Tampilkan metadata inode 128
   istat -o 1048576 ntfs.dd 128
   ```  
   **Output Penting**:  
   ```plaintext
   inode: 128
   Allocated
   Group: 0
   Generation Id: 0
   UID / GID: 0 / 0
   mode: -rwxrwxrwx
   size: 15360
   num of links: 1
   Inode Times:
   Accessed:       Wed Dec 31 19:00:00 1969
   File Modified:  Wed Dec 31 19:00:00 1969
   Inode Modified: Wed Dec 31 19:00:00 1969
   ```  

---

#### **Langkah 4: Ekstrak File & Verifikasi Timestamp**  
1. **Ekstrak File dengan `icat`**:  
   ```bash
   # Ekstrak inode 128 ke file
   icat -o 1048576 ntfs.dd 128 > secret.doc
   ```  

2. **Verifikasi Timestamp**:  
   ```bash
   # Cek timestamp file hasil ekstrak
   stat secret.doc
   ```  
   **Bandingkan dengan output `istat`**:  
   - Jika timestamp berbeda, kemungkinan ada manipulasi!  

---

#### **Langkah 5: Analisis MACB Timestamp**  
1. **Generate Timeline MACB**:  
   ```bash
   # Buat timeline dengan format MACB
   tsk_gettimes -o 1048576 ntfs.dd > macb_timeline.csv
   ```  

2. **Filter File Penting**:  
   ```bash
   # Cari file dengan modifikasi terbaru
   grep "secret.doc" macb_timeline.csv
   ```  
   **Output Format**:  
   ```plaintext
   0|/Users/Admin/Documents/secret.doc|128|0|1634567890|1634567890|1634567890|1634567890
   ```  
   - **Kolom**: `Mtime|Atime|Ctime|Crtime` (dalam Unix timestamp).  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Sebuah file `ransomware_note.txt` ditemukan di image NTFS. Analisis metadata untuk:  
> 1. Kapan file dibuat/diakses/dimodifikasi?  
> 2. Apakah timestamp asli atau dimanipulasi?  
> 3. Ekstrak isi file untuk bukti konten."*  

**Instruksi**:  
1. **Download Dataset**: [NTFS Image](https://github.com/ReFirmLabs/binwalk/raw/master/tests/images/ntfs.dd.gz).  
2. **Analisis**:  
   - Gunakan `fls` untuk cari file `ransomware_note.txt`.  
   - Ekstrak metadata dengan `istat`.  
   - Bandingkan timestamp di $MFT (SI vs FN).  
3. **Laporan**:  
   ```markdown
   ## Laporan Metadata Analysis  
   ### Temuan  
   1. **Inode/MFT Entry**: [nomor]  
   2. **Timestamp**:  
      - Mtime: [tanggal]  
      - Atime: [tanggal]  
      - Ctime: [tanggal]  
      - Crtime: [tanggal]  
   3. **Manipulasi?**: [Ya/Tidak + alasan]  
   4. **Isi File**:  
      ```plaintext
      [paste konten ransomware_note.txt]
      ```  
   ### Bukti  
   - Screenshot `istat`  
   - Screenshot `icat`  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `mmls: Invalid image` | Cek integrity file: `sha256sum ntfs.dd` → bandingkan dengan hash asli. |  
| `fls: Cannot determine file system type` | Gunakan offset partisi yang benar (lihat output `mmls`). |  
| Timestamp 1970 (Unix epoch) | Ini timestamp default (file belum pernah diakses). |  
| Tidak menemukan file | Coba dengan `fls -a` (termasuk file terhapus). |  

---

### **Referensi**  
1. [The Sleuth Kit Documentation](https://wiki.sleuthkit.org/index.php?title=The_Sleuth_Kit)  
2. [NTFS $MFT Structure](https://docs.microsoft.com/en-us/windows/win32/fileio/master-file-table)  
3. [MACB Timestamps Explained](https://www.sans.org/blog/mac-timestamps/)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami struktur metadata di NTFS/ext4.  
✅ Mengekstrak dan menganalisis timestamp MACB.  
✅ Mendeteksi potensi manipulasi bukti digital.  

**Pesan Penting**:  
> *"Metadata adalah DNA dari file digital. Satu detik perbedaan timestamp bisa mengubah narasi investigasi."*  

Siap untuk Hari ke-6? Kita akan mempelajari **Data Recovery** dengan Autopsy dan PhotoRec! 💾🔧
