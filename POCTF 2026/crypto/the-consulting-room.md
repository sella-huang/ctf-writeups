# The Consulting Room

| | |
|---|---|
| **Bagian** | wave 3 |
| **Kategori** | crypto |
| **Jumlah Solve** | 296 |

## Ringkasan

Tantangan ini menampilkan rekam medis terenkripsi dan titik akhir jaminan yang hanya melaporkan apakah rekam medis kandidat memiliki padding PKCS#7 yang valid. Itu adalah padding oracle. Catatannya adalah AES-CBC, jadi Oracle cukup untuk memulihkan blok teks biasa demi blok tanpa kunci.

## Penyelesaian

Halaman ini menyediakan token Pembawa per tim, catatan berkode hex, dan titik akhir:

```text
POST https://pointeroverflowctf.com/challenges/consulting-room/oracle
Authorization: Bearer ████████████████
Content-Type: application/json

{"record": "<hex>"}
```

Padding yang valid mengembalikan `{"padding":"valid"}`; yang tidak valid mengembalikan HTTP 400 dengan `{"padding":"invalid"}`. Rekornya adalah lima blok AES: satu blok IV dan empat blok ciphertext.

Untuk blok ciphertext `C_i`, plaintextnya adalah `P_i = D(C_i) XOR C_{i-1}`. Oracle menguji padding `D(C_i) XOR X`, di mana `X` adalah blok yang kita kendalikan. Dekripsi byte terakhir terlebih dahulu. Untuk nilai padding `p`, atur akhiran yang sudah diketahui ke `I_j XOR p`, di mana `I = D(C_i)`, dan coba semua 256 nilai untuk byte saat ini. Ketika oracle menerima, kandidatnya adalah `I_pos XOR p`. Untuk `p = 1`, positif palsu mungkin terjadi ketika teks biasa sebenarnya sudah berakhir di lebih dari satu byte padding, jadi balikkan byte sebelumnya dan minta Oracle untuk menolak catatan yang dimodifikasi.

```python
import json, urllib.request

URL = "https://pointeroverflowctf.com/challenges/consulting-room/oracle"
TOKEN = "..."  # per-team Bearer token from the page

def oracle(prev, block):
    req = urllib.request.Request(
        URL,
        data=json.dumps({"record": (prev + block).hex()}).encode(),
    )
    req.add_header("Authorization", "Bearer " + TOKEN)
    req.add_header("Content-Type", "application/json")
    try:
        with urllib.request.urlopen(req) as r:
            return json.loads(r.read())["padding"] == "valid"
    except urllib.error.HTTPError as e:
        return json.loads(e.read())["padding"] == "invalid"

def decrypt_block(prev, block):
    inter = [0] * 16
    for pad in range(1, 17):
        pos = 16 - pad
        for guess in range(256):
            probe = bytearray(prev)
            for j in range(pos + 1, 16):
                probe[j] = inter[j] ^ pad
            probe[pos] = guess ^ pad
            if oracle(bytes(probe), block):
                if pad == 1:
                    check = bytearray(probe)
                    check[pos - 1] ^= 0xff
                    if not oracle(bytes(check), block):
                        continue
                inter[pos] = guess
                break
    return bytes(inter[i] ^ prev[i] for i in range(16))
```

Jalankan untuk blok 1 hingga 4, gabungkan, dan hapus padding PKCS#7. Teks biasa yang berukuran 64 byte: flag 58 byte diikuti oleh enam byte padding `0x06`.

```text
████████████████
```

Catatan yang diumumkan adalah flag ████████████████.