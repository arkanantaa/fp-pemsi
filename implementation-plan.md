
# Rencana Implementasi: Simulasi Pengaruh Tim & Penyebaran Ideologi

## 1. Ringkasan Proyek
Dokumen ini menguraikan rencana teknis untuk membuat model simulasi berbasis agen menggunakan NetLogo. Simulasi ini bertujuan untuk memodelkan dinamika penyebaran pengaruh sosial antar tim yang bersaing. Model ini akan mencakup atribut individu agen (**kepercayaan diri, manipulasi**), peran kepemimpinan, pengaruh lingkungan (**cuaca**), dan mekanisme unik di mana agen dapat secara spontan membentuk tim baru dari perpaduan ideologi yang sudah ada.

Tujuan utamanya adalah untuk mengamati bagaimana struktur sosial (tim), pengaruh lingkungan, dan inovasi individu (penciptaan tim baru) secara kolektif membentuk lanskap ideologis suatu populasi.

---

## 2. Komponen Antarmuka (Widgets)

Antarmuka pengguna akan menyediakan kontrol untuk mengatur parameter awal dan memvisualisasikan hasil simulasi secara real-time.

### Sliders (Penggeser):
- **number-of-people**: Mengatur jumlah total agen dalam simulasi.  
	_Rentang: 3 - 500, Default: 100_
- **initial-teams**: Menentukan jumlah tim awal yang akan dibuat.  
	_Rentang: 2 - 5, Default: 3_
- **confidence-level-probability**: Mengatur probabilitas (dalam persen) bagi seorang untuk ikut atau meninggalkan suatu tim (mempertahankan pendiriannya terhadap satu tim).
- **manipulative-level-probability**: Mengatur probabilitas (dalam persen) bagi seorang untuk mempengaruhi orang lain untuk bergabung dengan timnya.
- **weather**: Mengatur kondisi lingkungan yang mempengaruhi interaksi.  
	_Pilihan: 0 (Cerah), 1 (Panas), 2 (Hujan)_
	- Jika 0: Maka confidence dan ...
	- Jika 1: ...
	- Jika 2: ...

### Buttons (Tombol):
- **setup**: Untuk menginisialisasi atau mengatur ulang dunia simulasi sesuai dengan nilai slider yang ditetapkan.
- **go**: Untuk memulai dan menjalankan simulasi secara terus-menerus.

### Plots & Monitors (Grafik & Monitor):
- **Grafik Populasi Tim**: Sebuah grafik garis yang menampilkan jumlah anggota dari setiap tim dari waktu ke waktu. Grafik ini harus dinamis untuk dapat menambahkan garis baru saat tim baru terbentuk.
- **Monitor Jumlah Tim**: Sebuah monitor sederhana yang menampilkan jumlah total tim yang aktif saat ini.
- **Monitor Jumlah Orang dalam Setiap Tim**: Sebuah monitor untuk menampilkan jumlah orang yang terdapat pada setiap tim.

---

## 3. Komponen Model (Agen & Variabel)

Model ini akan terdiri dari agen, variabel global, dan lingkungan.

### Tipe Agen (Breeds):
- **leaders**: Agen dengan peran khusus untuk memperkuat timnya.
- **followers**: Populasi umum yang dapat dipengaruhi.

### Variabel Agen (Berlaku untuk kedua tipe):
- **team-color**: Warna yang merepresentasikan afiliasi tim agen.
- **confidence**: Nilai numerik (0-100) yang mewakili kekuatan keyakinan agen pada timnya.
- **manipulation**: Nilai numerik (1-10) yang mewakili kemampuan agen untuk memengaruhi orang lain.
- **interaction-radius**: Jarak di mana seorang agen dapat berinteraksi dengan agen lain.

### Variabel Global:
- **team-list**: Sebuah list yang menyimpan data warna dari semua tim yang aktif. Penting untuk plotting dan logika.
- **environmental-modifier**: Sebuah pengali (misalnya, 0.5 hingga 1.0) yang ditentukan oleh weather. Nilai ini akan digunakan untuk mengurangi efektivitas interaksi selama cuaca buruk.

---

## 4. Logika Simulasi (Prosedur Utama)

Alur simulasi akan dikelola oleh prosedur utama `setup` dan `go`.

