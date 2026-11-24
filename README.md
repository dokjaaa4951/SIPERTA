# ⛽ SIPERTA

**Sistem Informasi Ketersediaan Pertalite** — aplikasi mobile web untuk melihat kondisi ketersediaan Pertalite di SPBU, dengan data dari laporan warga dan pembaruan petugas SPBU.

Proyek mata kuliah **Pemrograman Berbasis Objek**, dibangun dengan **Laravel** dan **REST API**.

## Fitur

- Daftar dan peta SPBU dengan status Pertalite: **tersedia**, **menipis**, atau **habis**
- Pencarian nama SPBU dan filter berdasarkan status
- Laporan warga terbaru beserta waktunya pada setiap SPBU
- Tombol petunjuk arah ke SPBU melalui Google Maps
- Login berbasis token (Laravel Sanctum) untuk mengirim laporan
- Dua peran pengguna: **warga** (laporan tercatat sebagai masukan) dan **petugas SPBU** (laporan langsung memperbarui status resmi)
- Antarmuka mobile-first dengan mode gelap otomatis

## Teknologi

Laravel (PHP 8.2+), Laravel Sanctum, SQLite (dapat diganti MySQL), HTML, CSS, dan JavaScript murni, Leaflet.js dengan peta OpenStreetMap.

## Arsitektur

```
[HP / Peramban]  --HTTP/JSON-->  [Laravel API + Sanctum]  -->  [Database]
 public/siperta/index.html        Controller → Model / Service
```

## Struktur Berkas Utama

```
siperta/
├── app/
│   ├── Http/Controllers/Api/   AuthController, SpbuController, LaporanController
│   ├── Models/                 User, Spbu, Laporan
│   └── Services/Pelapor/       Pelapor, WargaPelapor, PetugasPelapor
├── database/
│   ├── migrations/             tabel spbu, kolom tambahan users, tabel laporan
│   ├── seeders/                DatabaseSeeder (membaca CSV)
│   └── data/spbu.csv           data SPBU
├── routes/api.php              definisi endpoint API
└── public/siperta/index.html   antarmuka mobile web
```

## Instalasi

Prasyarat: **PHP 8.2+** (ekstensi `zip`, `fileinfo`, `pdo_sqlite` aktif) dan **Composer**.

```bash
# 1. Unduh proyek
git clone https://github.com/dokjaaa4951/SIPERTA.git
cd SIPERTA

# 2. Pasang dependensi
composer install

# 3. Siapkan konfigurasi
cp .env.example .env          # di Windows CMD: copy .env.example .env
php artisan key:generate

# 4. Buat tabel dan isi data awal
php artisan migrate:fresh --seed

# 5. Jalankan server
php artisan serve --host=0.0.0.0
```

Jika diminta membuat file `database/database.sqlite` saat migrasi, jawab **yes**. Untuk memakai MySQL, ubah pengaturan `DB_*` di file `.env`.

Buka aplikasi di `http://localhost:8000/siperta/index.html`.

### Membuka dari ponsel

Hubungkan ponsel dan komputer ke Wi-Fi yang sama, cek alamat IPv4 komputer dengan `ipconfig` (Windows) atau `ifconfig` (Linux/macOS), lalu buka `http://IP-KOMPUTER:8000/siperta/index.html`. Jika tidak terbuka, izinkan `php.exe` pada firewall untuk jaringan Private. Fitur lokasi otomatis umumnya hanya aktif pada HTTPS.

## Akun Contoh

| Peran | Email | Password |
|---|---|---|
| Warga | `warga@siperta.test` | `warga123` |
| Petugas (SPBU pertama di CSV) | `petugas@siperta.test` | `petugas123` |

Akun ini hanya untuk pengembangan. Ganti atau hapus sebelum aplikasi dipublikasikan.

## Mengubah Data SPBU

Data SPBU dibaca dari `database/data/spbu.csv` dengan format kolom:

```csv
nama,alamat,lat,lng,status
SPBU Contoh 1,"Jl. Contoh No. 1, Kota",-6.2000,106.8166,tersedia
```

Koordinat memakai titik sebagai pemisah desimal, dan `status` boleh kosong (bawaan `tersedia`). Setelah mengedit, jalankan `php artisan migrate:fresh --seed`. Perintah ini menghapus seluruh laporan yang sudah masuk. Untuk mengubah satu data tanpa menghapus laporan, gunakan `php artisan tinker`.

## Dokumentasi API

| Metode | Endpoint | Akses | Fungsi |
|---|---|---|---|
| POST | `/api/register` | Publik | Mendaftar sebagai warga, mengembalikan token |
| POST | `/api/login` | Publik | Masuk, mengembalikan token |
| POST | `/api/logout` | Token | Mencabut token saat ini |
| GET | `/api/spbu` | Publik | Daftar SPBU beserta laporan warga terbaru |
| GET | `/api/spbu/{id}` | Publik | Detail satu SPBU |
| POST | `/api/laporan` | Token | Mengirim laporan (`spbu_id`, `status`) |

Endpoint bertanda Token memerlukan header `Authorization: Bearer TOKEN`. Nilai `status` yang valid: `tersedia`, `menipis`, `habis`. Petugas yang melapor untuk SPBU selain miliknya menerima respons `403`.

Contoh penggunaan:

```bash
# Login
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -d '{"email":"petugas@siperta.test","password":"petugas123"}'

# Kirim laporan (ganti TOKEN)
curl -X POST http://localhost:8000/api/laporan \
  -H "Content-Type: application/json" -H "Accept: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"spbu_id":1,"status":"habis"}'
```

## Konsep PBO

**Enkapsulasi:** model Eloquent mengatur atribut lewat `$fillable`, `$hidden`, dan `casts`. **Pewarisan:** semua model mewarisi kelas dasar Laravel (`Model`, `Authenticatable`). **Abstraksi:** antarmuka `Pelapor` mendefinisikan kontrak `laporkan()`. **Polimorfisme:** `WargaPelapor` hanya menyimpan laporan, sedangkan `PetugasPelapor` memeriksa kewenangan, menyimpan laporan, lalu memperbarui status resmi SPBU; `User::pelapor()` memilih implementasi sesuai peran.

## Keterbatasan

Data bergantung pada laporan warga dan belum ada verifikasi otomatis. Data SPBU tidak terhubung dengan sistem resmi Pertamina.

## Pengembang

Kelompok 1 ,
Anggota :
Agil Miswary, 
Abdullah Nur, 
Muh Ilham Ismail

Mata kuliah Pemrograman Berbasis Objek, Teknik Informatika.

## Lisensi

Proyek ini dibuat untuk keperluan pembelajaran. 
