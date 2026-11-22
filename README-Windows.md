# NetScan — Pemindai IP & Perangkat Jaringan untuk Windows

Aplikasi desktop ringan (mirip **Fing**) untuk melihat perangkat apa saja yang terhubung ke
jaringan lokal: alamat IP, MAC, vendor, nama perangkat, latensi, dan port yang terbuka.

Satu file `NetScan.exe`, tanpa installer, tanpa runtime tambahan (memakai .NET Framework 4.x
yang sudah tersedia di Windows 10/11).

## Fitur

- Deteksi otomatis adapter jaringan aktif (Wi-Fi/Ethernet) beserta subnet-nya.
- Pemindaian cepat: sapuan ping + tabel ARP, biasanya **2–5 detik** untuk jaringan /24.
- Menampilkan perangkat yang memblokir ping (tetap terdeteksi lewat ARP) dengan status khusus.
- MAC address + **vendor/merek** dari database OUI IEEE (38.900+ prefix, tertanam di exe).
- Nama perangkat (reverse DNS), latensi (ms), dan port umum terbuka + nama layanannya
  (HTTP, HTTPS, SSH, RDP, SMB, Chromecast, printer, dll).
- Tabel bisa **disortir** (klik header; kolom IP diurutkan numerik) dan **difilter**
  (IP / nama / MAC / vendor).
- Panel detail perangkat, ekspor **CSV**, salin ke clipboard.
- Menu klik kanan: salin IP/MAC, ping berkelanjutan, buka di browser.
- Pemindaian otomatis tiap 45 detik (opsional), tombol Stop (Esc).
- Antarmuka bahasa Indonesia, DPI-aware, ikon aplikasi tertanam.

## Cara pakai

1. Jalankan `dist\NetScan.exe`.
2. Pilih adapter/jaringan pada dropdown (default: adapter yang punya gateway).
3. Klik **Pindai** (atau tekan **F5**).
4. Klik baris perangkat untuk melihat detail di panel bawah.
5. **Ekspor CSV** untuk menyimpan hasil; **Salin** untuk menyalin tabel ke clipboard.

Pintasan: `F5` pindai, `Esc` stop, `Ctrl+F` cari, `Ctrl+S` ekspor.

Mode baris perintah: `NetScan.exe /scan` langsung memindai saat aplikasi dibuka.

## Cara kerja singkat

1. **Ping sweep** seluruh subnet (timeout 500 ms, banyak paralel) untuk memancing resolusi ARP.
2. **Baca tabel ARP** Windows (`GetIpNetTable`), sehingga perangkat yang memblokir ICMP tetap
   terdeteksi beserta MAC-nya.
3. **Detail**: reverse DNS untuk perangkat yang merespons, lalu koneksi TCP singkat ke 21 port
   umum untuk mengetahui layanan yang terbuka.
4. **Vendor MAC** dicocokkan dengan database OUI yang dikompres dan ditanam di dalam exe.

Aplikasi hanya membaca informasi jaringan; tidak mengubah konfigurasi apa pun.

## Batasan

- Perangkat dengan **client/AP isolation** (mis. Wi-Fi tamu) tidak akan terlihat.
- Firewall perangkat dapat memblokir ping/port sehingga status menjadi "Tanpa ping" dan daftar
  port kosong — perangkat tetap muncul karena ARP.
- Jaringan lebih besar dari 2046 host dipindai sebagian (batas pengaman).
- Membutuhkan .NET Framework 4.8 (standar di Windows 10/11). Di Windows lama, pasang .NET
  Framework 4.8 terlebih dahulu.
- Exe tidak ditandatangani secara digital. Jika SmartScreen muncul, pilih **More info → Run anyway**
  (kode sumbernya ada di folder `src` dan bisa dibangun ulang sendiri).
- Nama perangkat hanya muncul jika router/DNS lokal menyediakan reverse DNS.

## Versi Android

Ada juga versi Android (APK) dengan fitur setara di folder [`NetScanAndroid`](../NetScanAndroid/README.md):
`powershell -ExecutionPolicy Bypass -File .\NetScanAndroid\build.ps1`.

## Build dari sumber

Tidak perlu install SDK: compiler C# bawaan Windows (`%WINDIR%\Microsoft.NET\Framework64\v4.0.30319\csc.exe`)
dipakai untuk menghasilkan exe.

```powershell
powershell -ExecutionPolicy Bypass -File .\NetScan\build.ps1
```

Hasil: `NetScan\dist\NetScan.exe`.

Untuk membuat ulang aset (opsional):

```powershell
powershell -ExecutionPolicy Bypass -File .\NetScan\tools\make-oui.ps1    # database vendor
powershell -ExecutionPolicy Bypass -File .\NetScan\tools\make-icon.ps1   # ikon aplikasi
```

## Struktur proyek

```
NetScan/
├─ src/                 kode C# (WinForms, .NET Framework 4.x)
│  ├─ Program.cs        entry point + mode /scan
│  ├─ MainForm.cs       UI & layout
│  ├─ MainForm.Actions.cs  alur scan, filter, ekspor, menu
│  ├─ ScanEngine.cs     ping sweep, ARP, DNS, port scan
│  ├─ AdapterInfo.cs    enumerasi adapter & subnet
│  ├─ Device.cs         model perangkat
│  ├─ OuiLookup.cs      lookup vendor MAC
│  └─ SortableBindingList.cs
├─ assets/              ikon, manifest, database OUI (gz)
├─ tools/               skrip pembuat aset + smoke test
├─ build.ps1            kompilasi ke dist/NetScan.exe
└─ dist/                hasil build
```

## Atribusi

- Data OUI/vendor: IEEE MA-L, didistribusikan melalui
  [nmap-mac-prefixes](https://github.com/nmap/nmap/blob/master/nmap-mac-prefixes).
- Ikon dibuat secara programatik (tanpa aset pihak ketiga).
