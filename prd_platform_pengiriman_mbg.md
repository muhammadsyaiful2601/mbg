# Product Requirement Document (PRD)
## Platform Informasi Menu & Pelacakan Pengantaran Makan Bergizi Gratis (MBG)

---

| Informasi Dokumen | Detail |
| :--- | :--- |
| **Nama Produk** | Platform Tracking & Manajemen Distribusi MBG |
| **Versi** | 1.0.0 |
| **Status** | Draft Konseptual |
| **Stack Teknologi** | Laravel (Backend API), React.js (Frontend), MySQL (Database) |
| **Tema Desain Visual** | Nuansa Biru-Putih Modern & Clean (*MBG Theme*) |

---

## 1. Ringkasan Eksekutif & Tujuan

### 1.1 Visi Produk
Membangun platform sistem informasi dan pemantauan pengiriman Program Makan Bergizi Gratis (MBG) secara *real-time* yang transparan, akurat, dan terintegrasi antara Dapur Satuan Pelayanan (SP), Kurir Pengantar, dan Pihak Sekolah.

### 1.2 Tujuan Utama
1. **Transparansi Menu:** Menyediakan jadwal menu harian lengkap dengan rincian gizi dan kalori yang dapat diakses oleh pihak sekolah dan orang tua.
2. **Notifikasi Otomatis:** Mengurangi keterlambatan dan hambatan komunikasi melalui notifikasi otomatis seketika saat dapur menekan tombol **"Siap Diantar"**.
3. **Pelacakan Real-time (Shopee-style):** Memberikan visibilitas status pengiriman makanan dari dapur hingga titik penyerahan di sekolah menggunakan peta *live-tracking*.
4. **Verifikasi Penerimaan:** Memastikan kuantitas dan kualitas makanan yang diterima sekolah terdokumentasi dengan baik melalui foto bukti penerimaan.

---

## 2. Arsitektur & Tech Stack

* **Backend Framework:** Laravel 11+ (RESTful API, WebSockets via Laravel Reverb / Pusher Integration, Queue Worker untuk Notification).
* **Frontend Framework:** React.js (Vite, Tailwind CSS, Lucide React Icons, React Query, Leaflet.js / Mapbox React).
* **Database:** MySQL 8.0+.
* **Real-time Engine:** Laravel Reverb / Pusher WebSocket (untuk update koordinat GPS & status pengiriman).
* **Notification Engine:** Firebase Cloud Messaging (FCM) & WhatsApp API Gateway (Fonnte/Wablas).

---

## 3. Sistem Peran & Hak Akses (User Roles)

1. **Admin Pusat / Dapur SP (Kitchen Admin):**
   * Mengelola input menu harian dan rincian gizi.
   * Menentukan alokasi sekolah tujuan dan porsi.
   * Mengubah status pengiriman (**Diproses** $\rightarrow$ **Siap Diantar**).
   * Memantau seluruh armada pengantar di *dashboard*.

2. **Driver / Kurir Pengantar:**
   * Melihat rute pengantaran harian.
   * Mengirimkan data lokasi GPS secara *real-time* saat pengantaran berlangsung.
   * Menyerahkan makanan dan melakukan verifikasi penyerahan.

3. **Sekolah (Guru Piket / Panitia MBG):**
   * Menerima notifikasi *real-time* saat makanan siap dikirim.
   * Memantau lokasi kurir di peta.
   * Melakukan konfirmasi penerimaan porsi + upload foto bukti penerimaan.
   * Memberikan *feedback* / ulasan makanan.

4. **Publik / Orang Tua Siswa (Read-Only):**
   * Melihat kalender dan katalog menu harian/mingguan beserta rincian nutrisi.

---

## 4. Alur Kerja Pengantaran & Notifikasi (Core User Flow)

