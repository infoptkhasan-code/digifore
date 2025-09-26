

### **Artikel Praktikum: Hari ke-2 - Analisis Struktur Disk & Partisi dengan CLI Tools**  

---

#### **Pengantar**  
Hari kedua kita akan *deep dive* ke level paling fundamental dalam Digital Forensic: **struktur fisik disk dan partisi**. Memahami bagaimana data diorganisasi di level sektor, partisi, dan file system adalah kunci untuk:  
- Menemukan partisi tersembunyi  
- Memulihkan data yang rusak  
- Mendeteksi anti-forensik (e.g., rootkit yang memodifikasi MBR)  

**Tujuan Hari Ini**:  
- Teori (20%): Memahami MBR, GPT, dan jenis file system (FAT/NTFS/ext4).  
- Praktik (80%): Analisis disk menggunakan tools CLI (`fdisk`, `gparted`, `hexedit`).  

---

### **Bagian 1: Teori (20%) - Anatomi Disk Digital**  
#### **1. Master Boot Record (MBR)**  
- **Lokasi**: Sektor pertama disk (sektor 0, cylinder 0, head 0).  
- **Ukuran**: 512 bytes.  
- **Struktur**:  
  ```plaintext
  [Boot Code (446 bytes)] [Partition Table (64 bytes)] [Signature (2 bytes: 0x55AA)]
  ```  
- **Batasan**:  
  - Hanya 4 partisi primer.  
  - Maksimum 2TB disk.  

#### **2. GUID Partition Table (GPT)**  
- **Lokasi**: Sektor pertama LBA (Logical Block Addressing).  
- **Keunggulan**:  
  - Hingga 128 partisi.  
  - Dukungan disk > 2TB.  
  - Redundansi (backup di akhir disk).  
- **Struktur**:  
  ```plaintext
  [Protective MBR] [GPT Header] [Partition Entries] ... [Data] [Backup GPT]
  ```  

#### **3. File System**  
| Type   | Karakteristik                          | Use Case               |  
|--------|----------------------------------------|------------------------|  
| FAT32  | Sederhana, kompatibel tinggi           | USB, SD Card           |  
| NTFS   | Support file >4GB, permission, encryption | Windows modern         |  
| ext4   | Journaling, support file besar         | Linux                  |  

---

### **Bagian 2: Praktik (80%) - Analisis Disk dengan CLI Tools**  
#### **Tools yang Digunakan**  
- `fdisk`: Manipulasi partisi (MBR/GPT).  
- `gparted`: GUI untuk partisi (opsional, untuk visualisasi).  
- `hexedit`: Editor heksa untuk inspeksi sektor.  
- `file`: Identifikasi tipe file/partisi.  

---

#### **Langkah 1: Membuat Disk Virtual untuk Praktik**  
Kita akan membuat file disk virtual 100MB untuk aman:  
```bash
# Buat file disk 100MB
dd if=/dev/zero of=disk_test.img bs=1M count=100

# Format sebagai MBR dengan 1 partisi FAT32
echo -e "o\nn\np\n1\n\n\nt\nc\nw" | fdisk disk_test.img

# Format partisi sebagai FAT32
sudo losetup --offset 1048576 /dev/loop0 disk_test.img  # 1048576 = 2048 * 512 (sector size)
sudo mkfs.vfat -F 32 /dev/loop0
sudo losetup -d /dev/loop0
```  

#### **Langkah 2: Analisis MBR dengan `fdisk`**  
```bash
# Tampilkan informasi partisi
fdisk -l disk_test.img
```  
**Output yang Harus Muncul**:  
```plaintext
Disk disk_test.img: 100 MiB, 104857600 bytes, 204800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x12345678

Device        Boot Start    End Sectors  Size Id Type
disk_test.img1       2048  204799   202752   99M  c W95 FAT32 (LBA)
```  

#### **Langkah 3: Inspeksi MBR dengan `hexedit`**  
```bash
# Buka disk di editor heksa
hexedit disk_test.img
```  
**Yang Perlu Diperhatikan**:  
1. **Signature MBR** (offset 0x1FE): `55 AA`.  
2. **Partition Table** (offset 0x1BE):  
   - Byte 1: Status boot (0x80 = aktif).  
   - Byte 5-8: Tipe partisi (0x0C = FAT32 LBA).  
   ![Hexedit MBR](https://i.imgur.com/5XJzZ9l.png)  

#### **Langkah 4: Analisis File System**  
```bash
# Mount partisi FAT32
mkdir /tmp/fat32
sudo mount -o loop,offset=1048576 disk_test.img /tmp/fat32

# Buat file test
echo "Data forensik" > /tmp/fat32/evidence.txt

# Unmount
sudo umount /tmp/fat32
```  

#### **Langkah 5: Ekstrak File dengan `strings` & `grep`**  
```bash
# Cari string ASCII di disk
strings disk_test.img | grep -i "forensik"
```  
**Output**:  
```plaintext
Data forensik
```  

---

### **Tugas Harian (Wajib Dikerjakan!)**  
#### **Skenario Kasus**:  
> *"Seorang tersangka menghapus partisi berisi bukti. Anda menemukan disk image `suspect.img` [download](https://github.com/andrew-morris/hands-on-with-disk-images/blob/master/images/suspect.img?raw=true). Analisis dan temukan:  
> 1. Tipe partisi (MBR/GPT).  
> 2. File system yang digunakan.  
> 3. File tersembunyi di partisi."*  

#### **Langkah Pengerjaan**:  
1. Download `suspect.img` (100MB).  
2. Jalankan perintah:  
   ```bash
   # Identifikasi tipe disk
   file suspect.img

   # Analisis partisi
   fdisk -l suspect.img

   # Cari string menarik
   strings suspect.img | grep -i "password\|secret\|flag"
   ```  
3. **Buat Laporan**:  
   ```markdown
   ## Laporan Hari ke-2: Analisis Disk  
   ### Temuan  
   1. Tipe Partisi: [MBR/GPT]  
   2. File System: [FAT32/NTFS/ext4]  
   3. File Penting: [nama file + konten]  
   ### Bukti  
   - Screenshot `fdisk -l`  
   - Screenshot `strings` yang menunjukkan file tersembunyi  
   ```  

---

### **Troubleshooting Umum**  
| Masalah | Solusi |  
|---------|--------|  
| `fdisk: cannot open disk_test.img` | Cek permission: `chmod 644 disk_test.img` |  
| `mount: wrong fs type` | Gunakan `fdisk -l` untuk cek offset partisi |  
| Tidak menemukan string | Coba dengan `strings -t d suspect.img` untuk tampilkan offset |  

---

### **Referensi**  
1. [MBR Structure Wiki](https://en.wikipedia.org/wiki/Master_boot_record)  
2. [GPT Specification](https://en.wikipedia.org/wiki/GUID_Partition_Table)  
3. [Linux File Systems Explained](https://www.kernel.org/doc/html/latest/filesystems/)  

---

### **Kesimpulan**  
Hari ini Anda telah:  
✅ Memahami struktur fisik disk (MBR/GPT).  
✅ Menggunakan tools CLI untuk analisis partisi.  
✅ Mengekstrak data dari partisi yang terhapus.  

**Pesan Penting**:  
> *"90% data yang 'terhapus' masih ada di disk. Kunci keberhasilan forensik adalah memahami di mana data itu disembunyikan."*  

Siap untuk Hari ke-3? Kita akan mempelajari **Disk Imaging** dengan `dd` dan `FTK Imager`! 💾🔍
