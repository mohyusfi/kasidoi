# Panduan Testing Manual & Business Flow Sistem Kasidoi (Fatek Untad)

Dokumen ini disusun sebagai **buku panduan praktis (runbook)** untuk melakukan pengujian manual fitur-fitur sistem KASIDOI, sekaligus membedah alur bisnis (*business flow*) dari sudut pandang kampus nyata.

---

## 🎭 1. Daftar Akun Pengujian (Siap Pakai)

Semua password akun default adalah: `12345678`

| Peran (*Role*) | Username | Nama Akun | Hak Akses Utama |
| :--- | :--- | :--- | :--- |
| **Role 2: Keuangan** | `adminfatek` | ADMIN KEUANGAN FATEK | Menentukan Pagu Induk, membuka akses input, mencairkan Panjar, kelola Master Data. |
| **Role 1: Unit Kerja** | `akademikfatek` | Akademik FATEK | Menginput rencana anggaran (RKAKL), mengajukan penarikan dana (RPD), upload bukti nota (LPJ). |
| **Role 3: Pimpinan** | `dekanfatek` | DEKAN FATEK | Memantau penyerapan anggaran, diagram pie chart, dan laporan eksekutif. |

---

## 💡 2. Peta Alur Bisnis (Analogi Kampus Nyata)

Bayangkan proses keuangan fakultas seperti **perjalanan uang kegiatan mahasiswa**:

```
[Dekan & Keuangan]              [Jurusan / Unit Kerja]                [Toko / Vendor]
    (Tahap 1)                         (Tahap 2 & 3)                      (Tahap 4 & 5)
Siapkan Dompet Induk   ===>   Rencanakan Belanja & Jadwal   ===>   Cairkan Panjar & Beli Barang
 (Pagu BOPTN & BLU)              (Pohon RKAKL & RPD)                   (Panjar + Kuitansi LPJ)
```

1. **Keuangan** membuat dompet anggaran tahun berjalan dan menentukan plafon maksimal.
2. **Unit Kerja** merinci kegiatan apa saja yang akan dibiayai (RKAKL) dan kapan uangnya mau diambil (RPD).
3. **Keuangan** mencairkan uang muka belanja (Panjar).
4. **Unit Kerja** membelanjakan uangnya lalu menyerahkan kuitansi/nota sah (LPJ).
5. **Sistem** otomatis memotong sisa saldo dan menampilkan grafik serapan ke **Pimpinan**.

---

## 🚀 3. Skenario Pengujian Manual End-to-End

Ikuti langkah-langkah berurutan di bawah ini untuk menguji siklus lengkap dari awal hingga akhir.

---

### SKENARIO 1: Setup Pagu Induk & Buka Akses Anggaran
> **Aktor**: `adminfatek` (Keuangan)  
> **Tujuan**: Membuka tahun anggaran baru dan mengizinkan jurusan menginput usulan.

#### Langkah Pengujian:
1. Buka browser di `http://localhost:3000/login`.
2. Masuk dengan kredensial:
   - **Username**: `adminfatek`
   - **Password**: `12345678`
3. Klik menu **RKAKL** di sidebar kiri.
4. Klik tombol **Tambah Laporan**:
   - **Tahun**: `2026`
   - **Pagu BOPTN**: `500.000.000` (Rp 500 Juta)
   - **Pagu BLU**: `500.000.000` (Rp 500 Juta)
   - Simpan data.
5. Klik tombol **Akses** (atau ikon kunci/akses) pada tabel:
   - Ubah status menjadi **Buka Akses Input** agar Unit Kerja dapat mulai mengisi anggaran.
6. Logout dengan klik tombol **Keluar** di sidebar kiri bawah.

#### 🔍 Behind-the-Scenes (Database & API):
- Frontend memanggil `POST /api/laporan/create`.
- Data tersimpan di tabel `laporans` dan `total_pagus` dengan `sisa_pagu_boptn` dan `sisa_pagu_blu` awal senilai Rp 500 Juta.

---

### SKENARIO 2: Pengusulan Pohon Anggaran (RKAKL)
> **Aktor**: `akademikfatek` (Unit Kerja)  
> **Tujuan**: Merinci pos belanja kegiatan unit kerja ke dalam hierarki rekening.

#### Langkah Pengujian:
1. Login dengan:
   - **Username**: `akademikfatek`
   - **Password**: `12345678`