```text
[ Dapur SP ]                 [ Server Backend (Laravel) ]               [ Pihak Sekolah ]
     │                                    │                                     │
     ├─► Menyiapkan Makanan               │                                     │
     │   & Kemasan                        │                                     │
     │                                    │                                     │
     ├─► Klik "Siap Diantar" ────────────►│                                     │
     │                                    ├─► Kirim WebPush / WA API ──────────►│ (Terima Notifikasi:
     │                                    │                                     │  "Makanan Sedang Diantar!")
     │                                    ├─► Buka Channel WebSocket ──────────►│ (Peta Tracking Aktif)
     │                                    │                                     │
[ Kurir / Driver ]                        │                                     │
     │                                    │                                     │
     ├─► Mulai Perjalanan (GPS Active) ──►│                                     │
     │   (Update koordinat tiap 15dtk)    ├─► Broadcast Koordinat GPS ─────────►│ (Lihat Icon Kurir
     │                                    │                                     │  Bergerak di Peta)
     │                                    │                                     │
     ├─► Tiba di Sekolah ───────────────►│                                     │
     │                                    ├─► Status: "Tiba di Lokasi" ────────►│
     │                                    │                                     │
     └────────────────────────────────────┼────────────────────────────────────►├─► Serah Terima
                                          │                                     ├─► Klik "Diterima"
                                          │◄────────────────────────────────────┼─► Upload Foto & Rating
                                          │                                     │
                                   [Status: Selesai]
```

---

## 5. Fitur Utama & Kebutuhan Fungsional

### 5.1 Modul Informasi & Katalog Menu (React Frontend)
* **Tampilan Kalender Menu:** Kalender interaktif mingguan/bulanan.
* **Informasi Nutrisi:** Menampilkan Kalori (kcal), Protein (g), Karbohidrat (g), Lemak (g), serta daftar bahan alergen (Kacang, Susu, Telur, dll).
* **Filter Wilayah/Sekolah:** Memilih daftar menu berdasarkan jenjang (SD, SMP, SMA) atau lokasi sekolah.

### 5.2 Modul Pengantaran & Live Tracking (Shopee Style)
* **Status Pengiriman 4-Tahap:**
  1. 🍳 `Diproses Dapur` (Makanan sedang dikemas)
  2. 🚚 `Dalam Pengantaran` (Kurir meluncur, peta *live tracking* aktif)
  3. 🏫 `Tiba di Sekolah` (Kurir telah sampai di area sekolah)
  4. ✅ `Selesai Ditinjau` (Sekolah selesai memverifikasi jumlah & kondisi)
* **Peta Real-time (Leaflet.js + WebSockets):**
  * Menampilkan posisi kurir (*marker* bergerak) menuju koordinat sekolah.
  * Estimasi Waktu Kedatangan (ETA) berbasis jarak.

### 5.3 Modul Notifikasi Otomatis
* **Mekanisme Pemicu (Trigger):**
  * Ketika admin dapur menekan tombol **"Siap Diantar"**, Laravel Queue memicu:
    1. **Web Push Notification:** Muncul di *browser* HP/Laptop guru piket sekolah.
    2. **WhatsApp Gateway Notification:** Mengirimkan pesan WhatsApp otomatis ke nomor WA sekolah berisi link unik *live tracking*.
    3. **Sound Alert:** Efek suara notifikasi pada *dashboard* sekolah yang sedang terbuka.

### 5.4 Modul Verifikasi & Feedback Sekolah
* **Tombol Konfirmasi Penerimaan:** Form input jumlah porsi yang diterima vs porsi yang direncanakan.
* **Upload Bukti Foto:** Mengunggah foto serah terima makanan/kondisi makanan langsung via kamera HP.
* **Ulasan & Rating (1-5 Bintang):** Sekolah dapat memberikan umpan balik mengenai ketepatan waktu, kebersihan, rasa, dan porsi.

---

## 6. Desain Antarmuka (UI/UX) & Skema Warna MBG

Tema visual dirancang bersih, profesional, dan tepercaya menggunakan warna dominan **Biru** dan **Putih** yang mencerminkan identitas resmi program MBG.

