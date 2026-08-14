# Panduan AI Agent Instruction
## Sistem WMS & Sales Order â€” PT Berger Paints Indonesia

> **Versi:** 1.0  
> **Tanggal:** 14 Agustus 2026  
> **Tujuan:** Menjadi acuan utama agar setiap AI Agent yang terlibat dalam pengembangan sistem ini bekerja terarah, konsisten, dan tidak keluar konteks.

---

## 1. Konteks Proyek

Sistem ini adalah **Warehouse Management System (WMS) terintegrasi dengan Sales Order** untuk PT Berger Paints Indonesia. Arsitektur menggunakan **Single Database, Multi-Portal (Monolithic)** dengan satu backend Laravel yang melayani dua portal terpisah: **Portal Warehouse** (desktop-first) dan **Portal Sales** (mobile-first).

### Teknologi Utama
| Layer | Teknologi | Catatan |
|---|---|---|
| Backend Engine | Laravel 11/12+ (PHP 8.3+) | Eloquent ORM, Middleware, Queue |
| Frontend | Laravel Blade + Livewire 3 + Bootstrap 5 | Tidak menggunakan SPA/React/Vue |
| Database | PostgreSQL 16+ | Transaksional, konkurensi tinggi |
| Cache | Redis | Session, cache query, real-time notification |
| Queue | Laravel Horizon + Redis | Background jobs |
| Auth MFA | Google Authenticator (TOTP) | Untuk semua role |
| Realtime | Laravel Echo + Laravel Reverb | Notifikasi real-time dengan suara |
| Infra | Docker + CI/CD (GitHub Actions) | Containerized deployment |

---

## 2. Pembagian Peran AI Agent

### 2.1 Google Gemini 2.5 Pro â€” Frontend Agent

**Scope Tanggung Jawab:**
- Seluruh file Blade template (`resources/views/**/*.blade.php`)
- Seluruh file CSS/SCSS (`resources/css/`, `public/css/`)
- Seluruh file JavaScript frontend (`resources/js/`, `public/js/`)
- Layout dan komponen UI Bootstrap 5
- Responsivitas dan adaptasi mobile/desktop
- Animasi, transisi, dan micro-interaction
- Integrasi notifikasi suara di frontend
- Form validation di sisi client (sebelum dikirim ke server)
- Chart/grafik dashboard menggunakan Chart.js atau ApexCharts

**Batasan Ketat:**
- âŒ TIDAK boleh mengubah file Controller, Model, Migration, Route, atau Middleware
- âŒ TIDAK boleh mengubah logika bisnis di backend
- âŒ TIDAK boleh membuat endpoint API baru
- âŒ TIDAK boleh mengubah konfigurasi database atau `.env`
- âœ… BOLEH membuat partial Blade component baru jika dibutuhkan UI
- âœ… BOLEH menambah JavaScript untuk interaksi UI (AJAX call ke endpoint yang sudah ada)
- âœ… BOLEH mengubah asset frontend (CSS, JS, gambar)

**Panduan Desain:**
```
Portal Sales (Mobile-First):
â”œâ”€â”€ Viewport target: 360px - 428px (smartphone)
â”œâ”€â”€ Touch-friendly: min tap target 44x44px
â”œâ”€â”€ Bottom navigation bar
â”œâ”€â”€ Card-based layout
â”œâ”€â”€ Warna utama: sesuai branding Berger Paints
â””â”€â”€ Font: Inter atau Roboto (Google Fonts)

Portal Warehouse (Desktop-First):
â”œâ”€â”€ Viewport target: 1280px - 1920px (monitor/tablet landscape)
â”œâ”€â”€ Sidebar navigation
â”œâ”€â”€ Data-dense table layout dengan DataTables
â”œâ”€â”€ Form-heavy interface
â”œâ”€â”€ Warna utama: tema profesional/industrial
â””â”€â”€ Font: Inter atau Roboto (Google Fonts)
```

