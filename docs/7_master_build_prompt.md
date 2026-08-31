# Prompt Induk: Pembangunan Aplikasi Berger WMS (untuk sesi Claude Opus 5)

> **Status dokumen:** DRAFT — untuk direview dulu oleh Alfian sebelum dipakai.
> **Cara pakai:** salin seluruh isi di bawah section "PROMPT" sebagai pesan pertama ke sesi Claude (model Opus 5), di root repo `berger-wms`.

---

## Kenapa dokumen ini ada

Tujuannya satu: supaya pembangunan sisa aplikasi berjalan **runtut per modul**, bukan meloncat-loncat antar file yang tidak berhubungan. Modul User Management (lihat commit `2423c46`, branch `backend/feature-user-management`) sudah selesai penuh dari migration sampai test — itu jadi **pola/template** yang harus ditiru untuk setiap modul berikutnya: migration → model → factory/seeder → Form Request → controller di-wire ke DB → view real data → feature test → Pint bersih → commit.

Prompt ini mengunci urutan fase berdasarkan **urutan dependensi FK sebenarnya** di `docs/2_database_design.md` §8 ("Urutan Migration"), bukan urutan yang terasa penting secara subjektif.

---

## PROMPT

```
Kamu adalah Senior Full-Stack Engineer yang melanjutkan pembangunan Berger WMS
(Sistem WMS & Sales Order Portal — PT Berger Paints Indonesia), Laravel 13 + PostgreSQL 16.

BACA DULU sebelum menulis kode apa pun, dalam urutan ini:
1. docs/0_ai_agent_instructions.md  — aturan kerja, konvensi commit, tech stack
2. docs/1_prd.md                   — kebutuhan fungsional per modul (§6.1–§6.10),
                                      aturan bisnis (§7), RBAC (§5)
3. docs/2_database_design.md       — skema tabel (§3), urutan migration wajib (§8)
4. docs/4_ui_ux_guidelines.md      — desain visual yang HARUS dipertahankan
5. docs/5_testing_strategy.md      — strategi test per modul
6. docs/6_cicd_docker_setup.md     — konvensi branch, pipeline CI/CD

STATUS SAAT INI (jangan dikerjakan ulang):
- Fase 0 "Keamanan Dasar & Manajemen User" SUDAH SELESAI di branch
  backend/feature-user-management (commit 2423c46): tabel roles, departments,
  warehouses, plus kolom manajemen user di tabel users. Controller
  app/Http/Controllers/Wms/UserController.php, view
  resources/views/wms/admin/users.blade.php, 21 feature test — semua hijau.
- Fase 1 "Autentikasi Nyata" SUDAH SELESAI (PR #2 + #3, merged ke develop):
  login/logout sungguhan (AuthController), progressive lockout 3x gagal ->
  5/10/30/60/120 menit, max 2 device + idle timeout 1 jam (TrackUserSession),
  pemisahan portal WMS vs Sales (EnsurePortalAccess), tabel user_sessions dan
  login_attempts. Role Switcher di navbar SUDAH DIHAPUS. 41 test hijau.
- CurrentActor tetap ada sebagai satu-satunya penentu aktor, tapi jalur
  utamanya kini auth()->user() sungguhan; jalur ?as=<slug> dipagari
  app()->environment('production') dan praktis tak terjangkau lewat HTTP.
- Fase 1b "Verifikasi Anti-Bot" SUDAH SELESAI: widget Google reCAPTCHA v2
  menyatu di form login yang sama (bukan halaman terpisah), diverifikasi
  lewat Http::asForm()->post() langsung ke siteverify (tanpa package
  tambahan) di AuthController::verifyRecaptcha(). Kegagalannya (kosong,
  tidak dicentang, ditolak Google) masuk counter lockout yang sama dengan
  password salah. Site key & secret key ada di .env (RECAPTCHA_SITE_KEY /
  RECAPTCHA_SECRET_KEY), TIDAK ikut commit. Test memakai Http::fake() —
  tidak pernah memanggil Google sungguhan. MFA/TOTP sudah DIHAPUS TOTAL dari
  rencana; jangan hidupkan lagi.
- RBAC sidebar & route SUDAH SELESAI: App\Support\Permission adalah matriks
  tunggal (fitur -> daftar role) yang dipakai bersama oleh sidebar (@can di
  layouts/wms.blade.php) dan middleware route (can:<fitur> di routes/web.php).
  Menambah menu/route baru WAJIB lewat matriks ini, jangan menulis pengecekan
  role langsung di Blade atau controller. Portal Sales memakai navigasi
  hibrida: bottom nav < 992px, sidebar >= 992px (docs/4 §3.1).
- SEMUA controller Wms/Sales lain (Inbound, Inventory, Outbound, Billing,
  Master, SalesOrder, Notification, Report, Epod) MASIH mengembalikan
  data dummy/hardcoded ke view. Belum ada migration untuk tabel bisnisnya.
  Verifikasi ini benar dulu dengan membaca controller & migration yang ada
  sebelum mempercayai klaim ini secara buta — kondisi bisa saja sudah berubah.

ATURAN KERJA WAJIB (baca sampai habis, ini yang paling penting):

1. SATU FASE = SATU FOKUS. Kerjakan fase yang sedang berjalan sampai TUNTAS
   (checklist Definition of Done di bawah 100% tercentang) sebelum menyentuh
   file yang menjadi domain fase lain. Jangan lompat memperbaiki modul lain
   "sambil lewat", walau kelihatan kecil — catat di TODO, tunda ke fasenya.

2. IKUTI URUTAN FASE di bawah — jangan diacak. Urutan ini mengikuti urutan
   migration wajib di docs/2_database_design.md §8 (dependensi foreign key
   sebenarnya, sudah diverifikasi konsisten dengan skema).

3. DEFINITION OF DONE per fase (checklist ini berlaku untuk SETIAP fase,
   tanpa kecuali):
   [ ] Migration dibuat, `php artisan migrate` sukses tanpa error urutan FK
   [ ] Model Eloquent + relasi + factory (+ seeder bila perlu data acuan awal)
   [ ] Form Request untuk validasi; pakai withValidator() untuk aturan
       lintas-field/bisnis (pola: app/Http/Requests/Wms/StoreUserRequest.php)
   [ ] Controller di-wire penuh ke database — TIDAK ADA lagi array
       hardcoded/dummy tersisa di controller maupun view untuk modul ini
   [ ] View Blade menampilkan data asli, TAPI desain visual (HTML/CSS/JS
       existing) dipertahankan — ini penyesuaian data, bukan desain ulang
   [ ] Feature test menutupi: akses per role (RBAC docs/1 §5.2), validasi
       input, dan aturan bisnis khusus modul ini (docs/1 §7)
   [ ] `php artisan test` hijau, tidak mengganggu test fase-fase sebelumnya
   [ ] `./vendor/bin/pint` bersih (PASS)
   [ ] Commit di branch fase ini, pesan sesuai konvensi docs/0 §3.3
       (format: <type>(<scope>): <deskripsi>)
   [ ] Laporan checkpoint ditulis (format di bawah) sebelum lanjut fase berikutnya

4. JIKA SPESIFIKASI TIDAK SESUAI KENYATAAN saat implementasi (kolom di
   docs/2 ternyata tidak cukup, aturan di docs/1 ambigu, dsb) — BERHENTI,
   laporkan gap-nya dengan opsi konkret, TUNGGU konfirmasi sebelum mengubah
   dokumen atau menyimpang dari spesifikasi. Jangan menebak sendiri lalu jalan terus.

5. Branch pendek umur per fase: `feat/<nama-fase-singkat>` (contoh:
   feat/auth-real-login, feat/master-produk-pelanggan, feat/inbound-module).
   Alur wajib begitu Definition of Done fase tercentang semua DAN user
   sudah menyetujui laporan checkpoint (lihat aturan #7):
     a. `git push -u origin feat/<nama-fase>`
     b. Buka Pull Request ke `develop` (base: develop)
     c. TUNGGU GitHub Actions job "CI — Lint & Test" selesai dan HIJAU
        di tab Checks PR tersebut — ini bukti CI sungguhan, bukan sekadar
        `php artisan test` lokal
     d. Baru merge PR ke develop, lalu hapus branch fitur-nya
   Jangan biarkan branch menumpuk berhari-hari tanpa di-push/PR — begitu
   satu fase selesai, langsung lewati siklus di atas sebelum mulai fase
   berikutnya, supaya `develop` selalu jadi fondasi yang sudah tervalidasi
   CI untuk fase selanjutnya.

6. Pakai ulang konvensi yang SUDAH ada di codebase, jangan reinvent:
   - app/Support/CurrentActor.php untuk "siapa aktor saat ini" (sampai Fase 1
     menggantinya dengan session Auth Laravel sungguhan)
   - Role::scopeAssignableBy() untuk menyembunyikan/menolak akses ke slug
     super_admin dari non-Super-Admin
   - User::scopeSearch() dengan ILIKE (PostgreSQL, case-insensitive)
   - cast 'encrypted' untuk kolom sensitif bila kelak ada (saat ini tidak ada)
   - soft deletes untuk tabel transaksional yang butuh jejak audit
     (lihat docs/2 §6 strategi archival)
   - Pint dijalankan SEBELUM commit, bukan sesudah CI menegur

7. Setiap fase WAJIB ditutup dengan laporan checkpoint memakai format ini,
   LALU BERHENTI TOTAL dan TUNGGU persetujuan eksplisit dari user sebelum
   menyentuh satu baris kode pun dari fase berikutnya. Jangan mulai FASE
   [n+1] hanya karena checklist FASE [n] sudah centang semua — harus ada
   kata "lanjut"/"setuju"/persetujuan sejenis dari user dulu:

   ---
   FASE [n]: [nama fase] — SELESAI, MENUNGGU PERSETUJUAN
   Dibangun: [3-5 baris ringkas: migration, controller, view yang di-wire]
   Migration baru: [daftar file]
   Test: [X passed, Y assertions]
   Pint: [PASS/FAIL]
   Deviasi dari dokumen (jika ada): [...]
   Lanjut ke FASE [n+1]: [nama]? (menunggu persetujuan Anda)
   ---

PETA FASE (urut, jangan diacak):

FASE 0 — SELESAI, jangan diulang. (Roles/Departments/Warehouses/Users)

FASE 1 — SELESAI, jangan diulang. (Login, Session, Lockout, Portal split)

FASE 1b — SELESAI, jangan diulang. (Verifikasi Anti-Bot / Google reCAPTCHA v2)

FASE 2 — Master Data: Produk & Pelanggan
  2a. Produk — SELESAI. Tabel product_categories + products, ProductController,
      halaman Master Produk terhubung DB, App\Support\PalletCapacity.
      Catatan penting untuk fase berikutnya:
        - products TIDAK punya kolom stok. Kolom "Inventory" pada ekspor ERP
          adalah hasil SUM; stok tinggal di inventory_stocks (Fase 4). Ada test
          regresi yang menggagalkan build bila kolom stok menyelinap masuk.
        - sku = 'ID1-F' + product_code + shade_code + pack_code (lihat
          Product::buildSku). Ketiga komponen tetap disimpan terpisah.
        - max_qty_per_pallet NULLABLE dan dihitung dari PalletCapacity memakai
          pack_size (ukuran WADAH), BUKAN unit_volume (isi sebenarnya). Pail
          "20Ltr" berisi 19.4 L tetap 27 pcs/palet. Ukuran di luar aturan
          gudang dibiarkan NULL, bukan ditebak.
        - Product Type "Tidak ditemukan" dari ERP -> category_id NULL, jangan
          dibuatkan kategori bernama itu.
  2b. Pelanggan — SELESAI. Tabel customers + payment_terms,
      CustomerController, halaman Master Pelanggan terhubung DB.
      Catatan penting untuk fase berikutnya:
        - Kolom mengikuti ekspor ERP: code (No./id), ship_to_code, name,
          phone, contact_name, email, address, address_2, territory_code.
        - address & address_2 disimpan TERPISAH (setara ekspor ERP) tapi
          ditampilkan digabung lewat accessor full_address.
        - customers TIDAK punya default_payment_term maupun credit_limit.
          Termin dipilih Sales per-pesanan, bukan sifat tetap pelanggan —
          keduanya tinggal di tabel payment_terms. Ada test regresi yang
          menggagalkan build bila kolom itu menyelinap kembali.
        - payment_terms sudah terisi (cash/transfer/tempo_30/60/90) dan siap
          dipakai dropdown form Buat Pesanan pada Fase 5.
        - MasterController DIHAPUS; kedua halaman master kini punya
          controller sendiri (ProductController, CustomerController).
  2d. Impor Excel — SELESAI. ImportController + App\Support\Import\*
      (SpreadsheetReader, Importer, ProductImporter, CustomerImporter).
      Catatan penting:
        - Memakai phpoffice/phpspreadsheet LANGSUNG. maatwebsite/excel belum
          mendukung Laravel 13; composer justru menawarkan v1.1.5 (2014) yang
          bergantung pada paket yang sudah ditinggalkan. Jangan pasang itu.
        - Alur DUA TAHAP: preview (tidak menyentuh DB sama sekali) lalu store.
          Impor bersifat memperbarui data lama, jadi pratinjau adalah pengaman
          agar berkas keliru tidak menimpa data yang benar.
        - Judul kolom dicocokkan setelah dinormalkan ("Base Unit of Measure"
          -> base_unit_of_measure), sehingga beda huruf besar/kecil dan spasi
          tidak menggagalkan impor.
        - Kolom Inventory pada ekspor ERP DIABAIKAN (stok bukan data master).
        - Impor ulang TIDAK menghidupkan kembali data yang sengaja
          dinonaktifkan Manager.
  2c. Lokasi Rak — SELESAI. Tabel locations, LocationController, halaman
      Master Lokasi Rak, LocationSeeder (2.264 bin untuk WH-01).
      Catatan penting untuk Fase 3 (put-away):
        - Kode berpola [Rak]-[Level]-[Sel]; rack/level/cell DITURUNKAN dari
          code lewat Location::parseCode, jadi tidak bisa tidak sinkron.
        - level & cell bertipe ANGKA, bukan string. Selalu urutkan dengan
          scopeInStorageOrder() — mengurutkan lewat string code menaruh
          B-01-10 sebelum B-01-02.
        - code unik PER GUDANG (warehouse_id + code), bukan global.
        - Zona (Fast/Slow/Middle Moving Area) menentukan strategi put-away.
          Ejaan ERP "Midle" dinormalkan Location::normalizeZone().
        - Tidak ada Rak "A" pada denah gudang. Level 4-5 memuat lebih banyak
          sel daripada Level 1-3 di sebagian besar rak.
        - Halaman DENAH GUDANG (/wms/master/locations/map) menampilkan peta
          visual seluruh bin: Level 5 di atas, warna per zona, klik bin untuk
          detail, kotak pencarian untuk melacak bin tertentu. Rute /map
          didaftarkan SEBELUM /locations/{location} agar "map" tidak
          tertangkap route model binding.

  KEPUTUSAN UNTUK FASE 4 (sudah dikonfirmasi pemilik produk, jangan ditanya
  ulang):
    - DIREVISI 2026-08-31 (lihat catatan Fase 3b di bawah): SATU BIN = SATU
      PALET. Baris "beberapa produk & batch sekaligus" di bawah ini SUDAH
      TIDAK BERLAKU — dipertahankan hanya sebagai jejak, jangan diikuti.
      ~~Satu bin boleh memuat BEBERAPA produk dan BEBERAPA batch sekaligus.
      Jangan tambahkan unique constraint pada location_id.~~
    - Koreksi stock opname WAJIB menyertakan alasan; dicatat sebagai
      stock_movements bertipe ADJUSTMENT dengan notes wajib + user_id.
    - Denah gudang adalah antarmuka opname-nya: tiap kotak bin sudah punya
      slot indikator keterisian yang tinggal diisi begitu inventory_stocks
      dibangun. Lihat komentar "Slot isi bin (FASE 4)" di
      resources/views/wms/master/locations-map.blade.php.

FASE 3 — Inbound (Barang Masuk)
  Dikerjakan bertahap atas permintaan pemilik produk.

  3a. Input Produksi (F-INB-01) — SELESAI. Tabel inbound_headers +
      inbound_details, App\Support\Inbound\ProductionSheet,
      App\Support\DocumentNumber, halaman create + preview.
      Catatan penting:
        - Nomor dokumen & tanggal DIBANGKITKAN SISTEM (format IN-YYMMDD-NNN),
          berbeda dari rancangan PRD lama yang menyebutnya input manual.
        - Berkas Excel memuat kolom A-L tapi HANYA A-E yang dibaca:
          A No.=nomor order produksi, B Source No.=SKU, C Description,
          D Quantity, E QC Number=batch.
        - Satu batch bisa mencakup BEBERAPA nomor order produksi — batch_no
          sengaja tidak unik.
        - inbound_details = SATU BARIS PER PALET. Qty dipecah saat menyimpan
          memakai PalletCapacity::split (PRD §7.1).
        - SKU tak dikenal DITOLAK barisnya; master produk TIDAK diisi otomatis
          dari berkas produksi agar salah ketik tidak jadi produk permanen.
        - Berkas Excel DIHAPUS setelah disimpan; sistem hanya menyimpan hasil
          pembacaannya.
        - Aturan palet 5 Liter ditambahkan (=180, setara 5 Kg) setelah data
          produksi nyata memuat kemasan itu.
        - Riwayat Produksi (daftar + detail) SUDAH terhubung DB. Detail dicari
          lewat document_number, bukan id, agar URL cocok dengan nomor yang
          tercetak di dokumen fisik.
        - HATI-HATI menjumlahkan qty: total_qty sengaja BERULANG di tiap palet
          yang berasal dari satu baris produksi (235 tertulis di palet 1 dan
          palet 2). Selalu jumlahkan pallet_qty. Ada test regresi untuk ini.
        - Satu dokumen memuat BANYAK batch; kolom batch di daftar menampilkan
          daftar unik, bukan satu nilai.

  3b. Put-away (F-INB-02) — SELESAI. putawayIndex/putawayProcess/putawayStore
      terhubung DB, halaman putaway-list + putaway-process.
      Catatan penting:
        - PUT-AWAY BOLEH SEBAGIAN. Palet yang lokasinya dikosongkan dilewati,
          tidak menggagalkan penyimpanan palet lain. Pekerjaan fisik di lantai
          gudang lazim terputus di tengah giliran kerja.
        - Status dokumen naik ke verification_pending HANYA bila SELURUH palet
          punya location_id (InboundHeader::isFullyPlaced). Menaikkan status
          saat baru separuh ditempatkan akan membuat Logistik memverifikasi
          barang yang belum ada di raknya. Ada test regresi untuk ini.
        - Bin DIBATASI ke warehouse_id dokumen. Kode rak seperti "B-01-01"
          berulang antar gudang, jadi salah gudang tidak terlihat sampai
          barangnya dicari.
        - Operator mengetik KODE bin (bukan memilih id); occupancy dipetakan
          per kode agar pencocokan di layar langsung.
        - SATU BIN = SATU PALET (direvisi 2026-08-31, membatalkan keputusan
          Fase 4 lama di atas). unique index pada inbound_details.location_id.
          Bin yang sudah terisi TIDAK muncul di rekomendasi lokasi untuk palet
          lain, dan mencoba menyimpan ke bin yang sudah terisi ditolak.
          Rekomendasi menampilkan isi bin (qty) bila diketik manual.
        - Qty Aktual boleh dikoreksi Operator; SKU & batch dikunci karena
          berasal dari dokumen produksi.
        - DIBUANG dari mock lama: simulasi QR scanner (rak acak) dan pemecahan
          otomatis "kapasitas rak 180 pail". Palet SUDAH menjadi satuan
          kapasitas — pemecahan terjadi di Fase 3a, bukan di sini.
  3c. Verifikasi Maker-Checker (F-INB-03) — BELUM.

  Ruang lingkup asli:
  Migration: inbound_headers, inbound_details
  Ruang lingkup: docs/1 §6.3. Wire InboundController: create/preview excel,
  putaway (Qty Aktual editable — F-INB-02), verify. Aturan palet otomatis §7.1.

FASE 4 — Inventory & Stok
  Migration: inventory_stocks, stock_movements
  Ruang lingkup: docs/1 §6.4, §7.2 (FIFO), §7.2.1 (Expiry & DDP).
  Wire InventoryController: index, adjust, transfer antar gudang.
  PENTING: inventory_stocks.sales_return_detail_id dibuat nullable TANPA FK
  constraint dulu (constraint-nya baru di Fase 7, lihat catatan sirkular
  docs/2 §8) — jangan tukar urutan ini.

FASE 5 — Sales Order Portal
  Migration: sales_orders, sales_order_details, sales_order_allocations
  Ruang lingkup: docs/1 §6.5 (bagian order masuk) + modul Sales Portal.
  Wire SalesOrderController: create/store new-order, history (my-orders).
  Aturan cutoff waktu order §7.5, partial fulfillment & lost sales §7.3.

FASE 6 — Outbound (Approval -> Picking -> Delivery -> Verifikasi)
  Migration: delivery_notes, delivery_proofs
  Ruang lingkup: docs/1 §6.5 (bagian proses gudang). Wire OutboundController
  penuh: approval, picking batching, picking, generate surat jalan, verifikasi
  bukti kirim.

FASE 7 — Retur (Penolakan Sales -> Retur Gudang)
  Migration: sales_returns, sales_return_details,
             add_sales_return_fk_to_inventory_stocks_table (FK susulan)
  Ruang lingkup: docs/1 §6.10 — PERHATIKAN tabel terminologi: Sales
  melaporkan PENOLAKAN (SalesOrderController::reportReturn), gudang yang
  memproses RETUR (InboundController::returnsIndex/processReturn). Jangan
  tertukar istilah di kode maupun pesan UI.

FASE 8 — Billing (Penagihan)
  Migration: customer_billings, billing_payments
  Ruang lingkup: docs/1 §6.6, §7.4. Wire BillingController. Overdue
  customer = PERINGATAN VISUAL saja, BUKAN blokir order (lihat docs/1 §6.6,
  keputusan yang sudah dikonfirmasi user sebelumnya).

FASE 9 — Tracking, Notifikasi Real-time, Audit Log
  Migration: order_trackings, notifications, audit_logs
  Ruang lingkup: docs/1 §6.8, §6.9. Wire NotificationController + broadcast
  via Soketi (sudah jalan di docker-compose). Audit log mencatat aksi
  sensitif (create/update/deactivate user, approve order, dst).

FASE 10 — Pengaturan Sistem & Penomoran Dokumen
  Migration: system_settings, document_sequences
  Ruang lingkup: wire app/Http/Controllers/Wms/AdminController::sequence()
  (view resources/views/wms/admin/sequence.blade.php sudah ada, masih dummy)
  ke tabel document_sequences untuk penomoran otomatis SJ/faktur/dsb.

FASE 11 — Dashboard & Laporan
  Ruang lingkup: docs/1 §6.7. Wire DashboardController (admin/produksi/
  operator) dan ReportController ke query agregat dari tabel-tabel yang
  sudah dibangun Fase 1-10 — bukan angka statis.

FASE 12 — E-POD (Electronic Proof of Delivery)
  Ruang lingkup: pastikan EpodController::show/confirm terhubung ke
  delivery_proofs (Fase 6) secara konsisten dari sisi customer-facing.

FASE 13 — Pengujian End-to-End & Pengerasan
  Jalankan 5 alur end-to-end penuh (order -> inbound -> putaway -> outbound
  -> billing, dst — rujuk laporan onboarding sebelumnya bila ada). Tambah
  test regresi lintas modul yang belum tercakup test per-fase.

FASE 14 — Finalisasi CI/CD & Persiapan Go-Live
  Validasi pipeline docs/6 end-to-end untuk seluruh modul baru. Pastikan
  jalur dev ?as=<slug> di CurrentActor benar-benar mati di production.
  Review keamanan §8.2 dan performa §8.1 PRD sebelum dianggap selesai.

MULAI DARI: verifikasi ulang bagian "STATUS SAAT INI" di atas masih akurat
(jalankan `git log --oneline -5` dan `php artisan migrate:status`), lalu
mulai fase pertama yang BELUM selesai. Jangan mulai menulis kode sebelum
langkah verifikasi ini dan pembacaan dokumen di atas selesai.
```

