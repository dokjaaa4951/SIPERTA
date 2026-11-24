SIPERTA
Sistem Informasi Ketersediaan Pertalite

1. Latar Belakang
Pertalite merupakan BBM bersubsidi yang banyak digunakan masyarakat, namun ketersediaannya di SPBU tidak selalu terjamin. Pengendara sering kali baru mengetahui stok habis setelah tiba di lokasi, sehingga membuang waktu dan bahan bakar. SIPERTA dirancang sebagai aplikasi mobile web sederhana yang memungkinkan siapa pun melihat kondisi ketersediaan Pertalite di SPBU sebelum berangkat, dengan data yang berasal dari laporan warga dan pembaruan resmi petugas SPBU.
2. Tujuan
Tujuan proyek ini adalah menyediakan informasi ketersediaan Pertalite yang mudah diakses lewat peramban di ponsel, memungkinkan warga dan petugas SPBU memperbarui status stok secara cepat, serta menerapkan konsep pemrograman berbasis objek (enkapsulasi, pewarisan, abstraksi, dan polimorfisme) pada aplikasi yang nyata dengan arsitektur Laravel dan REST API.
3. Ruang Lingkup dan Fitur
Aplikasi menampilkan daftar dan peta SPBU dengan status Pertalite berupa tersedia, menipis, atau habis. Pengguna dapat mencari SPBU berdasarkan nama, menyaring berdasarkan status, melihat laporan warga terbaru beserta waktunya, dan membuka petunjuk arah ke SPBU lewat Google Maps. Pengguna yang sudah masuk dapat mengirim laporan status. Pembacaan data SPBU bersifat publik (tanpa login), sedangkan pelaporan memerlukan akun.
Di luar ruang lingkup versi ini: integrasi dengan data resmi Pertamina, verifikasi otomatis kebenaran laporan, notifikasi, dan halaman admin untuk mengelola data lewat antarmuka.
4. Pengguna dan Peran
Ada dua peran. Warga dapat melapor, namun laporannya hanya dicatat sebagai masukan dan tidak mengubah status resmi SPBU. Petugas SPBU terhubung ke satu SPBU tertentu; laporannya langsung memperbarui status resmi SPBU tersebut, dan ia tidak boleh melapor untuk SPBU lain. Pembedaan ini menjaga keandalan data karena status resmi hanya berubah oleh pihak yang berwenang.
5. Arsitektur Sistem
Sistem memakai arsitektur klien-server yang terpisah lewat API. Backend dibangun dengan Laravel dan menyediakan REST API berformat JSON. Autentikasi memakai Laravel Sanctum berbasis token (dikirim lewat header Authorization: Bearer TOKEN). Database bawaan adalah SQLite (dapat diganti MySQL lewat berkas .env). Frontend berupa satu halaman HTML, CSS, dan JavaScript murni yang dioptimalkan untuk layar ponsel, memanggil API menggunakan fetch, serta menampilkan peta dengan Leaflet dan data peta OpenStreetMap. Token login disimpan di localStorage peramban.
[HP / Peramban]  --HTTP/JSON-->  [Laravel API + Sanctum]  -->  [Database]
 index.html (Leaflet)             Controller → Model / Service
6. Rancangan Basis Data
Terdapat tiga tabel utama.
Tabel spbu menyimpan data SPBU: id, nama, alamat, lat, lng, status (tersedia/menipis/habis, bawaan tersedia), serta created_at dan updated_at.
Tabel users adalah tabel pengguna bawaan Laravel yang ditambah dua kolom: peran (warga atau petugas, bawaan warga) dan spbu_id (diisi hanya untuk petugas).
Tabel laporan mencatat setiap laporan: id, spbu_id, user_id, status, sumber (warga atau petugas), dan stempel waktu. Relasinya: satu SPBU memiliki banyak laporan, dan satu pengguna dapat membuat banyak laporan.
Data awal SPBU dimasukkan lewat seeder yang membaca berkas database/data/spbu.csv (kolom: nama, alamat, lat, lng, status).
7. Rancangan Kelas dan Konsep PBO
Model Eloquent Spbu, Laporan, dan User membungkus data dan relasinya. Enkapsulasi tampak pada atribut yang diatur lewat $fillable, $hidden, dan casts (misalnya kata sandi otomatis di-hash). Pewarisan terjadi karena seluruh model mewarisi kelas dasar Laravel (Model dan Authenticatable).
Abstraksi diwujudkan oleh antarmuka Pelapor dengan satu kontrak, yaitu metode laporkan(User, Spbu, status). Polimorfisme diwujudkan oleh dua implementasinya: WargaPelapor hanya menyimpan laporan, sedangkan PetugasPelapor memeriksa kewenangan SPBU, menyimpan laporan, lalu memperbarui status resmi SPBU. Metode User::pelapor() memilih implementasi yang sesuai dengan peran pengguna, sehingga controller cukup memanggil $user->pelapor()->laporkan(...) tanpa perlu mengetahui perannya.
8. Rancangan API
Metode
Endpoint
Akses
Fungsi
POST
/api/register
Publik
Mendaftar sebagai warga, mengembalikan token
POST
/api/login
Publik
Masuk, mengembalikan token
POST
/api/logout
Token
Mencabut token saat ini
GET
/api/spbu
Publik
Daftar SPBU beserta laporan warga terbaru
GET
/api/spbu/{id}
Publik
Detail satu SPBU
POST
/api/laporan
Token
Kirim laporan (spbu_id, status)

