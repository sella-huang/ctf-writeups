# The Logic Inside

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | web |
| **Jumlah Solve** | 262 |

## Ikhtisar

Layanan pratinjau URL mengambil URL yang disediakan pengguna di sisi server dan hanya melaporkan hasil dan kode status HTTP hulu. Filter nama host memblokir loopback dan alamat pribadi, tetapi filter tersebut dapat dilewati dengan representasi alternatif `127.0.0.1`. Hal ini mengubah layanan pratinjau menjadi oracle kode status terhadap aplikasi kedua yang terikat ke `127.0.0.1:9001`, yang titik akhir `/flag`-nya memeriksa awalan nonce.

## Penyelesaian

Titik akhir pratinjau menerima isi JSON dalam bentuk `{"url": "..."}` dan mengembalikan objek JSON kecil seperti:

```json
{"note": "preview generated", "ok": true, "upstream_status": 200}
```

Permintaan ke loopback literal dan host pribadi ditolak sebelum pengambilan apa pun:

```json
{"error": "host blocked by policy", "hostname_seen": "127.0.0.1", "ok": false}
```

Filter beroperasi pada string nama host, bukan pada alamat yang diselesaikan. Host berlabel tunggal `0` lolos filter dan diselesaikan ke `127.0.0.1`; bentuk alamat desimal dan oktal juga berfungsi. Oleh karena itu, layanan pratinjau dapat diarahkan ke aplikasi internal:

```text
http://0:9001/flag?guess=<prefix>
```

Aplikasi internal mengembalikan `200` ketika string heksadesimal yang disediakan adalah awalan nonce 16 karakter, dan `404` sebaliknya (`400` untuk tebakan kosong). Karena layanan pratinjau melaporkan kode status upstream, layanan ini bertindak sebagai awalan oracle.

Saya memulihkan karakter nonce satu per satu. Untuk setiap posisi, awalan yang diketahui saat ini diperpanjang dengan masing-masing kandidat dari `0123456789abcdef`; calon yang berstatus `200` dipertahankan, dan pencarian dipindahkan ke posisi berikutnya:

```python
prefix = ""
for _ in range(16):
    for ch in "0123456789abcdef":
        if upstream_status(f"http://0:9001/flag?guess={prefix}{ch}") == 200:
            prefix += ch
            break
```

Enam belas posisi menghasilkan nonce 16 karakter lengkap, dan kueri akhir dengan nilai penuh mengembalikan `200`. Nonce yang dikirimkan adalah ████████████████ dan diterima.