---

## Catatan untuk Alfian (bukan bagian prompt, boleh dihapus sebelum dipakai)

- **Kenapa Autentikasi jadi Fase 1, bukan paling akhir?** Karena `CurrentActor` eksplisit adalah alat sementara (per catatan di `docs/0` §5.3), dan hampir semua fase berikutnya butuh "siapa aktor saat ini" yang valid. Lebih murah menggantinya lebih awal daripada merapikan puluhan tempat pemanggilan nanti.
- **Kenapa `departments` tidak muncul di urutan migration docs/2 §8?** Karena tabel itu ditambahkan belakangan atas permintaan Anda (di luar rencana awal dokumen) — sudah beres di Fase 0, tidak perlu tindakan lagi, saya cantumkan di bagian "STATUS SAAT INI" supaya sesi baru tidak bingung kenapa jumlah tabelnya tidak cocok dengan dokumen.
- **Soal checkpoint report:** sudah diubah sesuai keputusan Anda — Opus **melapor lalu BERHENTI TOTAL**, menunggu kata persetujuan eksplisit sebelum menyentuh fase berikutnya. Ini lebih lambat tapi lebih aman: setiap fase bisa Anda review dulu (termasuk cek langsung ke database via psql/DBeaver) sebelum fase berikutnya menumpuk di atasnya.
- **Estimasi skala:** ini ~14 fase, masing-masing kira-kira setara dengan modul User Management yang baru selesai (migration+model+controller+view+test). Dengan mode tunggu-approve, ini realistis berarti ~14 kali jeda percakapan terpisah. Dokumen ini sengaja dibuat self-contained (menyebutkan status terkini secara eksplisit) supaya sesi baru — atau sesi lanjutan setelah approve Anda — bisa langsung lanjut tanpa kehilangan konteks.
- **Belum dijawab:** branch `backend/feature-user-management` saat ini hanya ada di lokal, belum di-push ke GitHub (`git checkout -b` adalah operasi lokal murni, push adalah langkah terpisah yang belum dijalankan). Putuskan dulu apakah mau di-push/merge sebelum FASE 1 dimulai, supaya riwayat branch per-fase nanti rapi sejak awal.
