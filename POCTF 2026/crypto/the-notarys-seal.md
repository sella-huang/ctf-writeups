# The Notary's Seal

| | |
|---|---|
| **Bagian** | wave 7 |
| **Kategori** | crypto |
| **Jumlah Solve** | 150 |

## Ringkasan

Notaris menyegel suatu akta dengan menghitung `c = m^e mod n` dengan `e = 3`. Modulusnya 2048 bit, tapi sealnya hanya 1387 bit, jadi `m^3 < n`. Reduksi modular tidak pernah terjadi: segelnya adalah kubus bilangan bulat eksak. Dokumen tersebut dilengkapi dengan akar pangkat tiga bilangan bulat, tidak memerlukan pemfaktoran.

## Penyelesaian

Halaman ini memberikan modulus 2048-bit `n`, eksponen `e = 3`, dan segel

```text
c = 0x7e74fc33f9eec5d02a253749eeb1dbea50aa45a16c453324712b2419634a8dca31deeea213747c5c99d055a9e68e191a2e11450c7080fe4532f7a7f690dc0dd388dfd87ca6c817662158d4bffdf519e213cca8491e4fdfd7183225cd26dd3d6d2d8dacce7008b5c9a25dbdb1f1309cb85cf7be387cfd7c2a191fb3ea1951b95ace6b704718fd4fee753830d0fd76c03bada5f91d03334a1c42db26026d6d519614ce98f176c2fffcbd43a33df65
```

Karena `c < n` dan `c` adalah kubus sempurna, hitung akar pangkat tiga bilangan bulatnya dengan pencarian biner:

```python
def icbrt(x):
    lo, hi = 0, 1 << ((x.bit_length() + 2) // 3)
    while lo < hi:
        mid = (lo + hi + 1) // 2
        if mid**3 <= x:
            lo = mid
        else:
            hi = mid - 1
    return lo

m = icbrt(c)
assert m**3 == c
print(m.to_bytes((m.bit_length() + 7) // 8, 'big').decode())
```

Rootnya adalah 463 bit, atau 58 byte. Ini diterjemahkan ke dalam bendera:

```text
████████████████
```

Dokumen yang disegel adalah flag ████████████████.