**Konvensi Penamaan File Blade:**
```
resources/views/
â”œâ”€â”€ layouts/
â”‚   â”œâ”€â”€ app-sales.blade.php          # Layout utama portal Sales
â”‚   â”œâ”€â”€ app-warehouse.blade.php      # Layout utama portal Warehouse
â”‚   â””â”€â”€ partials/
â”‚       â”œâ”€â”€ _navbar-sales.blade.php
â”‚       â”œâ”€â”€ _sidebar-warehouse.blade.php
â”‚       â”œâ”€â”€ _notification-bell.blade.php
â”‚       â””â”€â”€ _footer.blade.php
â”œâ”€â”€ sales/                           # Semua view portal Sales
â”‚   â”œâ”€â”€ dashboard.blade.php
â”‚   â”œâ”€â”€ orders/
â”‚   â”‚   â”œâ”€â”€ index.blade.php
â”‚   â”‚   â”œâ”€â”€ create.blade.php
â”‚   â”‚   â”œâ”€â”€ show.blade.php
â”‚   â”‚   â””â”€â”€ tracking.blade.php
â”‚   â”œâ”€â”€ customers/
â”‚   â”‚   â”œâ”€â”€ index.blade.php
â”‚   â”‚   â””â”€â”€ create.blade.php
â”‚   â””â”€â”€ profile/
â”‚       â””â”€â”€ index.blade.php
â”œâ”€â”€ warehouse/                       # Semua view portal Warehouse
â”‚   â”œâ”€â”€ dashboard.blade.php
â”‚   â”œâ”€â”€ inbound/
â”‚   â”œâ”€â”€ inventory/
â”‚   â”œâ”€â”€ orders/
â”‚   â”œâ”€â”€ delivery/
â”‚   â”œâ”€â”€ billing/
â”‚   â””â”€â”€ reports/
â”œâ”€â”€ admin/                           # View khusus Manager/Super Admin
â”‚   â”œâ”€â”€ dashboard.blade.php
â”‚   â”œâ”€â”€ users/
â”‚   â”œâ”€â”€ products/
â”‚   â”œâ”€â”€ locations/
â”‚   â”œâ”€â”€ warehouses/
â”‚   â”œâ”€â”€ customers/
â”‚   â”œâ”€â”€ stock/
â”‚   â”œâ”€â”€ audit-logs/
â”‚   â””â”€â”€ settings/
â”œâ”€â”€ auth/
â”‚   â”œâ”€â”€ login.blade.php
â”‚   â”œâ”€â”€ mfa-verify.blade.php
â”‚   â””â”€â”€ locked.blade.php
â””â”€â”€ components/                      # Blade components reusable
    â”œâ”€â”€ alert.blade.php
    â”œâ”€â”€ modal-confirm.blade.php
    â”œâ”€â”€ status-badge.blade.php
    â”œâ”€â”€ notification-toast.blade.php
    â””â”€â”€ chart-card.blade.php
```

---

### 2.2 Claude Opus â€” Backend Agent

**Scope Tanggung Jawab:**
- Seluruh file PHP backend: Controllers, Models, Migrations, Seeders, Factories
- Route definitions (`routes/web.php`, `routes/channels.php`)
- Middleware (autentikasi, RBAC, rate limiting, session management)
- Service classes dan Repository pattern
- Business logic (FIFO, auto-adjustment, pallet splitting, SLA calculation)
- Database schema design dan migration
- Queue jobs dan event broadcasting
- API internal untuk AJAX calls dari frontend
- Unit test dan feature test
- Konfigurasi Docker, CI/CD, dan environment

**Batasan Ketat:**
- âŒ TIDAK boleh mengubah file Blade template, CSS, atau JavaScript frontend
- âŒ TIDAK boleh mengubah layout atau tampilan UI
- âŒ TIDAK boleh hardcode value yang seharusnya konfigurasi
- âœ… BOLEH mengembalikan data berformat JSON untuk konsumsi AJAX frontend
- âœ… BOLEH membuat Blade component sederhana jika dibutuhkan backend logic (`@can`, `@auth`)
- âœ… BOLEH menambahkan variable ke view melalui Controller/ViewComposer

