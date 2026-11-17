# Between the Lines

| | |
|---|---|
| **Bagian** | wave 7 |
| **Kategori** | web |
| **Jumlah Solve** | 121 |

## Ikhtisar

Proksi terbalik berada di depan server aplikasi. Proksi menerapkan aturan akses pada `/internal/*`, tetapi kedua komponen mengurai framing pesan HTTP/1.1 secara berbeda. Dengan mengeksploitasi ketidaksepakatan tersebut, permintaan dapat diselundupkan melewati pemeriksaan jalur proxy dan dieksekusi oleh back end.

## Penyelesaian

Layanan di `between-the-lines.pointeroverflowctf.com:9008` mengidentifikasi dirinya sebagai `aperture-telephone/0.1`, dengan proksi diidentifikasi sebagai `aperture-telephone/0.1-proxy`. Permintaan normal menunjukkan bahwa bagian belakang mengiklankan `/echo` dan `/internal/*` dibatasi; permintaan langsung ke `/internal/` dijawab oleh proxy dengan `403 Forbidden`.

Untuk menentukan arah desinkronisasi, saya mengirim `POST /echo` yang membawa `Content-Length` dan `Transfer-Encoding: chunked`, dengan `Content-Length` lebih pendek dari bodi yang dipotong. Bagian belakang menjawab dengan tampilan permintaan pengurai tubuhnya: ia memenuhi pengkodean yang dipotong (melaporkan isi yang kosong), sedangkan ujung depan memenuhi `Content-Length`. Oleh karena itu, ujung depan menggunakan `Content-Length` dan ujung belakang menggunakan `Transfer-Encoding` — desinkronisasi CL.TE.

Badan eksploitasi dibuat sedemikian rupa sehingga proxy akan meneruskannya secara penuh sebagai badan permintaan, sedangkan bagian belakang akan melihat akhir dari badan yang dipotong lebih awal dan menafsirkan byte yang tersisa sebagai permintaan baru:

```http
0

GET /internal/flag HTTP/1.1
Host: between-the-lines.pointeroverflowctf.com
X-Team-Token: <team token from the challenge page>
Connection: close

```

`Content-Length` yang dideklarasikan mencakup seluruh payload, sehingga proxy memperlakukan permintaan yang disematkan sebagai byte isi biasa dan tidak pernah menerapkan aturan `/internal/*` padanya. Bagian belakang menghentikan isi potongan pada potongan `0` dan kemudian menguraikan byte berikut sebagai permintaan baru, yang mencapai penangan terbatas.

Proksi meneruskan respons pipeline ujung belakang pada koneksi yang sama, memperlihatkan balasan pengendali yang dibatasi bersama dengan respons `/echo`. Balasannya berisi penanda tantangan. Flag yang dikirimkan adalah POCTF{83.1814.████████████████.████████████████} dan diterima.