# Streesing
URL Mode - Local Server Stress Testing Tool

Alat pengujian beban (*stress testing*) sederhana berbasis Python murni (*pure Python*) yang menggunakan metode *multi-threading* untuk mensimulasikan lalu lintas paket HTTP/HTTPS massal ke server target. Proyek ini dibuat murni untuk keperluan edukasi dan analisis performa server lokal.

## Fitur
- **Full URL Support**: Menerima input URL utuh (`-s`) tanpa perlu memisahkan IP, Port, atau Path secara manual.
- **Multi User-Agent Rotation**: Setiap bot secara acak menyamar menggunakan User-Agent browser populer (Chrome, Safari, Firefox) agar simulasi traffic terlihat lebih realistis.
- **High-Speed Pipelining**: Mengoptimasikan koneksi *Keep-Alive* untuk membanjiri request dengan konsumsi CPU/RAM seminimal mungkin di sisi pengirim.
- **Real-Time Monitor**: Menampilkan jumlah paket yang sukses terkirim dan paket yang gagal (RTO/Error) secara langsung di terminal.

## Cara Instalasi

1. Clone repositori ini ke komputer lu:
   ```bash
   git clone https://github.com/mikaelkristian56-spec/streesing
   cd streesing
   python streesing.py -s url -t turbo

   
