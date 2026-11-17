# Frequency and Consequence

| | |
|---|---|
| **Bagian** | wave 2 |
| **Kategori** | misc |
| **Jumlah Solve** | 373 |

## Ikhtisar

Artefaknya adalah `signal.wav`: 9,0 detik, 44,1 kHz, mono, PCM 16-bit. Ini membawa enam belas nada konstan yang terkubur dalam kebisingan broadband, satu nada per 500 ms. Enam belas nada diambil dari 32 simbol alfabet dengan jarak 50 Hz mulai dari 500 Hz (500, 550,…, 2050 Hz), yang dipetakan tepat ke alfabet base32 `A–Z2–7`. Tujuannya adalah nonce 16 karakter.

## Penyelesaian

Membaca WAV dan menghitung puncak spektral geser menunjukkan bahwa sinyal diatur dalam blok 500 ms. Enam belas blok rentang waktu 8 detik; file tersebut berdurasi 9 detik, jadi pesan dimulai pada 0,5 detik dan berakhir pada 8,5 detik, dengan noise 0,5 detik sebelum dan sesudahnya. Jendela yang dimulai pada kelipatan 0,5 detik menghasilkan satu puncak bersih (magnitudo ≈ 2,4·10⁷), sedangkan jendela diimbangi dengan 0,25 detik mengangkangi dua simbol dan memberikan spektrum campuran (≈ 1,2·10⁷), yang menetapkan batas slot pada `[0.5 + 0.5k, 1.0 + 0.5k)`.

Untuk setiap slot, ukur energi di seluruh 32 kandidat frekuensi dengan algoritma Goertzel (integrasi koheren 0,5 detik, yaitu DFT resolusi 2 Hz):

```python
for k in range(16):
    seg = x[int((0.5 + 0.5*k)*fs):int((1.0 + 0.5*k)*fs)]
    mags = [goertzel(seg, 500 + 50*i) for i in range(32)]
    sym  = ALPHABET[max(range(32), key=lambda i: mags[i])]
```

Nada kemenangan di setiap slot mengalahkan runner-up dengan faktor setidaknya ~21, jadi keputusannya tidak ambigu. Mengindeks setiap nada yang terdeteksi melalui `ABCDEFGHIJKLMNOPQRSTUVWXYZ234567` menghasilkan nonce base32.

Nonce 16 karakter adalah `████████████████`.