Validasi dilakukan di sisi server: status harus salah satu dari tersedia, menipis, atau habis, dan spbu_id harus ada di tabel SPBU. Jika petugas melapor untuk SPBU yang bukan miliknya, server menolak dengan kode 403. Kegagalan login mengembalikan 401.
9. Alur Kerja
Pengguna membuka halaman, aplikasi memanggil GET /api/spbu, lalu menampilkan daftar dan titik pada peta (warna hijau, kuning, atau merah sesuai status). Untuk melapor, pengguna menekan tombol Lapor, masuk bila belum, memilih SPBU dan status, lalu aplikasi mengirim POST /api/laporan dengan token. Server memilih WargaPelapor atau PetugasPelapor sesuai peran, menyimpan laporan, dan (khusus petugas) memperbarui status resmi. Daftar kemudian dimuat ulang sehingga perubahan langsung terlihat.
10. Rancangan Antarmuka
Antarmuka dirancang mobile-first dengan palet hijau, sudut membulat, dan jarak lega. Komponen utamanya adalah header dengan tombol masuk, peta, kolom pencarian, tombol filter status, kartu SPBU (ikon dan lencana status, laporan warga terbaru, tombol "Lihat di peta" dan "Petunjuk arah"), serta tombol melayang + Lapor yang membuka panel dari bawah untuk masuk dan melapor. Terdapat animasi halus, tampilan skeleton saat memuat, notifikasi singkat setelah aksi, dan mode gelap otomatis mengikuti pengaturan perangkat.
11. Struktur Berkas Utama
siperta/
├── app/
│   ├── Http/Controllers/Api/   AuthController, SpbuController, LaporanController
│   ├── Models/                 User, Spbu, Laporan
│   └── Services/Pelapor/       Pelapor, WargaPelapor, PetugasPelapor
├── database/
│   ├── migrations/             tabel spbu, tambahan kolom users, tabel laporan
│   ├── seeders/                DatabaseSeeder (membaca CSV)
│   └── data/spbu.csv           data SPBU
├── routes/api.php              definisi endpoint
└── public/siperta/index.html   antarmuka mobile web
12. Implementasi dan Pengujian
Pemasangan: buat proyek dengan composer create-project laravel/laravel siperta, jalankan php artisan install:api untuk memasang Sanctum, salin berkas proyek, lalu php artisan migrate:fresh --seed. Server dijalankan dengan php artisan serve --host=0.0.0.0, dan aplikasi dibuka di http://localhost:8000/siperta/index.html atau lewat IP laptop dari ponsel pada jaringan Wi-Fi yang sama.
Skenario uji: (1) GET /api/spbu menampilkan seluruh SPBU; (2) login sebagai warga lalu melapor, laporan muncul sebagai "laporan warga terbaru" tanpa mengubah status resmi; (3) login sebagai petugas lalu melapor untuk SPBU miliknya, status resmi berubah; (4) petugas melapor untuk SPBU lain ditolak dengan 403; (5) tampilan diuji di peramban ponsel. Akun uji dari seeder: warga@siperta.test dan petugas@siperta.test.
13. Keterbatasan dan Pengembangan Lanjutan
Data bergantung pada kejujuran laporan warga dan belum ada verifikasi otomatis, sehingga ke depan dapat ditambahkan sistem skor kepercayaan pelapor atau pembatasan jumlah laporan. Lokasi otomatis di ponsel umumnya baru berfungsi pada HTTPS, jadi pada tahap uji lokal peta tidak otomatis berpindah ke posisi pengguna. Pengembangan berikutnya meliputi halaman admin untuk mengelola SPBU, pengurutan SPBU berdasarkan jarak terdekat, notifikasi saat stok berubah, penerapan sebagai PWA, serta hosting dengan HTTPS agar dapat diakses publik.
