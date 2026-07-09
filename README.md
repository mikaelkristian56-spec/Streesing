# Streesing
URL Mode - Local Server Stress Testing Tool

Alat pengujian beban (*stress testing*) sederhana berbasis Python murni (*pure Python*) yang menggunakan metode *multi-threading* untuk mensimulasikan lalu lintas paket HTTP/HTTPS massal ke server target. Proyek ini dibuat murni untuk keperluan edukasi dan analisis performa server lokal.  jangan cuma di liat doang cobain lah

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

import threading
import http.client
import sys
import time
import argparse
import random
from urllib.parse import urlparse

def get_arguments():
    parser = argparse.ArgumentParser(description="Local Server Stress Testing Tool (Hammer URL Mode - No Protection)")
    # -s menerima Full URL (Bisa pakai nama domain lokal kustom)
    parser.add_argument("-s", "--url", required=True, help="Target URL lengkap (contoh: http://webgue.local/ atau http://mysite.test:5000/page)")
    # -t untuk jumlah thread/turbo
    parser.add_argument("-t", "--turbo", type=int, default=100, help="Jumlah thread / kekuatan turbo (default: 100)")
    return parser.parse_args()

# Kumpulan User-Agent acak biar bot bervariasi
USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.2 Safari/605.1.15",
    "Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Mobile Safari/537.36",
    "Mozilla/5.0 (iPhone; CPU iPhone OS 17_2 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.2 Mobile/15E148 Safari/605.1.15",
    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
]

request_counter = 0
failed_counter = 0
lock = threading.Lock()

def turbo_bot(host, port, path, is_ssl):
    """Bot turbo yang menembak langsung ke path URL target"""
    global request_counter, failed_counter
    
    while True:
        try:
            # Menentukan jenis koneksi (HTTP atau HTTPS)
            if is_ssl:
                conn = http.client.HTTPSConnection(host, port, timeout=3)
            else:
                conn = http.client.HTTPConnection(host, port, timeout=3)
            
            # Pipelining flood paket (50 request per koneksi)
            for _ in range(50):
                random_ua = random.choice(USER_AGENTS)
                
                conn.request("GET", path, headers={
                    "User-Agent": random_ua,
                    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
                    "Accept-Language": "en-US,en;q=0.5",
                    "Connection": "keep-alive"
                })
                
                response = conn.getresponse()
                response.read() # Buang data respon untuk menghemat RAM komputer lu
                
                with lock:
                    request_counter += 1
            
            conn.close()
        except Exception:
            with lock:
                failed_counter += 1
            time.sleep(0.1) # Jeda singkat jika server mulai tumbang/RTO

def monitor(url, threads):
    global request_counter, failed_counter
    print(f"==================================================")
    print(f"[+] Target URL    : {url}")
    print(f"[+] Turbo Threads : {threads} Bot")
    print(f"[+] Status        : Pengujian Beban Dimulai...")
    print(f"==================================================")
    
    while True:
        try:
            time.sleep(1)
            with lock:
                # Menggunakan \r supaya counter terupdate rapi di satu baris terminal
                print(f"[MONITOR] Total Paket Terkirim: {request_counter} | Gagal/RTO: {failed_counter}", end="\r")
        except KeyboardInterrupt:
            print("\n[-] Pengujian dihentikan.")
            sys.exit()

if __name__ == "__main__":
    args = get_arguments()
    
    # Memecah komponen URL
    parsed_url = urlparse(args.url)
    
    # Validasi skema URL
    if not parsed_url.scheme in ('http', 'https'):
        print("[-] Error: URL harus diawali dengan http:// atau https://")
        sys.exit(1)
        
    host = parsed_url.hostname
    path = parsed_url.path if parsed_url.path else "/"
    is_ssl = parsed_url.scheme == "https"
    
    # Otomatisasi pendeteksian port
    if parsed_url.port:
        port = parsed_url.port
    else:
        port = 443 if is_ssl else 80

    # Jalankan thread monitor
    mon_thread = threading.Thread(target=monitor, args=(args.url, args.turbo), daemon=True)
    mon_thread.start()

    # Jalankan pasukan bot sesuai jumlah turbo (-t)
    for i in range(args.turbo):
        t = threading.Thread(target=turbo_bot, args=(host, port, path, is_ssl))
        t.daemon = True
        t.start()

    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\n[-] Selesai.")
   
