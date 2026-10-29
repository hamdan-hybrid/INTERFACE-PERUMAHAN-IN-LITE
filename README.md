# HYRES Interface Akses Hunian Pintu Masuk (ESP32)

Firmware siap upload untuk **ESP32 DevKit (ESP32-WROOM-32)** sebagai interface kontrol akses
hunian pintu masuk (**HYRES** — Smart Access for Better Living). Membaca kartu penghuni lewat
reader Wiegand, memutuskan akses sendiri dari database kartu di **microSD** (terdaftar, aktif,
sudah mulai berlaku, belum kedaluwarsa), lalu membuka pintu lewat relay. Suara lewat modul
Serial MP3 Player v1.1 (YX5300) + speaker. Semua tempelan kartu tercatat di log transaksi.
Pengaturan, data kartu, dan riwayat dikelola lewat **aplikasi Android HYRES** (Bluetooth LE + PIN).

Pintu tetap bekerja tanpa internet dan tanpa HP. Wi-Fi opsional: untuk jam otomatis (NTP) dan
HTTP API agar pintu lain (mis. pintu keluar) bisa memakai database kartu yang sama.

Versi firmware: lihat [`firmware/VERSI.txt`](firmware/VERSI.txt). Versi yang terpasang tampil
di log saat boot (`=== HYRES INTERFACE 1.2.1 ===`), di perintah `status`, dan di aplikasi
(Pengaturan → Info perangkat).

## Isi folder

| Folder / file | Isi |
|---------------|-----|
| `upload.sh` | Upload firmware (Linux) |
| `firmware/` | File firmware (`*.bin`) dan `VERSI.txt` |
| `sdcard/01/` | File suara `001.mp3` ... `021.mp3` untuk kartu SD modul MP3 |
| `aplikasi/` | Aplikasi Android HYRES (`.apk`) untuk setting dan data kartu |
| `tools/` | esptool (dipakai otomatis oleh `upload.sh`) |

## Upload firmware

Upload dari komputer **Linux 64-bit** (Ubuntu, Debian, Mint, dll). Tidak perlu Arduino IDE
dan tidak perlu memasang apa pun. Yang dibutuhkan hanya kabel USB **data** (kabel yang hanya
untuk charge tidak bisa dipakai).

Colokkan ESP32, buka terminal di folder ini, lalu jalankan:

```
bash upload.sh
```

Tunggu sampai muncul `SELESAI`, ESP32 restart sendiri.
Jika muncul `Tidak punya izin ke /dev/ttyUSB0`, jalankan sekali
`sudo usermod -aG dialout $USER`, lalu logout dan login lagi.

### Pilihan upload

| Perintah | Kegunaan |
|----------|----------|
| `bash upload.sh` | Update firmware. Wi-Fi, PIN admin, PIN Bluetooth, pairing HP, dan pengaturan **tetap**. |
| `bash upload.sh --erase` | ESP32 **baru** / reset total. Pengaturan terhapus, PIN admin kembali `000000`, PIN Bluetooth `123456`, pairing HP hilang. |
| `bash upload.sh /dev/ttyUSB1` | Pilih port jika ada lebih dari satu. |
| `BAUD=115200 bash upload.sh` | Langsung upload pelan. Biasanya tidak perlu: jika gagal, `upload.sh` otomatis mengulang di 115200. |

Data kartu dan log transaksi ada di **microSD**, bukan di ESP32, jadi **tidak ikut terhapus**
walaupun memakai `--erase`.

### Jika gagal

| Pesan | Solusi |
|-------|--------|
| `ESP32 tidak terdeteksi` | Ganti kabel USB (harus kabel data), coba port USB lain. |
| `Failed to connect` / `No serial data received` | Tahan tombol **BOOT** di ESP32 saat upload, lepas setelah muncul `Writing`. |
| `This chip is ESP32-S3, not ESP32` (atau chip lain) | Board salah, firmware ini hanya untuk ESP32-WROOM-32. |
| `upload.sh hanya untuk Linux 64-bit` | Pakai laptop / PC Linux 64-bit. |
| `UPLOAD GAGAL` (setelah diulang otomatis di 115200) | Ganti kabel USB yang lebih pendek / bagus, colok langsung ke laptop (tanpa hub USB). |
| Karakter aneh `����` di Serial Monitor sesaat setelah restart | Normal, abaikan. |

