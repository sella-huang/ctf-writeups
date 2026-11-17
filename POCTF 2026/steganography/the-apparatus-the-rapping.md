# The Apparatus, The Rapping

| | |
|---|---|
| **Bagian** | wave 3 |
| **Kategori** | steganography |
| **Jumlah Solve** | 311 |

## Ikhtisar

Artefaknya adalah WAV mono 16-bit 20,16 detik, 22050 Hz. Rekaman sebagian besar berupa kebisingan latar belakang yang diinterupsi oleh 65 semburan nada ~175 Hz. Semburannya adalah "rap"; durasinya dan jarak di antara keduanya membentuk kode Morse.

## Penyelesaian

Amplop RMS 10 ms dengan ambang batas 1000 mengisolasi semburan nada dari latar belakang (RMS latar belakang sekitar 150). Durasi burst terbagi dalam dua kelompok:

```text
short ~0.085 s  -> dot
long  ~0.240 s  -> dash
```

Kesenjangan tersebut terbagi dalam dua kelompok:

```text
short ~0.080 s  -> intra-character gap
long  ~0.240 s  -> inter-character gap
```

Tidak ada kesenjangan kata 7 unit. Mengelompokkan 65 semburan pada celah yang panjang dan memetakan durasinya menghasilkan 16 karakter Morse:

```text
.- ----. -.-. ..-. ..... ----- ...-- . -.-. -.-. --... . ...-- ----. ----- --...
```

Menguraikan tabel Morse standar menghasilkan:

```text
████████████████
```

Detektor minimal:

```python
import wave
import numpy as np

w = wave.open("seance-recording-1814.wav", "rb")
sr = w.getframerate()
x = np.frombuffer(w.readframes(w.getnframes()), dtype="<i2").astype(float)

win, hop = int(sr * 0.01), int(sr * 0.005)
rms = np.array([
    np.sqrt(np.mean(x[i:i + win] ** 2))
    for i in range(0, len(x) - win, hop)
])
on = rms > 1000

runs = []
start = None
for i, v in enumerate(on):
    if v and start is None:
        start = i
    elif not v and start is not None:
        runs.append((start * hop / sr, i * hop / sr))
        start = None

chars = []
cur = []
for i, (a, b) in enumerate(runs):
    cur.append((a, b))
    if i == len(runs) - 1 or runs[i + 1][0] - b > 0.15:
        chars.append(cur)
        cur = []

morse = {
    ".-": "A", "-...": "B", "-.-.": "C", "-..": "D", ".": "E",
    "..-.": "F", "--.": "G", "....": "H", "..": "I", ".---": "J",
    "-.-": "K", ".-..": "L", "--": "M", "-.": "N", "---": "O",
    ".--.": "P", "--.-": "Q", ".-.": "R", "...": "S", "-": "T",
    "..-": "U", "...-": "V", ".--": "W", "-..-": "X", "-.--": "Y",
    "--..": "Z", "-----": "0", ".----": "1", "..---": "2",
    "...--": "3", "....-": "4", ".....": "5", "-....": "6",
    "--...": "7", "---..": "8", "----.": "9",
}

out = ""
for ch in chars:
    code = "".join("-" if b - a > 0.15 else "." for a, b in ch)
    out += morse[code]

print(out)
```

Keluaran:

```text
████████████████
```

Nonce yang dipulihkan adalah `████████████████`.