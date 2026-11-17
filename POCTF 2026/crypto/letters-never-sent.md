# Letters Never Sent

| | |
|---|---|
| **Bagian** | wave 1 |
| **Kategori** | crypto |
| **Jumlah Solve** | 876 |

## Ringkasanan

Tantangannya memberi kita sepucuk surat dan pesan yang diperoleh bersamanya:

```
GMAHI{2.1814.TQPCOJ5THOKCT6HM.FWDJZEANBQV7AO57ATESOZZIYD}
```

Surat itu sebagian besar bersifat atmosferik, tetapi berisi dua fakta berguna. Hal ini ditujukan kepada *Laksamana Sir Francis Beaufort, Ahli Hidrograf Angkatan Laut*, dan badan tersebut memuji "tablo timbal balik yang menyandang nama Anda". Itu adalah sandinya, dinamai menurut nama orang yang sama: **Beaufort**. "Timbal balik" adalah properti yang mendefinisikannya — enkripsi dan dekripsi adalah operasi yang sama:

```
P = (K - C) mod 26
```

Petunjuk lainnya adalah batas surat. Ini adalah bingkai nama tanaman dan objek bergaya Victoria — Mawar, Violet, Tetesan Salju, Jangkar, Tulip, Poppy, Anyelir, Harpa, Willow, Marigold, Merpati, Oak, Narcissus, Pakis, Lily — dan beberapa di antaranya ditandai dengan bintang kecil. Membaca label tersebut searah jarum jam dari yang pertama, inisialnya mengeja:

```
V A C A N T
```

Yang mana kuncinya. Ada dua hal yang membuat pembacaan menjadi jelas: ada enam label berbintang dan enam huruf, dan `A` yang diulang dicocokkan dengan dua entri *Anchor* yang berbintang, satu di tepi atas dan satu di bawah.

Ciphertext itu sendiri menegaskan bahwa angka dan pemisah dibiarkan kosong: blok `2.1814.` tetap tidak tersentuh.

## Penyelesaian

Kuncinya diperiksa terhadap awalan. Dengan `VACANT` dan relasi Beaufort:

| Ciphertext | G | M | A | H | I |
|---|---|---|---|---|---|
| Key | V | A | C | A | N |
| Plaintext | P | O | C | T | F |

`K - C` memberikan `P O C T F` untuk lima huruf pertama, yang merupakan awalan flag dan konfirmasi bahwa kunci dan arah pengurangan keduanya benar. Penggunaan Vigenère biasa (`C - K`) malah menghasilkan `LKNYZ`, jadi hurufnya bukan sekadar hiasan: ia memilih variannya.

Satu detail penyelarasan memerlukan waktu lebih lama dibandingkan sandi itu sendiri. Aliran kunci dijalankan pada string **keseluruhan**, termasuk awalan `GMAHI`/`POCTF`, dan hanya huruf yang mendahuluinya — angka dan tanda baca diteruskan tanpa menggunakan karakter kunci. Setelah lima huruf `GMAHI` kuncinya sudah berada pada huruf keenam, jadi karakter pertama setelah kurung kurawal pembuka sejajar dengan `T`, bukan `V`. Memulai ulang kunci di kurung kurawal menghasilkan sampah dari byte pertama.

```python
ct  = "GMAHI{2.1814.TQPCOJ5THOKCT6HM.FWDJZEANBQV7AO57ATESOZZIYD}"
key = "VACANT"

out, i = [], 0
for ch in ct:
    if ch.isalpha():
        k = ord(key[i % len(key)]) - ord("A")
        out.append(chr((k - (ord(ch) - ord("A"))) % 26 + ord("A")))
        i += 1
    else:
        out.append(ch)

print("".join(out))
```

Menjalankan skrip akan mencetak bendera:

```
POCTF{2.1814.████████████████.██████████████████████████}
```

Kata pertama dari isi, `AFLAME`, adalah pemeriksaan kewarasan yang berguna: kunci yang salah atau penyelarasan satu kali tidak akan pernah menghasilkan kata bahasa Inggris pada posisi tersebut.