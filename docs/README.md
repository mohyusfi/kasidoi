# Dokumentasi Sistem Keuangan Kasidoi (Fatek Untad)

Folder ini berisi seluruh panduan proses bisnis, kamus istilah, arsitektur database, dan contoh implementasi pohon anggaran untuk sistem keuangan kampus Fakultas Teknik Untad.

---

## 📚 Daftar Dokumen

| Dokumen | Deskripsi |
| :--- | :--- |
| 📊 **[diagram-kasidoi.drawio](./diagram-kasidoi.drawio)** | **Diagram Lengkap Sistem (Draw.io)**: File Draw.io multi-tab yang memuat 15 tab diagram arsitektur: ERD database (17 tabel lengkap PK/FK, grouped & monokrom), 7 Sequence Diagram alur bisnis non-teknis, dan 7 Use Case Diagram (Overview sistem, per aktor, dan per modul transaksi). |
| 📌 **[ALUR_DISTRIBUSI_ANGGARAN.md](./ALUR_DISTRIBUSI_ANGGARAN.md)** | **Alur Distribusi & Proses Bisnis End-to-End**: Menjelaskan bagaimana dana dari Total Pagu Fakultas mengalir ke Jurusan/Lab, lengkap dengan teks narasi dan pemetaan penyimpanan data ke tabel/kolom database. |
| 🌳 **[contoh_kasus_pohon_anggaran.md](./contoh_kasus_pohon_anggaran.md)** | **Studi Kasus Pohon Anggaran Kampus**: Contoh riil penganggaran Lab Jaringan & Sertifikasi Mahasiswa Teknik Informatika (Pagu Rp 150 Juta) dengan visualisasi diagram hierarki Mermaid. |
| 🛠️ **[DESAIN_SKEMA_SINGLE_TABLE_LEVEL.md](./DESAIN_SKEMA_SINGLE_TABLE_LEVEL.md)** | **Rancangan Single-Table Hierarchy**: Analisis dan rekomendasi penyederhanaan tabel `levels_1` s/d `levels_7` menjadi satu tabel tunggal rekursif (`parent_id`) dengan kolom enum `type`. |
| 📖 **[ISTILAH_PROYEK.md](./ISTILAH_PROYEK.md)** | **Kamus Istilah Keuangan Kampus**: Penjelasan istilah birokrasi seperti Pagu, BOPTN, BLU, MAK, RPD, Panjar, dan LPJ dengan analogi sehari-hari. |
| 🗄️ **[DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)** | **Dokumentasi ERD & Tabel Saat Ini**: Penjelasan relasi antartabel database aktif di `service-fatek-untad`. |

---

## 🎨 Panduan Membuka Diagram Draw.io (`diagram-kasidoi.drawio`)

File diagram dapat dibuka menggunakan:
1. **Aplikasi Draw.io Desktop** atau melalui web di [app.diagrams.net](https://app.diagrams.net).
2. Buka menu **File** > **Open From** > **Device**, lalu pilih file `docs/diagram-kasidoi.drawio`.
3. Gunakan tab-bar di bagian bawah untuk berpindah antar diagram:
   - **Tab 1**: `ERD - Database KASIDOI` (17 tabel lengkap beserta relasi)
   - **Tab 2 - 8**: Sequence Diagram (Autentikasi/OTP, Setup Laporan & Pagu, Setup RKAKL & SBU, Pengajuan RAB, Pengajuan RPD & Kalender, Pencairan Panjar & OTP, serta Verifikasi LPJ)
   - **Tab 9 - 15**: Use Case Diagram (Overview sistem, Aktor Unit Kerja, Aktor Staf Keuangan, Aktor Pimpinan, Modul Perencanaan & RKAKL, Modul Transaksi & Panjar, serta Modul LPJ & Realisasi)