### 6.1 Palet Warna (Tailwind CSS Palette)
* **Primary Blue (Warna Utama):** `#1E40AF` (`bg-blue-800`) - Header, Tombol Utama, Navbar.
* **Secondary Light Blue (Aksen):** `#3B82F6` (`bg-blue-500`) - Status tag, Icon aktif, Highlight peta.
* **Background Light (Latar Belakang):** `#F8FAFC` (`bg-slate-50`) - Area konten utama.
* **Card & Container:** `#FFFFFF` (`bg-white`) dengan border tipis `#E2E8F0`.
* **Accent Success (Status Selesai):** `#10B981` (`bg-emerald-500`).
* **Text Dark:** `#0F172A` (`text-slate-900`) untuk kejelasan keterbacaan (*readability*).

### 6.2 Konsep Layout Website
* **Top Navigation Bar:** Logo MBG, Nama Sekolah/Dapur, Notifikasi Bell Icon dengan Badge Merah jika ada pengiriman aktif.
* **Dashboard Sekolah (Dua Kolom):**
  * **Kolom Kiri (Status & Detail):** Lini masa status pengantaran ala Shopee (*Timeline Status*) + detail menu hari ini.
  * **Kolom Kanan (Peta Interactive):** Peta luas yang menampilkan rute pengantar dari Dapur SP ke sekolah terkait.

---

## 7. Skema Basis Data (MySQL Schema)

Berikut adalah struktur tabel utama MySQL yang akan digunakan:

```sql
-- Tabel Users
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone_number VARCHAR(20) NOT NULL,
    role ENUM('admin_kitchen', 'driver', 'school', 'public') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabel Sekolah
CREATE TABLE schools (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT,
    school_name VARCHAR(255) NOT NULL,
    address TEXT NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    target_portion INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Tabel Menu Harian
CREATE TABLE menus (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    date DATE NOT NULL,
    menu_name VARCHAR(255) NOT NULL,
    description TEXT,
    calories INT,
    protein_grams DECIMAL(5,2),
    carbs_grams DECIMAL(5,2),
    fat_grams DECIMAL(5,2),
    allergens VARCHAR(255),
    photo_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabel Pengiriman (Deliveries)
CREATE TABLE deliveries (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    delivery_code VARCHAR(50) UNIQUE NOT NULL,
    menu_id BIGINT NOT NULL,
    school_id BIGINT NOT NULL,
    driver_id BIGINT,
    portion_planned INT NOT NULL,
    portion_received INT NULL,
    status ENUM('processing', 'ready_for_delivery', 'on_delivery', 'arrived', 'completed') DEFAULT 'processing',
    ready_at TIMESTAMP NULL,
    arrived_at TIMESTAMP NULL,
    completed_at TIMESTAMP NULL,
    proof_photo_url VARCHAR(255) NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (menu_id) REFERENCES menus(id),
    FOREIGN KEY (school_id) REFERENCES schools(id),
    FOREIGN KEY (driver_id) REFERENCES users(id)
);

-- Tabel Tracking GPS Driver Real-time
CREATE TABLE driver_locations (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    delivery_id BIGINT NOT NULL,
    driver_id BIGINT NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (delivery_id) REFERENCES deliveries(id) ON DELETE CASCADE
);
```

---

## 8. Persyaratan Non-Fungsional (Non-Functional Requirements)

1. **Performansi Real-time:** Latensi update lokasi GPS kurir ke peta penerima di bawah **2 detik**.
2. **Ketersediaan Sistem (Availability):** Sistem beroperasi minimal **99.5%** pada jam pengantaran aktif (06.00 WIB - 13.00 WIB).
3. **Responsivitas UI:** Desain *Mobile-First*, responsif diakses dari HP Android guru piket maupun monitor komputer sekolah.
4. **Keamanan:** Autentikasi berbasis **Laravel Sanitized/Passport Token**, koneksi HTTPS wajib, serta sanitasi file upload foto bukti penerimaan.