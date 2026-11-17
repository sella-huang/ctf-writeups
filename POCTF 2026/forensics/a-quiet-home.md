# A Quiet Home

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | forensics |
| **Jumlah Solve** | 281 |

## Ikhtisar

Kumpulan artefak adalah cadangan Home Assistant (`home-assistant_v2.db`, `configuration.yaml`), log broker Mosquitto, dan pengambilan paket 1.857 byte. Narasinya menempatkan pelacak perangkat Elena di Hollis, membuka kunci pintu depan pada 14:47 dengan slot keypad 04, menunjukkan gerakan di ruang kerja hingga 15:34, dan mengunci pintu dari dalam pada 15:36. Tokennya adalah serial base32 16 karakter dari perangkat yang tidak boleh ada di apartemen.

Catatan penangkapan dan perantara dijalankan tiga jam lebih awal dari jam narasi — 17:47, bukan 14:47 — namun urutan relatifnya tepat. Masalahnya dikurangi menjadi menghubungkan tiga sumber: tangkapan MQTT, yang berisi lalu lintas perangkat; log broker, yang menunjukkan klien mana yang terhubung; dan database Home Assistant, yang menunjukkan perangkat mana yang diketahui sistem. Perangkat yang muncul dalam tangkapan dan log broker tetapi tidak memiliki tempat normal dalam registri perangkat adalah perangkat yang dibawa oleh penyusup.

## Penyelesaian

Pengambilannya adalah pcap Ethernet/IPv4/TCP klasik (`linktype 1`) yang berisi paket MQTT PUBLISH. Tidak ada jabat tangan TCP, sehingga stream reassembler tidak dapat merekonstruksi sesi; penguraian muatan TCP secara langsung bersifat deterministik. `strings` cukup untuk pass pertama:

```bash
strings -n 3 mqtt-capture.pcap | grep -iE 'rf_key|serial|Aeotec|MiniMote|front_door|keypad'
```

Untuk alamat sumber dan stempel waktu yang tepat, telusuri catatan pcap dan cetak setiap muatan TCP:

```python
import struct
import datetime

b = open("mqtt-capture.pcap", "rb").read()
o = 24

while o + 16 <= len(b):
    ts, us, n, _ = struct.unpack_from("<IIII", b, o)
    o += 16
    p = b[o:o+n]
    o += n

    if len(p) < 14 or struct.unpack_from("!H", p, 12)[0] != 0x0800:
        continue

    ip = p[14:]
    ihl = (ip[0] & 0x0f) * 4
    if ip[9] != 6:
        continue

    tcp = ip[ihl:]
    thl = (tcp[12] >> 4) * 4
    data = tcp[thl:]

    if data:
        t = datetime.datetime.fromtimestamp(ts + us / 1e6, datetime.timezone.utc)
        print(t.isoformat(), data)
```

Paket yang relevan, didekodekan dari framing MQTT-nya:

| time (UTC) | source | topic | payload |
|---|---|---|---|
| `17:47:00` | `192.168.1.11:44100` | `zwave/lock.front_door/state` | `{"state": "unlocked", "code_slot": 4, "changed_by": "keypad_04"}` |
| `17:47:30` | `192.168.1.47:55555` | `zwave/rf_key_c4pf5/attributes` | `{"node_id": 47, "product_id": "0x0037", "product_type": "0x0217", "product_serial": "████████████████", "manufacturer": "Aeotec", "model": "MiniMote Key Fob (ZW088)"}` |
| `18:36:00` | `192.168.1.11:44100` | `zwave/lock.front_door/state` | `{"state": "locked", "code_slot": 4, "changed_by": "keypad_04"}` |

Setiap paket MQTT dimulai dengan `0x30` (PUBLISH), diikuti dengan kolom panjang sisa panjang variabel, panjang topik dua byte, topik, dan payload JSON. Pesan key fob tiba tepat 30 detik setelah pembukaan kunci, dari host yang tidak muncul di kumpulan klien MQTT normal, dan sebelum pintu dikunci kembali. Paket gerak ruang tamu dan ruang belajar adalah lalu lintas Zigbee normal; key fob adalah satu-satunya perangkat baru di jendela.

Basis data Asisten Rumah adalah SQLite. Buka hanya baca dan tanyakan tabel `devices`:

```python
import sqlite3

con = sqlite3.connect("file:home-assistant_v2.db?mode=ro", uri=True)

for row in con.execute(
    "select name, manufacturer, model, identifiers from devices order by name"
):
    print(row)
```

Baris yang mencurigakan adalah:

