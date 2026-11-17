# Commit to Memory

| | |
|---|---|
| **Bagian** | wave 2 |
| **Kategori** | steganography |
| **Jumlah Solve** | 376 |

## Ikhtisar

Bundel git diekspor dari proyek sampingan peneliti. Klonnya adalah riwayat 60-komit linier dengan dua file, `README.md` dan `CHANGELOG.md`. `git grep`, `grep -r`, dan `strings` pada setiap gumpalan tidak menghasilkan apa pun. Flag tidak ada di pohon kerja atau di gumpalan mana pun. Itu ada di stempel waktu penerapan.

## Penyelesaian

`git log --reverse --format='%H %at %s' origin/main` menunjukkan bahwa setiap komit memiliki stempel waktu penulis dan komit yang identik, meningkat secara monoton. Perbedaan subjek dan file adalah sampul. Interval antara stempel waktu penulis berturut-turut dapat dicetak ASCII:

```text
80 79 67 84 70 123 49 48 48 46 ...
 P  O  C  T  F  {  1  0  0  . ...
```

Komit pertama adalah baseline; setiap komit kemudian memajukan jam dengan nilai ASCII dari karakter flag berikutnya. Ada 60 komitmen dan 59 delta, cocok dengan flag 59 karakter.

Ekstraksi minimal:

```python
import subprocess

ts = [int(x) for x in subprocess.check_output(
    ["git", "log", "--reverse", "--format=%at", "origin/main"],
    text=True,
).split()]

print("".join(chr(b - a) for a, b in zip(ts, ts[1:])))
```

Keluaran:

```text
POCTF{100.1814.████████████████████████████████████████}
```

Blok pesan awal yang berulang di log adalah penutup; itu tidak mempengaruhi saluran stempel waktu. Flag yang dipulihkan adalah `POCTF{████…}`.