# NetScan Android — Pemindai Perangkat Jaringan (APK)

Versi Android dari NetScan: melihat perangkat apa saja yang terhubung ke Wi-Fi yang sama —
alamat IP, MAC, vendor, nama perangkat, latensi, dan port yang terbuka.

Hasil build siap pasang: **`dist\NetScan.apk`** (± 414 KB, Android 7.0+).

## Fitur

- Deteksi otomatis jaringan Wi-Fi/Ethernet aktif beserta subnet dan gateway-nya.
- Pemindaian paralel (32 thread ping) — jaringan `/24` biasanya selesai dalam beberapa detik.
- Perangkat yang memblokir ping tetap terdeteksi lewat tabel ARP/neighbour, ditandai "Tanpa ping".
- MAC + **vendor** dari database OUI IEEE 38.900+ prefix (tertanam di APK).
- Nama perangkat via reverse DNS **dan** NetBIOS (banyak perangkat Windows/NAS tidak punya reverse DNS).
- Port umum terbuka + nama layanannya (HTTP, HTTPS, SSH, SMB, Chromecast, printer, Plex, dll).
- Panel detail perangkat, tombol **Buka** untuk perangkat dengan web server.
- Ekspor/salin hasil dalam format CSV (dipisah `;`, siap dibuka di Excel).
- Subnet luas (`/8` … `/16`) dipersempit ke jendela maksimum 1022 alamat di sekitar HP,
  sehingga pemindaian tetap cepat dan tidak membebani jaringan.
- Tanpa iklan, tanpa pelacakan, tanpa koneksi ke internet — semua data tetap di HP.

## Cara pakai

1. Salin `dist\NetScan.apk` ke HP (kabel USB, Google Drive, atau chat).
2. Buka berkasnya, izinkan **"Instal dari sumber tidak dikenal"** bila diminta.
3. Sambungkan HP ke Wi-Fi yang sama dengan perangkat yang ingin dipindai.
4. Aplikasi langsung memindai saat dibuka; tekan **Pindai** untuk mengulang atau **Stop** untuk berhenti.
5. Ketuk baris perangkat untuk melihat detail; tekan lama untuk menyalin IP-nya.
6. **Salin** / **Bagikan** mengirim hasil sebagai teks CSV.

## Catatan penting

- **MAC address di Android 10+.** Android membatasi akses `/proc/net/arp` sejak Android 10.
  Aplikasi mencoba beberapa sumber (`/proc/net/arp`, `ip neigh show`, `arp -an`). Bila semuanya
  diblokir, kolom MAC akan kosong dan status pemindaian memberi tahu hal itu. Ini batasan sistem,
  bukan bug aplikasi — aplikasi tidak memakai root.
- **Ping bisa diblokir.** Banyak perangkat (terutama iPhone dan Windows dengan firewall ketat)
  tidak menjawab ping. Perangkat seperti itu tetap muncul dengan status "Tanpa ping" selama
  masih ada di tabel ARP.
- **Nama perangkat** hanya muncul bila router/DNS lokal menyediakan reverse DNS atau perangkat
  menjawab NetBIOS. Kueri NetBIOS dibatasi 32 host pertama per pemindaian agar tetap cepat.
- **VPN aktif** dapat membuat subnet yang terbaca bukan jaringan Wi-Fi sebenarnya. Matikan VPN
  bila hasil pemindaian terasa aneh.
- Jaringan dengan **isolasi klien** (umum di Wi-Fi publik/kantor) tidak akan menampilkan
  perangkat lain, meskipun keduanya terhubung ke SSID yang sama.

## Izin aplikasi

| Izin | Alasan |
| --- | --- |
| `INTERNET` | Ping, koneksi TCP ke port perangkat, dan kueri NetBIOS (semua hanya ke jaringan lokal). |
| `ACCESS_NETWORK_STATE` | Membaca jaringan aktif, subnet, dan gateway. |
| `ACCESS_WIFI_STATE` | Menentukan apakah koneksi aktif berupa Wi-Fi. |

## Build dari sumber

Tidak perlu Android Studio maupun Gradle. Skrip memakai JDK 17 + Android SDK
platform 33 + build-tools 33.0.2 yang diunduh otomatis (sekali saja, ± 310 MB) ke
`%LOCALAPPDATA%\NetScanAndroid`.

```powershell
powershell -ExecutionPolicy Bypass -File .\NetScanAndroid\build.ps1
```

Hasil: `NetScanAndroid\dist\NetScan.apk`.

`build.ps1` memanggil `tools\setup-sdk.ps1` sendiri bila toolchain belum ada. Untuk menyiapkan
toolchain tanpa membangun APK:

```powershell
powershell -ExecutionPolicy Bypass -File .\NetScanAndroid\tools\setup-sdk.ps1
```

Semua unduhan diverifikasi checksum (SHA-256 untuk JDK, SHA-1 untuk SDK) dan skrip aman
dijalankan berulang. Alat dipanggil langsung (`aapt2`, `javac`, `d8`, `zipalign`, `apksigner`),
bukan lewat `d8.bat`/`apksigner.bat` yang tidak kompatibel dengan JDK 9+.

APK ditandatangani dengan keystore lokal `keystore\netscan.jks` yang dibuat otomatis pada build
pertama. Simpan berkas itu bila ingin merilis pembaruan — APK dengan kunci berbeda tidak bisa
menimpa instalasi lama.

Membuat ulang ikon launcher (opsional):

```powershell
powershell -ExecutionPolicy Bypass -File .\NetScanAndroid\tools\make-icons.ps1
```

## Struktur proyek

```
NetScanAndroid/
├─ app/
│  ├─ AndroidManifest.xml
│  ├─ assets/oui.csv.gz        database vendor MAC (gzip)
│  ├─ java/com/netscan/app/
│  │  ├─ MainActivity.java     UI, dialog detail, salin/bagikan CSV
│  │  ├─ DeviceAdapter.java    daftar perangkat
│  │  ├─ Scanner.java          ping sweep, detail, port scan
│  │  ├─ ArpTable.java         tabel ARP dari 3 sumber + MAC sendiri
│  │  ├─ NetbiosName.java      kueri nama NetBIOS (NBSTAT)
│  │  ├─ NetInfo.java          jaringan aktif, subnet, gateway
│  │  ├─ Subnet.java           perhitungan subnet (murni Java)
│  │  ├─ Device.java           model perangkat
│  │  └─ OuiDb.java            lookup vendor MAC
│  └─ res/                     layout, gambar, warna, string
├─ tools/
│  ├─ setup-sdk.ps1            unduh & siapkan JDK + Android SDK
│  └─ make-icons.ps1           ikon launcher PNG
├─ build.ps1                   kompilasi + tanda tangan -> dist/NetScan.apk
└─ dist/                       hasil build
```

## Atribusi

- Data OUI/vendor: IEEE MA-L, didistribusikan melalui
  [nmap-mac-prefixes](https://github.com/nmap/nmap/blob/master/nmap-mac-prefixes).
- Ikon dibuat secara programatik (tanpa aset pihak ketiga).