## Setting pertama (aplikasi HYRES)

Bluetooth (BLE) selalu aktif dan dilindungi PIN pairing 6 angka (awal `123456`). Setelah
pairing, perubahan data tetap butuh **PIN admin** (awal `000000`).

1. Pasang `aplikasi/HYRES-1.0.0-release.apk` di HP Android 8 ke atas (izinkan "pasang
   aplikasi tidak dikenal").
2. Nyalakan interface. Buka aplikasi **HYRES** → **Hubungkan perangkat** → **Cari perangkat** → pilih
   `HYRES INTERFACE` → masukkan PIN Bluetooth `123456`.
3. **Login admin** dengan PIN `000000`. Aplikasi mewajibkan ganti PIN admin (6–12 angka).
4. Pengaturan → **Keamanan Bluetooth** → ganti PIN Bluetooth (6 angka).
5. Pengaturan → **Wi-Fi** (disarankan): **Scan QR Wi-Fi** atau **Cari jaringan di
   sekitar perangkat** → **Sambungkan**. Hanya Wi-Fi **2,4 GHz**.
6. Tab **Kartu** → **Tambah Kartu** → **Baca kartu dari reader** → tempel kartu → isi nama,
   unit, masa berlaku → **Simpan ke perangkat**.
7. Tempel kartu tadi: relay klik dan terdengar "Selamat datang. Silakan masuk."

**Wajib ganti PIN Bluetooth dan PIN admin dari bawaan.** Jika lupa, sambungkan USB, lalu di
Serial Monitor (115200) ketik `reset-admin-pin YA` (PIN admin kembali `000000`) atau
`reset-ble-pin YA` (PIN Bluetooth kembali `123456`, semua pairing dihapus).

## Jam dan tanggal

Masa berlaku kartu butuh jam yang benar, tetapi ESP32 tidak punya baterai jam (RTC). Setelah
listrik padam, jam hilang dan **semua kartu ditolak** (suara "Akses belum dapat diproses")
sampai jam diketahui lagi:

- **Wi-Fi diatur**: jam diambil otomatis dari internet (NTP) beberapa detik setelah Wi-Fi
  tersambung, lalu terdengar "Selamat datang. Sistem siap digunakan."
- **Tanpa Wi-Fi** / internet mati: 3 detik setelah listrik menyala (30 detik jika Wi-Fi diatur)
  terdengar "Tanggal dan jam belum diatur. Silakan sambungkan ponsel...". Buka aplikasi dan
  login: jam HP dikirim otomatis ke interface.

Restart biasa (dari aplikasi / watchdog) tidak menghapus jam.

## Daftar barang (1 unit)

Kunci / pintu sudah ada di lokasi, tidak termasuk daftar ini.

**Modul utama**

| Barang | Spesifikasi | Jumlah | Dipakai untuk |
|--------|-------------|:------:|---------------|
| ESP32 DevKit | chip **ESP32-WROOM-32**, USB CP2102 | 1 | otak sistem, Bluetooth, Wi-Fi |
| Modul microSD | modul SPI microSD + kartu microSD ≤ 32 GB (**FAT32**) | 1 | database kartu, log transaksi, arsip, cadangan |
| Modul step down | LM2596 DC-DC, 12V → **5,0V** (atur trimpot sebelum dipasang) | 2 | A: ESP32 + relay + modul microSD. B: modul MP3 |
| Modul MP3 | Serial MP3 Player v1.1 (YX5300) + kartu microSD ≤ 32 GB (FAT32) | 1 | suara pengumuman |
| Speaker | colokan jack 3,5 mm, ke output modul MP3 | 1 | suara pengumuman |
| Modul relay | 1 channel 5V, ada jumper H/L | 1 | buka pintu (jumper di **H**) |
| Reader kartu | RFID 12V, output Wiegand 26/34 | 1 | kartu penghuni (set ke Wiegand 34) |
| Adaptor | 12V 3A | 1 | catu daya semua |
| Socket DC | jack female sesuai colokan adaptor (umumnya 5,5 × 2,1 mm), dipasang di box | 1 | colokan adaptor ke box |
| Kipas | 12V DC, ukuran sesuai box | 1 | pendingin di dalam box |
| Box project | plastik / ABS, ukuran secukupnya (semua modul, terminal, dan kabel muat) | 1 | wadah |

Ada **dua kartu microSD**: satu di modul microSD (data kartu & log), satu di modul MP3 (suara).
Jangan tertukar.

**Pengaman reader (wajib)**

Reader yang kami pakai mengeluarkan **5 V** di D0/D1, sedangkan pin ESP32 maksimal 3,3 V.

| Barang | Nilai | Jumlah | Dipakai untuk |
|--------|-------|:------:|---------------|
| Dioda Schottky | BAT85 (boleh 1N5819 / BAT43 / SS14) | 2 | D0 dan D1 |
| Resistor | 4k7, ¼ W | 2 | pull-up D0 dan D1 ke 3V3 |

**Kabel dan lain-lain (secukupnya)**

| Barang | Dipakai untuk |
|--------|---------------|
| Kabel jumper female-female | ESP32 ke modul-modul di dalam box |
| Kabel daya 2 core (±18 AWG) | socket DC → terminal → step down dan kipas |
| Kabel 2 core | relay COM/NO ke input buka pintu / kunci |
| Kabel twisted / shielded | ke reader kartu |
| Terminal blok sekrup | +12V, GND bersama, kabel keluar box |

Selain pengaman reader, resistor / kapasitor / elco lain **tidak wajib**: firmware memakai
pull-up internal ESP32 dan memfilter noise sendiri. Lihat
[Tambahan opsional](#tambahan-opsional) jika di lokasi ada gangguan.

## Pin ESP32

Semua sambungan langsung (kabel jumper), kecuali Wiegand D0/D1 yang lewat pengaman reader.

| GPIO | Fungsi | Sambungan |
|------|--------|-----------|
| 2   | Relay pintu | IN modul relay 5V (jumper **H**). HIGH = buka (pulsa 1 detik, bisa diubah di aplikasi) |
| 27  | Wiegand D0 | kabel D0 reader → **katoda** dioda (sisi bergaris) → **anoda** ke GPIO27. Resistor 4k7 dari GPIO27 ke 3V3 |
| 26  | Wiegand D1 | sama dengan D0: kabel D1 → katoda dioda → anoda ke GPIO26. Resistor 4k7 dari GPIO26 ke 3V3 |
| 16  | Audio TX → RX modul MP3 | langsung |
| 17  | Audio RX ← TX modul MP3 | langsung (RX/TX tertukar dideteksi otomatis) |
| 33  | microSD MISO | MISO / DO modul microSD |
| 32  | microSD MOSI | MOSI / DI modul microSD |
| 25  | microSD SCK | SCK / CLK modul microSD |
| 14  | microSD CS | CS / SS modul microSD |
| GND | GND semua modul | ESP32, step down, modul MP3, modul microSD, relay, reader, kipas: GND jadi satu |

Jangan dipakai: GPIO 1/3 (USB serial), 6–11 (flash), 0/12 (strapping boot), 34–39 (input
saja: tidak bisa untuk SCK / CS microSD).

**Reader kartu:**
- Set ke **Wiegand 34** (kabel abu-abu ke GND) agar nomor kartu utuh dan sama dengan
  reader USB. Wiegand 26 tetap terbaca, tapi nomornya berbeda untuk kartu yang sama, jadi
  jangan ganti format setelah kartu didaftarkan.
- Dioda menahan 5 V dari reader agar tidak masuk ke ESP32. Saat reader mengirim data
  (kabel ditarik ke 0 V), dioda menghantar dan GPIO terbaca LOW. Tanpa dioda, pin ESP32
  menerima 5 V terus-menerus dan bisa rusak pelan-pelan walaupun kartu tetap terbaca.
- Cek setelah dipasang: tegangan GPIO27 dan GPIO26 ke GND saat diam harus ±3,3 V (bukan
  5 V), dan tap kartu harus muncul `LOG #... uid=...` di log dengan nomor yang sama seperti
  sebelumnya. D0/D1 tertukar → semua kartu "Kartu tidak terbaca".
- Reader merek lain: ukur dulu kabel D0/D1 ke GND (reader diberi 12V, D0/D1 belum ke
  ESP32). 5 V / 12 V → pakai pengaman ini. 0 V (open-collector) → boleh langsung, tanpa dioda.

### Daya

- Adaptor 12V 3A → socket DC di box → terminal +12V.
- Step down A (LM2596) diset 5,0 V → pin 5V ESP32 + DC+ modul relay + VCC modul microSD
  (modul tanpa regulator: ambil dari pin 3V3 ESP32).
- Step down B (LM2596) diset 5,0 V → modul MP3.
- +12V langsung → reader dan kipas.
- Semua GND ke satu terminal GND.

Atur tegangan step down dengan multimeter **sebelum** disambung ke ESP32 / modul. Lebih dari
5,5 V bisa merusak ESP32 dan modul MP3.

Saat ESP32 dinyalakan / restart, GPIO2 dalam keadaan LOW, jadi relay jumper **H** tidak klik.
Jika relay klik sesaat saat ESP32 menyala, jumper masih di **L**.

### Anti-noise

- Firmware sudah memfilter noise: pulsa Wiegand < 15 µs / > 500 µs dibuang, D0 dan D1 LOW
  bersamaan dibuang, frame < 20 bit diabaikan, kartu sama dalam 3 detik diabaikan, balasan
  modul MP3 dicek formatnya, record microSD dicek CRC.
- GND semua modul wajib jadi satu. Kabel reader twisted / shielded.
- Kabel speaker jauhkan dari kabel relay / kunci.
- Jika karakter rusak `����` muncul di Serial Monitor saat alat sedang jalan, cek ground
  antara USB laptop dan adaptor 12V. Di lapangan pantau status lewat aplikasi.

### Tambahan opsional

Tidak wajib. Pasang hanya jika ada masalah di lokasi, atau untuk unit yang ingin lebih awet.

| Masalah / tujuan | Tambahan |
|------------------|----------|
| Pengaman kebakaran / korsleting | Sekring 3A + holder setelah socket adaptor 12V (sangat disarankan, murah) |
| Kartu kadang "tidak terbaca" / pulsa derau di `status` sering naik | Kapasitor 1 nF dari GPIO27 dan GPIO26 ke GND |
| Pintu sempat terbuka sendiri saat ESP32 dinyalakan | Resistor 10k dari GPIO2 ke GND |
| Suara berdengung / berdesis | Resistor 1k seri di GPIO16 → RX modul MP3. Filter LC di jalur 5V modul MP3: induktor 100 µH (≥ 1A) + elco 1000 µF 16V + kapasitor 100 nF |
| Lonjakan tegangan di jalur 12V (lokasi banyak petir / motor besar) | Dioda TVS SMBJ15A + elco 470 µF 25V dari +12V ke GND |

## microSD data (modul microSD)

Format kartu **FAT32** (≤ 32 GB), masukkan ke modul microSD. Folder `/HRL` dibuat otomatis:

| File | Isi |
|------|-----|
| `HRL/cards.dat` | database kartu (maks. 1000 kartu) |
| `HRL/txlog.dat` | 5000 log transaksi terakhir |
| `HRL/log/YYYY-MM.csv` | arsip semua log per bulan (bisa dibuka di Excel) |
| `HRL/backup/cards.dat`, `cards.csv` | cadangan database kartu (otomatis 10 detik setelah perubahan) |

**Tanpa microSD semua kartu ditolak** (suara "Terjadi gangguan sistem"). Tombol Open Gate di
aplikasi tetap bisa membuka pintu. Jangan cabut microSD saat alat menyala kecuali perlu; jika
dicabut, pasang lagi dan data dimuat ulang otomatis. Untuk cadangan manual / pemulihan:
aplikasi → Pengaturan → **Perangkat** → **Cadangkan** / **Pulihkan**.

## Kartu SD (modul MP3)

Format kartu SD **FAT32**, lalu salin folder `sdcard/01` ke root kartu, sehingga isinya
`01/001.mp3` ... `01/021.mp3`. Nama folder harus persis `01`. Cek di aplikasi → Pengaturan →
Audio: status modul harus "Normal • 21 file".

| Kode | Suara | Kapan diputar |
|------|-------|---------------|
| 001 | Selamat datang. Sistem siap digunakan. | siap dalam 30 detik setelah listrik menyala |
| 002 | Maaf, kartu Anda belum terdaftar. | kartu tidak terdaftar |
| 003 | Masa berlaku kartu Anda telah berakhir. Silakan hubungi pengelola. | kartu kedaluwarsa |
| 004 | Selamat datang. Silakan masuk. | kartu valid, pintu dibuka |
| 005 | Kartu Anda sedang tidak aktif. Silakan hubungi pengelola. | kartu dinonaktifkan |
| 006 | Masa berlaku kartu Anda belum dimulai. | sebelum tanggal mulai berlaku |
| 007 | Kartu tidak terbaca. Silakan tempelkan kembali. | data reader rusak / D0-D1 tertukar |
| 008 | Koneksi Wi-Fi terputus. | Wi-Fi putus (maks. sekali per 30 menit) |
| 009 | Sistem siap digunakan. | siap menyusul (mis. jam baru diketahui) |
| 010 | Terima kasih. Selamat jalan. | kartu valid, arah reader diset "keluar" |
| 011 | Koneksi perangkat terputus. | Bluetooth admin putus tak terduga |
| 012 | Mohon tunggu. Permintaan sedang diproses. | kartu dibaca untuk pendaftaran |
| 013 | Data kartu berhasil disimpan. | kartu baru tersimpan |
| 014 | Data kartu berhasil diperbarui. | data / status kartu diubah |
| 015 | Terjadi gangguan sistem. Silakan hubungi pengelola. | microSD tidak ada / tidak terbaca |
| 016 | Maaf, akses tidak diizinkan. | data kartu rusak |
| 017 | Akses belum dapat diproses. Silakan coba kembali. | jam belum diketahui |
| 018 | Sinkronisasi data berhasil. | sinkronisasi aplikasi selesai |
| 019 | Koneksi Wi-Fi berhasil. | Wi-Fi berhasil disimpan / diuji |
| 020 | Koneksi Wi-Fi gagal. Silakan periksa pengaturan jaringan. | Wi-Fi gagal |
| 021 | Tanggal dan jam belum diatur. Silakan sambungkan ponsel melalui aplikasi untuk mengatur waktu. | jam belum diketahui setelah listrik menyala |

Suara akses (002–007, 010, 016, 017) memotong suara lain; suara sistem dan admin antre.
Volume dan aktif / nonaktif tiap suara diatur di aplikasi.

## HTTP API (pintu lain)

Aktif otomatis selama Wi-Fi tersambung, port **80**, tanpa autentikasi (hanya untuk jaringan
lokal tepercaya). Alamat tampil di aplikasi (Beranda, dan Pengaturan → Wi-Fi → "Alamat API untuk pintu lain"). Sarankan
**IP tetap** (reservasi DHCP di router) untuk ESP32.

| Permintaan | Fungsi |
|------------|--------|
| `GET /api/transaksi/{rfid}?jenis=out` | keputusan akses untuk kartu `{rfid}`; `jenis` = `out` (bawaan) atau `in` |
| `GET /api/status` | cek server: siap, jam, versi firmware, jumlah kartu |

Contoh: `http://192.168.1.50/api/transaksi/0012345678?jenis=out` →
`{"granted":true,"reason":"GRANTED","relay":{"open":true,"pulse_ms":1000},"audio":{"folder":1,"track":10,...},...}`.
Aturan sama persis dengan reader di pintu ini dan tercatat di log yang sama. Interface ini
**tidak** membuka relay / memutar suara miliknya untuk permintaan API: peminta yang membuka
pintunya sendiri sesuai `relay.open` dan memutar `audio.track`. Server tidak terjangkau /
balasan 503 → jangan buka.

## Perintah Serial (USB)

Serial Monitor (USB, 115200 baud, akhiran newline). Butuh akses fisik ke board. Pengaturan
lewat Bluetooth memakai aplikasi HYRES.

| Perintah | Fungsi |
|----------|--------|
| `status` | versi firmware, jam, Bluetooth, Wi-Fi, microSD, kartu, log, audio, reader, relay, heap |
| `log 20` | 20 log transaksi terakhir |
| `cards` | daftar kartu terdaftar |
| `audio 4` | putar suara kode 4 (1–21) |
| `audio-info` | tanya status modul MP3 dan jumlah file di kartu SD (harus 21) |
| `relay` | buka pintu 0,5 detik (tes relay) |
| `sd` | status microSD, arsip, cadangan |
| `reboot` | restart ESP32 |
| `reset-admin-pin YA` | PIN admin kembali `000000` (wajib diganti saat login) |
| `reset-ble-pin YA` | PIN Bluetooth kembali `123456`, semua pairing dihapus |
