# HackTool — Android (APK)

Ini project Capacitor yang membungkus app kontrol HackTool menjadi APK
Android asli. Hasilnya adalah aplikasi mandiri: punya ikon sendiri, proses
sendiri, jalan lewat WebView bawaan sistem Android — bukan lewat Chrome.

## Yang perlu diinstall di komputer kamu (sekali saja)

1. **Node.js** (LTS) — https://nodejs.org
2. **Android Studio** — https://developer.android.com/studio
   Saat instalasi pertama, biarkan Android Studio mengunduh Android SDK
   default-nya.

## Langkah build

Buka terminal di folder project ini (folder yang berisi `package.json`),
lalu jalankan:

```bash
npm install
npx cap add android
npx cap sync android
```

Ini akan membuat folder `android/` berisi project Android native lengkap,
dengan `www/index.html` sudah otomatis dimasukkan sebagai halaman utama.

### Buka & build di Android Studio

```bash
npx cap open android
```

Perintah ini membuka folder `android/` di Android Studio. Tunggu proses
Gradle sync selesai (pertama kali bisa beberapa menit), lalu:

- **Build > Build Bundle(s) / APK(s) > Build APK(s)**
- Setelah selesai, klik notifikasi "locate" atau cari file APK di:
  `android/app/build/outputs/apk/debug/app-debug.apk`

### Pasang ke HP

1. Salin `app-debug.apk` ke HP Android (lewat USB, share, dsb).
2. Buka file itu di HP → izinkan "Install dari sumber tidak dikenal" jika
   diminta → Install.
3. Buka app **HackTool** dari app drawer seperti app biasa.

## Catatan penting

- App ini konek ke ESP32-C3 lewat WebSocket biasa (`ws://`, bukan `wss://`
  yang terenkripsi). Karena itu `capacitor.config.json` sudah diset
  `"cleartext": true` supaya Android mengizinkan koneksi tanpa enkripsi ke
  IP lokal seperti `192.168.1.1`. Jangan dihapus, kalau tidak koneksi akan
  diblokir Android sejak versi 9 ke atas.
- Kalau ESP32-C3 pakai mode Access Point (`HackTool` / `192.168.1.1`),
  HP kamu harus konek ke WiFi itu dulu sebelum buka app-nya (HP tidak akan
  ada akses internet selama konek ke AP ini — itu wajar).
- Setiap kali kamu ubah tampilan/kode di `www/index.html`, jalankan ulang
  `npx cap sync android` lalu build APK lagi supaya perubahan ikut masuk.
- App sekarang pakai getaran (haptic) saat tombol ditekan. Tambahkan izin ini
  di `android/app/src/main/AndroidManifest.xml` (di luar tag `<application>`)
  supaya getaran berfungsi di APK:
  ```xml
  <uses-permission android:name="android.permission.VIBRATE" />
  ```
  Lalu jalankan `npx cap sync android` lagi dan build ulang.
- Default token akses kontrol adalah `110610` — ganti lewat menu Pengaturan
  di app begitu APK pertama kali dipasang, supaya orang lain yang ikut
  konek ke WiFi `HackTool` tidak bisa asal kontrol.
- File `www/index.html` di sini sama persis dengan app WebSocket yang
  sudah kita buat, minus bagian khusus PWA browser (manifest/install
  prompt) yang tidak relevan lagi karena sekarang sudah jadi APK asli.

## Kenapa tidak langsung dikompilkan APK-nya di sini?

Lingkungan ini tidak punya akses internet dan Android SDK/build tools,
jadi saya tidak bisa menjalankan proses build Android langsung. Tapi
seluruh source code project sudah lengkap dan siap pakai — tinggal jalankan
tiga perintah di atas di komputer kamu.

## Pembaruan: layar OLED + sinkron dengan firmware (ESP32-C3 & ESP32 basic)

- `www/index.html` sekarang sudah memuat **kanvas layar OLED 128x64**. Setelah login,
  app mengirim `OLED:1` ke ESP32-C3; C3 lalu mengirim frame biner (FULL/DELTA) yang
  digambar ke kanvas, plus snapshot layar terakhir. Ketuk layar untuk pindah antara
  tampilan OLED dan log teks. Baris teks dari firmware (`[MENU] ...`, `[FLASH] ...`)
  tetap tampil di log.
- Tanpa `OLED:1` (mis. app lama), C3 hanya mengirim teks - jadi app versi lama tidak rusak.
- **Jangan edit HTML di dua tempat.** Edit hanya `www/index.html`, lalu di folder induk
  (yang berisi `hacktool_wifi_bridge/`) jalankan `python3 sync_portal.py` supaya
  portal yang disajikan ESP32-C3 identik dengan APK. Sesudahnya: upload ulang firmware
  C3, lalu `npx cap sync android` dan build APK lagi.
- Host default `192.168.1.1:81` sudah cocok dengan WebSocket ESP32-C3 (port 81).

## Pembaruan: tombol RESET ESP32 basic & ambang baterai lemah

- **Tombol RESET** (di bawah BACK): mengirim `MRST` ke ESP32-C3, yang menarik pin
  `MAIN_RESET_PIN` (default GPIO2, sambungkan ke pin **EN** ESP32 basic) LOW selama 200 ms.
  ESP32-C3 tetap menyala; layar app kosong sampai ESP32 basic selesai boot.
  Masuk menu bootloader: **tahan tombol OK di app, ketuk RESET, tetap tahan OK ~1,5 detik**
  (firmware yang berjalan harus memanggil `WebDisplayBootCheck()` di baris pertama `setup()`).
- **Pengaturan > Kalibrasi Baterai** punya kolom **Ambang baterai lemah (%)** (default 20).
  Di bawah ambang: LED daya berkedip (lambat di ambang, makin cepat mendekati 0%) dan
  ikon baterai di app merah. Di atas ambang: LED daya mati.
- Pesan galat dari ESP32-C3 untuk pengaturan baterai/sleep (`BAT:ERR`, `SLEEP:ERR`) sekarang
  ditampilkan di app (sebelumnya diabaikan).
- Setelah mengubah `www/index.html`: jalankan `python3 sync_portal.py` (folder induk),
  upload ulang firmware C3, lalu `npx cap sync android` dan build ulang APK.

## Ikon aplikasi

- Ikon dari gambar buatan kamu (`resources/icon.png`, 1024x1024) sudah dipakai di app (favicon, boot screen,
  apple-touch-icon, manifest).
- Untuk APK: setelah `npx cap add android`, salin isi folder `android-res/` ke
  `android/app/src/main/res/` (timpa file yang sama). Isinya `mipmap-*/ic_launcher*.png`,
  ikon adaptif (`ic_launcher_foreground.png` + warna latar `#0D1216`), dan `values/ic_launcher_background.xml`.
  Lalu `npx cap sync android` dan build ulang.
- Ambang baterai lemah default sekarang **20%** (firmware C3 dan app). Perangkat yang sebelumnya sudah menyimpan
  nilai sendiri di flash tetap memakai nilai lama sampai diubah di Pengaturan atau di-reset total (tombol fisik).
- IP Access Point ESP32-C3 sekarang **192.168.1.1** (bukan 192.168.4.1).