### Prosedur `setup`
- **Bersihkan Dunia**: Jalankan `clear-all` untuk mereset simulasi.
- **Inisialisasi Populasi**: Buat agen sejumlah `number-of-people` dengan tipe followers.
- **Inisialisasi Tim & Angkat Pemimpin Awal**:
	- Buat daftar warna dasar (misalnya, merah, biru, hijau) berdasarkan `initial-teams`.
	- Pilih `initial-teams` orang dengan nilai confidence tertinggi dari populasi followers untuk menjadi leaders.
	- Tetapkan `team-color` yang berbeda untuk setiap leader yang baru diangkat.
	- Pada awalnya, semua followers tidak memiliki `team-color` kecuali leaders yang baru diangkat.
	- Berikan nilai acak untuk speed, confidence (dengan mempertimbangkan nilai minimum untuk menjadi leader), dan manipulation kepada semua agen.
	- Sebarkan mereka secara acak di dunia.

### Prosedur `go`
- **Perbarui Lingkungan**: Atur `environmental-modifier` berdasarkan slider weather.
- **Gerak**: Semua agen bergerak sesuai dengan speed mereka.
- **Aksi Pemimpin**:
	- Semua leaders menjalankan prosedur `boost-confidence`.
	- Jika nilai confidence seorang leader <= 50, maka leader tersebut dan semua followers yang berafiliasi dengan timnya akan dibuat tidak bergabung dengan tim manapun (kehilangan `team-color` mereka).
- **Aksi Follower**: Semua followers menjalankan prosedur `evaluate-surroundings` dan `attempt-to-create-new-team`.
- **Perbarui Visual**: Perbarui grafik dan tampilan agen.
- **Lanjutkan Waktu**: Jalankan `tick` untuk maju ke langkah waktu berikutnya.

---

## 5. Mekanisme Inti (Prosedur Pembantu)

Fungsi-fungsi ini berisi logika utama yang membuat simulasi ini unik.

### `boost-confidence` (Hanya untuk leaders)
- Pemimpin akan mengidentifikasi semua follower dari tim yang sama dalam `leader-boost-radius`.
- Kepercayaan (confidence) dari para follower tersebut akan ditingkatkan sedikit.
- **Pembaruan Manipulasi**: Jika pemimpin berhasil memanipulasi (misalnya, meningkatkan confidence pengikutnya), maka confidence pemimpin akan meningkat dan manipulation pemimpin juga akan meningkat.
- **Pembaruan Manipulasi Gagal**: Jika pemimpin tidak berhasil memanipulasi (misalnya, tidak ada pengikut yang terpengaruh atau confidence mereka tidak meningkat), maka confidence pemimpin akan menurun dan manipulation pemimpin tidak berubah (bertambah 0).

### `evaluate-surroundings` (Hanya untuk followers)
- Ini adalah mekanisme "konversi" atau perpindahan tim.
- Seorang agen akan menghitung "Skor Pengaruh" dari setiap tim yang ada di dalam `interaction-radius`-nya.
- **Rumus Skor Pengaruh (per tim)**: Jumlah dari `[confidence * manipulation]` dari semua agen tim tersebut di sekitar.
- Agen akan membandingkan skor timnya dengan skor tertinggi dari tim lawan.
- Jika skor lawan lebih tinggi, ada kemungkinan agen tersebut pindah tim.
- **Rumus Peluang Pindah Tim**: `((SkorLawan - SkorTimSaya) / SkorLawan) * environmental-modifier`
- Jika hasil undian acak memenuhi syarat, agen akan mengubah `team-color`-nya.

### `attempt-to-create-new-team` (Hanya untuk followers)
- Ini adalah fitur inovasi yang langka.
- **Syarat**: Agen harus berada di area yang beragam secara ideologis (misalnya, ada minimal dua tim berbeda di sekitarnya).
- **Pemicu**: Jika undian acak lebih kecil dari `new-team-chance`.
- **Proses**:
	- Agen berubah menjadi leader.
	- Ia menciptakan `team-color` baru dengan mencampurkan warnanya dengan warna salah satu tetangganya dari tim lain (menggunakan `scale-color` di NetLogo).
	- Warna baru ini ditambahkan ke `team-list` global.
	- Pemimpin baru ini mendapatkan peningkatan atribut seperti pemimpin awal.

---

## Update Baru

- Dari awal terdapat n leaders dengan warna yang berbeda. Leader tersebut dipilih berdasarkan n orang tertinggi menjadi leader. Pada awalnya, semua orang tidak memiliki team kecuali n orang tersebut. 
- Jika nilai confidence leader <= 50, maka leader dan anggota tim itu dibuat tidak bergabung tim manapun.
- Confidence +, Manipulasi + jika leader berhasil memanipulasi
- Confidence -, Manipulasi +0 jika leader tidak berhasil memanipulasi
- Set berapa nilai confidence minimum untuk jadi leader
- Mekanisme Pembentukan Tim Baru






