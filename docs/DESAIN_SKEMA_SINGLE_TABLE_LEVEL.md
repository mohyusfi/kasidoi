# Rancangan Desain Database: Single Table Hierarchy (`levels` / `budget_nodes`)

Dokumen ini memuat analisis dan rekomendasi teknis untuk merampingkan 7 tabel terpisah (`levels_1` s/d `levels_7`) menjadi **1 tabel hierarki tunggal** dengan pola *Adjacency List* rekursif dan penanda kategori bertipe Enum.

---

## 1. Masalah pada Skema Sekarang (7 Tabel Terpisah)

Skema database saat ini menggunakan 7 tabel terpisah:
- `levels_1`, `levels_2`, `levels_3`, `levels_4`, `levels_5`, `levels_6`, `levels_7`.

### Dampak Negatif:
1. **Redundansi Foreign Key**:
   Tabel `Level_6` harus menampung `id_level_1`, `id_level_2`, `id_level_3`, `id_level_4`, dan `id_level_5` hanya untuk melacak garis silsilahnya.
2. **Kekakuan Struktur (Inflexibility)**:
   Jika di masa depan kampus butuh kedalaman level ke-8 atau sub-kegiatan tambahan, sistem harus migrasi database besar dan menambah controller/service baru.
3. **Kode Backend Gemuk**:
   Ada 7 pasang Controller, Service, dan Repository terpisah yang logikanya hampir 95% identik (hanya beda nama tabel).

---

## 2. Solusi: 1 Tabel Rekursif (*Self-Referencing*)

Menggabungkan semua level ke dalam satu tabel: **`budget_nodes`** (atau tetap dinamakan **`levels`**).

### A. Analogi Folder Komputer
Sama persis seperti sistem folder di Windows Explorer atau Google Drive:
- Setiap folder berada di tempat yang sama.
- Folder hanya perlu tahu satu hal: **"Siapa folder induk saya?" (`parent_id`)**.
- Jika `parent_id = null`, berarti dia adalah folder paling atas (Level 1 / Fakultas).

### B. Mengapa Memakai Kolom `type` (Enum) Bukan Angka Integer?
- **Jika angka (`level: 6`)**: Rawan tertukar antara level 5 dan 6, kode kurang deskriptif.
- **Jika tipe (`type: POS_BEBAN`)**: Sangat eksplisit (*self-documenting*). Tim backend langsung tahu bahwa pada baris tersebut RPD dan LPJ boleh diajukan.

---

## 3. Rekomendasi Skema Prisma

```prisma
enum BudgetNodeType {
  FAKULTAS       // Level 1: Unit Teratas
  PROGRAM_KERJA  // Level 2: Sasaran Strategis / Renstra
  JURUSAN        // Level 3: Unit Pengelola / Prodi
  KEGIATAN       // Level 4: Program Utama
  SUB_KEGIATAN   // Level 5: Rincian Kegiatan
  POS_BEBAN      // Level 6: Pos Akun Belanja (Sumber RPD & LPJ)
  ITEM_BELANJA   // Level 7: Rincian Item Barang (SBU / Produk)
}

model BudgetNode {
  id          Int            @id @default(autoincrement())
  parent_id   Int?           // Nullable jika node teratas (FAKULTAS)
  kode_uang   String         @db.VarChar(100)
  nama        String         @db.VarChar(255)
  type        BudgetNodeType // Kategori hierarki bisnis
  product_id  Int?           // Opsional: Relasi ke katalog Product jika type == ITEM_BELANJA

  // Relasi Rekursif (Self-Relation)
  parent      BudgetNode?    @relation("NodeHierarchy", fields: [parent_id], references: [id], onDelete: Cascade)
  children    BudgetNode[]   @relation("NodeHierarchy")

  // Relasi Transaksional
  rpds        Rpd[]
  lpjs        Lpj[]
  kode_uang_masters Kode_uang_master[]

  createdAt   DateTime       @default(now())
  updatedAt   DateTime       @updatedAt

  @@index([parent_id])
  @@index([type])
  @@index([kode_uang])
  @@map("budget_nodes")
}
```

---

## 4. Keuntungan Teknis

1. **Query Menjadi Sederhana**:
   - Ambil anak langsung: `prisma.budgetNode.findMany({ where: { parent_id: selectedId } })`.
   - Ambil seluruh pohon dari atas: menggunakan `include: { children: true }` atau *recursive CTE* di database.
2. **Maintenance Sangat Cepat**:
   - Cukup 1 pasang `BudgetNodeController`, `BudgetNodeService`, dan `BudgetNodeRepository`.