**Arsitektur Backend:**
```
app/
â”œâ”€â”€ Http/
â”‚   â”œâ”€â”€ Controllers/
â”‚   â”‚   â”œâ”€â”€ Auth/
â”‚   â”‚   â”‚   â”œâ”€â”€ LoginController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ MfaController.php
â”‚   â”‚   â”‚   â””â”€â”€ SessionController.php
â”‚   â”‚   â”œâ”€â”€ Sales/                    # Controller portal Sales
â”‚   â”‚   â”‚   â”œâ”€â”€ DashboardController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ OrderController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ CustomerController.php
â”‚   â”‚   â”‚   â””â”€â”€ ProofUploadController.php
â”‚   â”‚   â”œâ”€â”€ Warehouse/               # Controller portal Warehouse
â”‚   â”‚   â”‚   â”œâ”€â”€ DashboardController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ InboundController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ InventoryController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ OrderApprovalController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ PickingController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ DeliveryNoteController.php
â”‚   â”‚   â”‚   â”œâ”€â”€ BillingController.php
â”‚   â”‚   â”‚   â””â”€â”€ ReportController.php
â”‚   â”‚   â””â”€â”€ Admin/                   # Controller Manager/Super Admin
â”‚   â”‚       â”œâ”€â”€ DashboardController.php
â”‚   â”‚       â”œâ”€â”€ UserController.php
â”‚   â”‚       â”œâ”€â”€ ProductController.php
â”‚   â”‚       â”œâ”€â”€ LocationController.php
â”‚   â”‚       â”œâ”€â”€ WarehouseController.php
â”‚   â”‚       â”œâ”€â”€ CustomerApprovalController.php
â”‚   â”‚       â”œâ”€â”€ StockAdjustmentController.php
â”‚   â”‚       â”œâ”€â”€ AuditLogController.php
â”‚   â”‚       â””â”€â”€ SettingsController.php
â”‚   â”œâ”€â”€ Middleware/
â”‚   â”‚   â”œâ”€â”€ CheckRole.php
â”‚   â”‚   â”œâ”€â”€ CheckMfa.php
â”‚   â”‚   â”œâ”€â”€ EnforceMaxSessions.php
â”‚   â”‚   â”œâ”€â”€ CheckOrderCutoff.php
â”‚   â”‚   â”œâ”€â”€ CheckCustomerBlocked.php
â”‚   â”‚   â””â”€â”€ TrackAuditLog.php
â”‚   â””â”€â”€ Requests/                    # Form Request validation
â”‚       â”œâ”€â”€ StoreInboundRequest.php
â”‚       â”œâ”€â”€ StoreOrderRequest.php
â”‚       â”œâ”€â”€ UploadProofRequest.php
â”‚       â””â”€â”€ ...
â”œâ”€â”€ Models/
â”‚   â”œâ”€â”€ User.php
â”‚   â”œâ”€â”€ Role.php
â”‚   â”œâ”€â”€ Customer.php
â”‚   â”œâ”€â”€ Product.php
â”‚   â”œâ”€â”€ ProductCategory.php
â”‚   â”œâ”€â”€ Location.php
â”‚   â”œâ”€â”€ Warehouse.php
â”‚   â”œâ”€â”€ InboundHeader.php
â”‚   â”œâ”€â”€ InboundDetail.php
â”‚   â”œâ”€â”€ InventoryStock.php
â”‚   â”œâ”€â”€ StockMovement.php
â”‚   â”œâ”€â”€ SalesOrder.php
â”‚   â”œâ”€â”€ SalesOrderDetail.php
â”‚   â”œâ”€â”€ DeliveryNote.php
â”‚   â”œâ”€â”€ DeliveryProof.php
â”‚   â”œâ”€â”€ OrderTracking.php
â”‚   â”œâ”€â”€ CustomerBilling.php
â”‚   â”œâ”€â”€ BillingPayment.php
â”‚   â”œâ”€â”€ Notification.php
â”‚   â”œâ”€â”€ AuditLog.php
â”‚   â”œâ”€â”€ UserSession.php
â”‚   â”œâ”€â”€ LoginAttempt.php
â”‚   â””â”€â”€ SystemSetting.php
â”œâ”€â”€ Services/
â”‚   â”œâ”€â”€ FifoAllocationService.php
â”‚   â”œâ”€â”€ PalletSplitService.php
â”‚   â”œâ”€â”€ StockMovementService.php
â”‚   â”œâ”€â”€ SlaCalculationService.php
â”‚   â”œâ”€â”€ OrderProcessingService.php
â”‚   â”œâ”€â”€ NotificationService.php
â”‚   â”œâ”€â”€ BillingService.php
â”‚   â””â”€â”€ AuditService.php
â”œâ”€â”€ Jobs/
â”‚   â”œâ”€â”€ ProcessOrderApproval.php
â”‚   â”œâ”€â”€ SendNotification.php
â”‚   â”œâ”€â”€ CalculateSla.php
â”‚   â””â”€â”€ ArchiveOldAuditLogs.php
â”œâ”€â”€ Events/
â”‚   â”œâ”€â”€ OrderSubmitted.php
â”‚   â”œâ”€â”€ OrderApproved.php
â”‚   â”œâ”€â”€ OrderReadyToShip.php
â”‚   â”œâ”€â”€ DeliveryNoteIssued.php
â”‚   â”œâ”€â”€ ProofUploaded.php
â”‚   â”œâ”€â”€ StockVerified.php
â”‚   â””â”€â”€ BillingStatusChanged.php
â”œâ”€â”€ Listeners/
â”‚   â””â”€â”€ (Listener untuk setiap Event di atas)
â”œâ”€â”€ Policies/
â”‚   â”œâ”€â”€ OrderPolicy.php
â”‚   â”œâ”€â”€ StockPolicy.php
â”‚   â”œâ”€â”€ CustomerPolicy.php
â”‚   â””â”€â”€ AuditLogPolicy.php
â””â”€â”€ Exports/
    â”œâ”€â”€ OrdersExport.php
    â”œâ”€â”€ StockReportExport.php
    â””â”€â”€ LostSalesExport.php
```

