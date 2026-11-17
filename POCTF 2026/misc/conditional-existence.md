# Conditional Existence

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | misc |
| **Jumlah Solve** | 163 |

## Ikhtisar

Arsip per tim berisi program C kecil dan sumbernya. Tanpa variabel lingkungan yang diperlukan, program hanya mencetak `.`. Sumber memeriksa `POCTF_INCANTATION`: harus tepat 12 karakter heksadesimal huruf besar. Nilai tersebut digunakan sebagai kunci XOR berulang pada ciphertext yang tertanam, dan program memverifikasi SHA-256 dari teks biasa yang didekripsi terhadap intisari yang tertanam. Sumbernya juga berisi teka-teki empat bagian yang jawabannya menghasilkan mantera.

## Penyelesaian

Keempat jawaban teka-teki tersebut adalah:

- tahun kematian Helena P. Blavatsky di London: `1891`;
- nama belakang penyelidik paranormal London yang menjalankan Laboratorium Nasional Penelitian Psikis dan menyelidiki Borley Rectory: `price`;
- pinggiran kota India tempat Theosophical Society memindahkan kantor pusat internasionalnya pada tahun 1882: `adyar`;
- nama satu kata dari scryer Enochian yang mencatat komunikasi malaikat dengan John Dee: `kelley`.

Oleh karena itu frasa tersebut adalah `1891 price adyar kelley`. Menggabungkan jawaban dengan spasi tunggal, melakukan hashing pada byte UTF-8 dengan SHA-256, dan mengambil 12 karakter heksadesimal huruf besar pertama akan menghasilkan mantra yang diperlukan.

```python
import hashlib

phrase = "1891 price adyar kelley"
incantation = hashlib.sha256(phrase.encode()).hexdigest()[:12].upper()
print(incantation)
```

Menjalankan program yang disediakan dengan nilai tersebut di `POCTF_INCANTATION` akan mendekripsi ciphertext, memenuhi pemeriksaan hash yang tertanam, dan mencetak flag.

Flag-nya adalah ████████████████.