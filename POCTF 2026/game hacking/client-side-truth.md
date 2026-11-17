# Client-Side Truth

| | |
|---|---|
| **Bagian** | wave 8 |
| **Kategori** | game hacking |
| **Jumlah Solve** | 53 |

## Ikhtisar

Game ini adalah klien Godot. Skripnya dikemas ke dalam PCK yang tertanam di ELF; bagian `pck` dimulai pada file offset `0x4611138`. Header PCK memberikan offset file-tabel, tabel mencantumkan 128 file, dan setiap entri menyediakan offset data, ukuran, dan MD5. Mengekstrak dan mendekompilasi skrip `.gdc` mengungkapkan protokol jaringan di `NetworkClient.gdc`:

```text
POST /challenges/client-side-truth/game/start
POST /challenges/client-side-truth/game/pos   {"ts":..., "x":..., "y":..., "z":...}
POST /challenges/client-side-truth/game/fire  {"ts":...}
GET  /challenges/client-side-truth/game/state
GET  /challenges/client-side-truth/game/win_marker
```

Klien mengautentikasi dengan `Authorization: Bearer <game.token>` dan `X-Team-ID: <team_id>`. Bos memulai dengan 500 HP dan setiap tembakan menghasilkan 50 kerusakan. Kerusakan hanya berlaku dari dalam area `count_room`, namun posisi disediakan oleh klien dan tidak divalidasi sisi server.

## Penyelesaian

Mulailah sesi permainan, lalu atur posisi di dalam ruang hitung. Posisi kerja adalah `[-20, 21.8, 18.1]`. Tembak sepuluh kali; setiap panggilan mengurangi 50 HP, mengurangi bos menjadi nol. Kueri status untuk melihat `"won": true`, lalu ambil penanda kemenangan:

```python
import json, time, urllib.request

BASE = "https://pointeroverflowctf.com"
TOKEN = "<game.token>"
TEAM = "<team_id>"
H = {
    "Authorization": "Bearer " + TOKEN,
    "X-Team-ID": TEAM,
    "Content-Type": "application/json",
}

def req(method, path, body=None):
    data = json.dumps(body).encode() if body is not None else None
    r = urllib.request.Request(BASE + path, data=data, headers=H, method=method)
    with urllib.request.urlopen(r, timeout=15) as resp:
        return resp.read().decode()

req("POST", "/challenges/client-side-truth/game/start", {})
req("POST", "/challenges/client-side-truth/game/pos",
    {"ts": int(time.time() * 1000), "x": -20, "y": 21.8, "z": 18.1})
for _ in range(10):
    req("POST", "/challenges/client-side-truth/game/fire",
        {"ts": int(time.time() * 1000)})
print(req("GET", "/challenges/client-side-truth/game/win_marker"))
```

Penanda kemenangan berisi bendera. Flag yang diterima adalah `████████████████`.