---

## 3. Konvensi Kode Bersama

### 3.1 Penamaan

| Elemen | Konvensi | Contoh |
|---|---|---|
| Model | PascalCase, singular | `SalesOrder`, `InventoryStock` |
| Controller | PascalCase + Controller | `OrderController` |
| Migration | snake_case, deskriptif | `create_sales_orders_table` |
| Blade View | kebab-case | `order-detail.blade.php` |
| Route Name | dot notation per modul | `sales.orders.create`, `warehouse.inbound.store` |
| CSS Class | kebab-case (BEM jika kompleks) | `order-card`, `status-badge--approved` |
| JS Function | camelCase | `submitOrder()`, `playNotificationSound()` |
| DB Table | snake_case, plural | `sales_orders`, `inventory_stocks` |
| DB Column | snake_case | `qty_available`, `created_at` |

### 3.2 Komentar & Dokumentasi
- Setiap **method publik** di Controller dan Service wajib memiliki PHPDoc
- Setiap **migration** wajib memiliki komentar di atas kolom yang tidak self-explanatory
- Setiap **Blade section** kompleks wajib memiliki komentar HTML `<!-- Section: ... -->`
- Setiap **JavaScript function** wajib memiliki JSDoc

### 3.3 Git Commit Message
```
Format: <type>(<scope>): <deskripsi singkat>

Contoh:
feat(inbound): add pallet split logic on inbound creation
fix(order): fix FIFO allocation when stock is partially allocated
ui(sales): redesign order tracking timeline on mobile
style(warehouse): adjust table spacing on inventory page
test(order): add unit test for partial fulfillment
chore(docker): update nginx config for file upload limit
```

---

## 4. Workflow Kolaborasi Antar Agent

### 4.1 Urutan Pengembangan Per Fitur

```mermaid
flowchart LR
    A["1. Backend Agent<br>(Claude Opus)"] --> B["2. Frontend Agent<br>(Gemini 2.5 Pro)"]
    B --> C["3. Backend Agent<br>(Integration Test)"]
    
    A -- "Membuat:<br>Migration, Model,<br>Controller, Route,<br>Service, Test" --> B
    B -- "Membuat:<br>Blade, CSS, JS,<br>Component UI" --> C
    C -- "Menjalankan:<br>Feature Test,<br>Browser Test" --> D[âœ… Merge]
```

**Aturan:** Backend Agent SELALU mengerjakan duluan sebelum Frontend Agent menyentuh fitur yang sama. Ini memastikan endpoint, data structure, dan validation sudah siap sebelum UI dibuat.

### 4.2 Kontrak Data (Data Contract)

Setiap kali Backend Agent membuat Controller method baru, wajib mendokumentasikan **kontrak data** yang akan dikirim ke view:

```php
/**
 * Menampilkan halaman daftar pesanan untuk Sales.
 *
 * @return \Illuminate\View\View
 *
 * DATA CONTRACT untuk Frontend:
 * - $orders: Collection<SalesOrder> (paginated, 15/page)
 *   -> id, order_number, customer.name, total_items, status, created_at
 * - $statusFilter: string|null (active filter)
 * - $stats: object {pending: int, approved: int, shipping: int, completed: int}
 */
public function index(Request $request): View
```

Frontend Agent membaca kontrak ini untuk mengetahui variabel apa saja yang tersedia di Blade.

### 4.3 Branching per Agent

