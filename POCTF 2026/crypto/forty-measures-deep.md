# Forty Measures Deep

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | crypto |
| **Jumlah Solve** | 168 |

## Ringkasanan

Artefaknya adalah file MIDI satu jalur, `score.mid`, dengan satu nada per karakter. Not-not tersebut bukanlah musik acak: nomor MIDI-nya adalah kode ASCII yang digeser dengan jumlah yang tetap. Tantangannya memberikan `POCTF` sebagai teks biasa, yang cukup untuk memulihkan substitusi.

## Penyelesaian

Lagu ini berisi 58 peristiwa catatan, masing-masing sepanjang 480 tick. Membaca nomor not secara berurutan memberikan:

```text
80 79 67 84 70 64 60 57 63 54 61 54 57 63 ...
```

Sebagai ASCII, enam yang pertama adalah `POCTF@`. Mengurangi 5 dari setiap not menghasilkan ciphertext:

```text
KJ>OA;74:1814:THCUU@FOPK2R4NEO:2GDJS3B3TS=6SDS=G772EGO2EDV
```

Teks biasa `POCTF{` cocok dengan `KJ>OA;` dengan substitusi sederhana:

- huruf, `>`, `@`, dan `=` bergeser +5 (`K`→`P`, `J`→`O`, `>`→`C`, ...),
- angka tidak berubah,
- `:` menjadi `.`, `;` menjadi `{`, dan `V` menjadi `}`.

Menerapkan substitusi itu ke seluruh ciphertext:

```python
cipher = "KJ>OA;74:1814:THCUU@FOPK2R4NEO:2GDJS3B3TS=6SDS=G772EGO2EDV"
out = []
for ch in cipher:
    if ch.isdigit():
        out.append(ch)
    elif ch == ':':
        out.append('.')
    elif ch == ';':
        out.append('{')
    elif ch == 'V':
        out.append('}')
    else:
        out.append(chr(ord(ch) + 5))
print(''.join(out))
```

Outputnya memiliki bentuk kanonik `POCTF{74.1814.<nonce>.<sig>}`:

```text
████████████████
```

Skor diterjemahkan ke flag ████████████████.