2. Masuk ke menu **RKAKL**.
3. Klik tombol **Lihat / Detail** (ikon mata) pada laporan tahun 2026 yang sudah dibuat Keuangan.
4. Anda akan melihat struktur pohon anggaran (Level 1 s/d Level 6).
5. Tambahkan Pos Anggaran Belanja (misalnya: *Belanja Bahan Praktikum Mahasiswa*):
   - Pilih kode rekening induk.
   - Input target alokasi dana, misalnya: `Rp 50.000.000`.
   - Simpan data.
6. Periksa apakah nominal total pagu yang dialokasikan terakumulasi dengan benar di layar.

#### 🔍 Behind-the-Scenes (Database & API):
- Data hierarki tersimpan pada tabel relasional `levels_1` hingga `levels_6` serta `kode_uang_masters`.
- Sistem membatasi agar alokasi unit kerja tidak melebihi pagu total fakultas.

---

### SKENARIO 3: Pengajuan Jadwal Penarikan Dana (RPD & Kalender)
> **Aktor**: `akademikfatek` (Unit Kerja)  
> **Tujuan**: Menentukan di bulan apa dana kegiatan akan ditarik dari bendahara.

#### Langkah Pengujian:
1. Masih dalam akun `akademikfatek`, buka menu **RPD** (atau navigasi sub-menu RPD).
2. Pilih pos belanja yang tadi telah dibuat (Rp 50.000.000).
3. Klik **Tambah RPD**:
   - **Nama Kegiatan**: `Pengadaan Modul Praktikum Semester Genap`
   - **Nominal Penarikan**: `20.000.000` (Rp 20 Juta)
   - **Bulan / Tanggal**: Pilih tanggal di bulan berjalan (misal: 15 Maret).
   - Simpan data.
4. Buka tab/tampilan **Kalender**:
   - Pastikan muncul jadwal rencana penarikan dana pada tanggal yang dipilih.

#### 🔍 Behind-the-Scenes (Database & API):
- Frontend memanggil `POST /api/rpd/create`.
- Data tersimpan di tabel `rpds` yang terhubung ke `id_kode_uang` dan `id_user`.
- Kalender membaca data ini untuk mencegah penumpukan pencairan dana pada hari yang sama.

---

### SKENARIO 4: Pencairan Uang Muka (Panjar)
> **Aktor**: `adminfatek` (Keuangan)  
> **Tujuan**: Mentransfer uang muka operasional kepada unit pengusul kegiatan.

#### Langkah Pengujian:
1. Logout dari Unit Kerja, lalu login kembali sebagai Keuangan (`adminfatek`).
2. Masuk ke menu **RPD**, cari pengajuan `Pengadaan Modul Praktikum Semester Genap`.
3. Klik tombol **Detail RPD**.
4. Di bagian tabel Panjar, klik tombol **Tambah Panjar**:
   - **Nominal Panjar**: `20.000.000` (atau termin awal misal Rp 10.000.000).
   - **Keterangan**: `Pencairan Uang Muka Tahap 1 Belanja Bahan`.
5. *(Fitur Keamanan)*: Jika sistem meminta verifikasi OTP email, periksa kotak masuk atau cek log terminal backend untuk melihat kode token OTP yang dikirimkan.
6. Masukkan kode OTP dan konfirmasi.
7. Pastikan baris riwayat Panjar berhasil tercatat dengan status pencairan aktif.

#### 🔍 Behind-the-Scenes (Database & API):
- Frontend memanggil `POST /api/panjar/create`.
- Tersimpan di tabel `panjars`.
- Kasbon/panjar ini kini berstatus "belum dipertanggungjawabkan" sampai unit kerja mengunggah LPJ.

---

### SKENARIO 5: Pertanggungjawaban Belanja Riil & Upload Kuitansi (LPJ)
> **Aktor**: `akademikfatek` (Unit Kerja)  
> **Tujuan**: Melaporkan nota kuitansi belanja asli agar panjar ditutup dan tercatat sebagai realisasi sah negara.

#### Langkah Pengujian:
1. Login kembali sebagai `akademikfatek`.
2. Buka menu **RPD** -> Buka **Detail RPD** kegiatan pengadaan tadi.
3. Di bagian panel **LPJ**, klik tombol **Tambah LPJ**:
   - **Nominal Realisasi**: `19.850.000` (misal sisa belanja Rp 150.000 dikembalikan).
   - **Keterangan**: `Pembelian Kertas, Tinta, dan Komponen Praktikum`.
   - **Upload File Bukti**: Pilih file PDF atau gambar kuitansi belanja.