```
main (production)
â”œâ”€â”€ develop (staging/integration)
â”‚   â”œâ”€â”€ backend/feature-inbound       # Claude Opus kerja di sini
â”‚   â”œâ”€â”€ backend/feature-order         
â”‚   â”œâ”€â”€ frontend/feature-inbound      # Gemini 2.5 Pro kerja di sini
â”‚   â”œâ”€â”€ frontend/feature-order        
â”‚   â”œâ”€â”€ backend/fix-fifo-allocation   
â”‚   â””â”€â”€ frontend/fix-mobile-layout    
```

---

## 5. Batasan Konteks yang Tidak Boleh Dilanggar

### 5.1 Batasan Bisnis (Berlaku untuk Kedua Agent)

| Aturan | Detail |
|---|---|
| Sales tidak boleh lihat stok | Sistem Semi-Blind: hanya indikator âœ…âš ï¸âŒ, bukan angka |
| Order cutoff jam 15:00 | Setelah jam 15:00 WIB, form order terkunci |
| Pembayaran 30/60/90 hari | Harus masuk menu billing, blokir order jika belum bayar |
| Cash/Transfer | Langsung complete tanpa billing |
| Palet otomatis | Qty dipecah otomatis berdasarkan kapasitas per UoM |
| FIFO wajib | Stok tertua HARUS keluar duluan |
| Maker-Checker inbound | Operator put-away â†’ Logistik verifikasi â†’ Stok aktif |
| Max 2 device per user | Session ketiga menendang session tertua |
| MFA wajib | Semua role wajib Google Authenticator |
| Edit stok | Hanya Manager dan Super Admin melalui menu Stock |

### 5.2 Batasan Teknis

| Aturan | Detail |
|---|---|
| Upload file | Hanya PNG/JPG, max 5MB, atau langsung dari kamera |
| No SPA | Tidak menggunakan React, Vue, atau framework SPA |
| No external API | Tidak ada integrasi ke sistem keuangan atau pihak ketiga |
| No barcode | Tidak menggunakan scanner barcode |
| No expiry tracking | Cat tidak memiliki tanggal kadaluarsa |
| No backorder | Sisa pesanan yang tidak terpenuhi = lost sales |
| No retur (fase 1) | Modul retur dikembangkan setelah sistem utama 80-90% |
| No transfer gudang | Setiap gudang independen, filter by dispatch code |

---

## 6. Checklist Sebelum Merge ke `develop`

### Backend Agent Checklist:
- [ ] Semua migration berjalan tanpa error (`php artisan migrate:fresh`)
- [ ] Semua model relationship didefinisikan dan di-test
- [ ] Semua route dilindungi middleware yang sesuai
- [ ] Form Request validation lengkap
- [ ] Service class memiliki unit test
- [ ] Controller memiliki feature test
- [ ] Tidak ada N+1 query (gunakan `->with()` eager loading)
- [ ] Audit log tercatat untuk operasi sensitif
- [ ] Event dan Listener terdaftar di `EventServiceProvider`
- [ ] Data contract terdokumentasi di PHPDoc Controller

### Frontend Agent Checklist:
- [ ] Semua halaman responsive sesuai target viewport
- [ ] Tidak ada inline style (semua di CSS file)
- [ ] Notifikasi suara berfungsi
- [ ] Form validation client-side sebelum submit
- [ ] Loading state ditampilkan saat AJAX call
- [ ] Modal konfirmasi untuk aksi destruktif (delete, approve, reject)
- [ ] Semua tabel menggunakan DataTables dengan pagination
- [ ] Dashboard chart menggunakan data dari endpoint yang sudah ada
- [ ] Accessibility: semua input memiliki label, semua button memiliki aria-label
- [ ] Cross-browser test: Chrome, Firefox, Safari mobile

---

## 7. Referensi Dokumen Terkait

| Dokumen | Deskripsi |
|---|---|
| `1_prd.md` | Product Requirements Document lengkap |
| `2_database_design.md` | Skema database, relasi, dan penjelasan kolom |
| `3_system_architecture.md` | Arsitektur teknis, infrastruktur, dan keamanan |
| `4_ui_ux_guidelines.md` | Panduan desain UI/UX per portal |
| `5_testing_strategy.md` | Strategi dan skenario testing |
| `6_cicd_docker_setup.md` | Setup Docker, GitHub Actions, dan branching |

> [!IMPORTANT]
> Setiap AI Agent **WAJIB membaca dokumen referensi yang relevan** sebelum memulai pengembangan fitur apapun. Jangan berasumsi â€” selalu cek dokumen.
