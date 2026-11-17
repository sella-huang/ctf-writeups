# Invisible Ink

| | |
|---|---|
| **Bagian** | wave 1 |
| **Kategori** | steganography |
| **Jumlah Solve** | 583 |

## Ikhtisar

Tantangan ini mengirimkan satu skrip, `invisible_text.py`, dan sebuah paragraf tentang bagaimana manusia bertahan hidup dengan berkomunikasi — menangis, bahasa, Morse, ASL. Kalimat terakhir adalah kalimat yang penting: beberapa pesan tidak disiarkan agar dapat dilihat oleh dunia, dan "Saya yakin Anda akan **melihat** apa yang saya komunikasikan jika Anda **melihat** di tempat yang tepat."

Skrip menyusun kembali daftar potongan base64, mengembangkannya dengan zlib dan mencetak hasilnya, yang oleh dokumennya disebut sebagai entri buku harian yang dipulihkan dalam "format arsip DR3" khusus. Apa yang sebenarnya keluar adalah blok Unicode Braille, lebarnya kira-kira 50 sel dan tinggi 30 baris ganjil.

Braille bukanlah font, melainkan raster: setiap mesin terbang adalah matriks titik 2x4. Jadi blok tersebut berupa bitmap yang lebarnya sedikit di atas seratus piksel, dan dirender menjadi gambar samar dari **siluet profil default** — avatar kepala-dan-bahu anonim yang ditampilkan jejaring sosial ketika tidak ada orang yang mengunggah gambar.

Itu adalah lelucon dari lapisan yang terlihat. Pembaca buku harian menjanjikan buku harian dan memberikan padanan visual dari "tidak ada gambar di sini": pengganti untuk seseorang yang tidak memiliki apa pun untuk ditampilkan. Outputnya tidak pernah menjadi pesan. Flag-nya ada di file yang memproduksinya.

## Penyelesaian

`cat` menampilkan file Python biasa. Lihatlah apa yang terjadi di akhir setiap baris:

```bash
grep -nP '[ \t]+$' invisible_text.py | head
```

Hampir setiap baris memiliki spasi antara bagian akhir konten dan baris baru, dan polanya bergantian: spasi dan tab yang panjang, lalu garis dengan satu tab, lalu jangka panjang lagi. Jangka panjang adalah pembawa; tab tunggalnya bersifat kamuflase, cukup berisik sehingga file selalu terlihat ceroboh dan tidak sengaja ditulis.

Ini adalah steganografi spasi putih klasik. Spasi dan tab tidak terlihat di terminal, tahan terhadap salin-tempel dan sebagian besar perjalanan bolak-balik editor, dan masing-masing menampung sedikit: spasi adalah `0`, tab adalah `1`. Satu proses pembawa adalah satu byte, diisi di sebelah kiri dengan spasi, itulah sebabnya proses tersebut lebih lebar dari delapan bit yang sebenarnya dibutuhkan karakter.

```python
from pathlib import Path

data = Path("invisible_text.py").read_bytes()
out = []

for line in data.splitlines():
    tail = line[len(line.rstrip(b" \t")):]
    if len(tail) < 10:
        continue
    bits = "".join("0" if c == 0x20 else "1" for c in tail)
    out.append(chr(int(bits, 2)))

print("".join(out))
```

Ambang batas hanya melewatkan baris yang terlalu pendek untuk menampung byte empuk; itu bukan bagian dari pengkodean. Outputnya terbuka dengan `POCTF{`, yang merupakan pemeriksaan apakah urutan bit dan sisi padding sudah benar:

```
POCTF{████████████████}
```Judulnya adalah keseluruhan petunjuknya, dan sangat literal. Teks program yang dapat dibaca adalah umpan; *byte* program adalah pesannya. Dua konsekuensi praktis: file harus dibaca sebagai byte, karena setiap perjalanan bolak-balik decode/encode dapat menormalkan atau menghentikan proses, dan editor yang dikonfigurasi untuk memangkas spasi di akhir penyimpanan akan menghancurkan payload tanpa peringatan.