```text
('RF Key Fob (unknown pairing)', 'Aeotec', 'MiniMote Key Fob (ZW088)',
 '[["zwave", "████████████████"]]')
```Basis data berisi 15 perangkat dan 14 entitas. Key fob adalah satu-satunya perangkat bernama `unknown pairing`, dan merupakan satu-satunya perangkat yang tidak memiliki baris entitas terkait. 14 perangkat lainnya diberi nama, perangkat berpasangan dengan entitas: kunci pintu depan Schlage, sensor gerak Aeotec, lampu Hue, termostat Ecobee, Roomba, dan iPhone Elena. Z-Wave JS membuat baris registri perangkat saat fob berada dalam jangkauan radio, tetapi tidak pernah membuat entitas untuknya. Itu adalah tanda tangan registri dari perangkat yang tidak berpasangan yang diamati dan kemudian menghilang.

Log broker mengonfirmasi masa hidup klien:

```bash
grep -nE 'zwave_c4pf5|192\.168\.1\.47' mosquitto.log
```

```text
124:1719942450: New client connected from 192.168.1.47 as zwave_c4pf5 (p2, c1, k60).
125:1719942451: Received PUBLISH from zwave_c4pf5 (d0, q0, r0, m0, 'zwave/rf_key_c4pf5/attributes', ... (256 bytes))
134:1719945420: Client zwave_c4pf5 disconnected.
```

Nilai epoch dikonversi ke `17:47:30`, `17:47:31`, dan `18:37:00` UTC. Klien fob hanya ada untuk jendela intrusi 49,5 menit: klien terhubung 30 detik setelah pintu terbuka dan terputus setelah pintu terkunci di `18:36:00`. Sumber `192.168.1.47` bukan salah satu klien normal:

| client | source | role |
|---|---|---|
| `homeassistant` | `192.168.1.10` | Home Assistant |
| `zigbee2mqtt` | `192.168.1.11` | Zigbee gateway |
| `zwavejs2mqtt` | `192.168.1.12` | Z-Wave gateway |
| `front_door_lock` | `192.168.1.15` | front-door lock |
| `roomba_j7` | `192.168.1.20` | robot vacuum |
| `zwave_c4pf5` | `192.168.1.47` | only during the intrusion |

Host `.47` hanya muncul seumur hidup fob. Oktet terakhirnya cocok dengan `node_id: 47` fob, dan ID klien `zwave_c4pf5` adalah identitas broker sementara, bukan serial. Log memberikan klien dan topik; pcap memasok serial; database mengonfirmasi serial yang sama di registri perangkat. Ketiga sumber tersebut sepakat.

Fobnya adalah Aeotec MiniMote Key Fob (ZW088), remote Z-Wave portabel. Itu bukan metode aksesnya: pintu dibuka dengan slot keypad 04, ditugaskan ke M. Halberd, bukan dengan fob. Fob adalah benda yang dibawa penyusup. Radionya terdeteksi oleh jaringan Z-Wave Elena ketika berada dalam jangkauan, ia menerbitkan atribut produknya 30 detik setelah pintu terbuka, dan meninggalkan broker ketika penyusup mengunci pintu dan pergi. Perangkat yang "tidak boleh ada" adalah fob, bukan kunci, sensor gerak, atau telepon Elena.Beberapa detail merupakan jalan buntu yang disengaja. Basis data berisi banyak string base32 16 karakter — `XKHPH6UB6KQYW2PM` adalah kunci pintu depan, `C4OUN5RVMLW522LU` adalah iPhone Elena, dan seterusnya — tetapi itu milik perangkat yang diberi nama dan dipasangkan. Key fob adalah satu-satunya pasangan yang tidak diketahui dan satu-satunya perangkat tanpa entitas. Tabel `states` dan `entities` berisik dan tidak berisi fob; menanyakannya saja akan melewatkan baris registri perangkat. `(256 bytes)` log broker adalah jumlah framing, bukan serial; muatan TCP pcap adalah JSON otoritatif. Pcap tidak memiliki jabat tangan TCP, jadi disektor MQTT normal mungkin tidak memasang kembali aliran; penguraian muatan langsung sudah cukup. Jam artefak juga digeser: pcap dan log bertuliskan `2024-07-02`, sedangkan narasinya mengatakan Kamis `06/25/2026`, dan waktunya berbeda tiga jam. Urutan relatif — buka kunci, fob, gerak, kunci — adalah yang terpenting.

Serial fob terdiri dari 16 karakter base32 dan diterima mentah atau dibungkus oleh halaman artefak. Perangkat tersebut adalah Aeotec MiniMote Key Fob (ZW088), dan tokennya adalah `product_serial`-nya. Seri key fob yang tidak diketahui adalah `████████████████`, dan flag-nya adalah `POCTF{████…}`.