4. Klik **Kirim LPJ**.
5. Verifikasi:
   - Link file kuitansi LPJ dapat dibuka dan dilihat.
   - Status RPD berubah menjadi selesai / terealisasi.

#### 🔍 Behind-the-Scenes (Database & API):
- File kuitansi disimpan ke server di direktori `public/uploads/LPJ/` dan dilayani secara statis via endpoint `/lpj/:filename`.
- Record baru dicatat di tabel `lpjs`.
- Kolom `total_realisasi` pada pos anggaran di-update, dan `sisa_pagu` berkurang.

---

### SKENARIO 6: Evaluasi Dashboard Eksekutif
> **Aktor**: `dekanfatek` (Pimpinan)  
> **Tujuan**: Memantau kesehatan keuangan dan kecepatan serapan anggaran fakultas.

#### Langkah Pengujian:
1. Login sebagai `dekanfatek` (Password: `12345678`).
2. Begitu login, Anda otomatis diarahkan ke `/dashboard`.
3. Periksa visualisasi yang tampil:
   - **Filter Unit Kerja**: Ganti antara "Semua Unit" atau pilih unit tertentu (misal: "Akademik FATEK").
   - **Pie Chart Pagu vs Realisasi**: Pastikan angka realisasi (Rp 19.850.000) dan sisa pagu terefleksi di diagram lingkaran.
   - **Statistik Box**: Cek card "Total Pagu", "Total Realisasi", dan "Sisa Pagu".
4. Verifikasi bahwa akun Pimpinan **tidak memiliki** tombol edit/hapus anggaran (hanya *read-only* pengawasan).

---

## 📋 4. Checklist Matriks Pengujian Fitur

Gunakan tabel ini sebagai lembar cek lis pengujian Anda:

| Modul Fitur | Uji Skenario | Hasil yang Diharapkan | Status |
| :--- | :--- | :--- | :---: |
| **Autentikasi** | Login dengan password salah | Muncul toast merah "Nama Pengguna atau Kata Sandi tidak valid" | [ ] |
| **Autentikasi** | Input password < 8 karakter | Muncul validasi "Kata Sandi minimal 8 karakter" | [ ] |
| **Hak Akses** | Login `adminfatek` (Keuangan) | Muncul menu khusus "Master Data" di sidebar | [ ] |
| **Hak Akses** | Login `akademikfatek` / `dekan` | Menu "Master Data" tersembunyi | [ ] |
| **Laporan/Pagu** | Input Pagu format mata uang | Otomatis terformat Rupiah (cth: `Rp 500.000.000`) | [ ] |
| **RKAKL** | Navigasi pohon anggaran | Level 1 hingga Level 6 terbuka secara hierarkis | [ ] |
| **RPD** | Pengajuan tanggal penarikan | Jadwal muncul di komponen kalender bulanan | [ ] |
| **Panjar** | Verifikasi OTP pencairan | Kode OTP terkirim dan memvalidasi sebelum submit | [ ] |
| **LPJ** | Upload berkas kuitansi | File PDF/gambar terupload dan tautan dapat diklik | [ ] |
| **Dashboard** | Filter unit kerja & tahun | Grafik pie chart reaktif memperbarui proporsi angka | [ ] |
| **Logout** | Konfirmasi modal keluar | Cookie `token` & data sesi terhapus, redirect ke `/login` | [ ] |

---

## 🛠️ 5. Tips & Penanganan Kendala Saat Testing

1. **Server Belum Menyala**:
   Pastikan backend dan frontend aktif:
   ```bash
   # Terminal 1 (Backend Express port 8000)
   cd /home/mint/project-real/kasidoi/service-fatek-untad && npm run dev

   # Terminal 2 (Frontend Next.js port 3000)
   cd /home/mint/project-real/kasidoi/web-fatek-untad && npm run dev
   ```

2. **Koneksi Database Putus**:
   Periksa container MySQL:
   ```bash
   docker ps | grep mysql-server
   ```

3. **Kembali Bersih ke Data Semula (Reset DB)**:
   Jika data uji ingin dikembalikan ke kondisi awal, cukup import ulang file SQL:
   ```bash
   docker exec -i mysql-server mysql -u root -p1234 keuangan-fatek < /home/mint/project-real/kasidoi/service-fatek-untad/keuangan.sql
   ```
