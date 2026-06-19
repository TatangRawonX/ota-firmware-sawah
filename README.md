# OTA Firmware — Pest Controller Sawah

Repo ini cuma buat nyimpen file .bin firmware ESP32 (Box1) dan ESP8266 (Box3) untuk OTA update jarak jauh.

## Cara update firmware (dari jarak jauh, tanpa kabel USB)

1. Edit kode `.ino` di Arduino IDE seperti biasa
2. **Naikkan nomor versi** di kode, contoh:
   ```cpp
   #define FW_VERSION    "1.0.1"   // sebelumnya "1.0.0"
   ```
3. Compile: `Sketch` → `Export Compiled Binary`
   File `.bin` akan muncul di folder project yang sama dengan nama seperti `ESP32Box1.ino.esp32.bin`
4. Buka tab **"Releases"** di repo ini → **"Draft a new release"**
5. Buat tag baru, contoh `v1.0.1`
6. Upload file `.bin` sebagai asset, **rename jadi**:
   - `box1.bin` untuk ESP32 Box 1
   - `box3.bin` untuk ESP8266 Box 3
7. **Edit `version.json`** di branch `main` (bisa langsung dari web GitHub, klik pensil edit), ubah:
   - `version` → samakan dengan tag baru (misal `"1.0.1"`)
   - `url` → samakan juga tag-nya di URL (misal `.../download/v1.0.1/box1.bin`)
8. Commit langsung ke `main`
9. Publish release

## Bagaimana ESP update sendiri

- Tiap box otomatis cek `version.json` ini **setiap 1 jam**
- Kalau `version` di file ini beda dari yang tertanam di kode (`FW_VERSION`), ESP otomatis download `.bin` dan flash sendiri
- Setelah selesai, ESP **restart otomatis** dan langsung pakai firmware baru

## Cek update lebih cepat (tanpa nunggu 1 jam)

Buka Firebase Console → Realtime Database → set manual:
```
esp1/ota_check_now = true   (untuk Box1)
esp3/ota_check_now = true   (untuk Box3)
```
ESP akan cek dalam beberapa detik, lalu flag ini otomatis kembali ke `false`.

## Cek status OTA

Lihat di Firebase:
```
esp1/ota_status   → "Idle" / "Updating to 1.0.1" / "OTA sukses ke 1.0.1" / "OTA gagal: ..."
esp1/fw_version    → versi yang sedang berjalan saat ini
```

## ⚠️ PENTING — Jangan sampai bata (brick)

- **Selalu naikkan `FW_VERSION` di kode** sebelum compile — kalau tidak dinaikkan, ESP akan menganggap sudah versi terbaru dan tidak update
- **Jangan pernah upload `.bin` yang salah board** (Box1 = ESP32, Box3 = ESP8266) — kalau salah, ESP bisa gagal boot
- Tes dulu firmware baru pakai kabel USB sebelum dijadikan release OTA, supaya kalau ada bug fatal, ketahuan duluan sebelum dikirim jarak jauh
- Kalau OTA gagal di tengah jalan (download putus, dll), ESP **tetap jalan dengan firmware lama** — tidak langsung bata, karena flash baru ditulis di partition terpisah dan baru dipakai kalau proses selesai sempurna
