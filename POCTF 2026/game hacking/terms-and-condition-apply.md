# Terms and Conditions Apply

| | |
|---|---|
| **Bagian** | wave 7 |
| **Kategori** | game hacking |
| **Jumlah Solve** | 92 |

## Ikhtisar

Game ini merupakan ekspor Godot 4.7.1. PCK yang tertanam berada di bagian `pck` dari ELF dan berisi 200 file, termasuk skrip `.gdc` yang dikompilasi. Skrip penting adalah `NetworkClient.gdc`, `boss.gdc`, `arena.gdc`, dan `heal_zone.gdc`.

Server berwenang untuk negara bos. Sesi baru dimulai dengan:

```text
is_invulnerable = true
avoid_heal_zones = true
hp = 2147483000
```

HP awal bos hanya 648 di bawah `INT32_MAX`. Peristiwa kerusakan diabaikan saat bos kebal, dan kerusakan mantra normal pemain hanya 25, jadi pertarungan biasa tidak bisa mencapai nol. Kondisi kemenangan arena berbeda: ketika status server melaporkan `hp < 0`, `arena.gdc` memanggil `NetworkClient.finish_and_get_flag()`.

Zona penyembuhan menyembuhkan bos yang berdiri di dalamnya sebesar 400 HP per detik, diterapkan dalam tick 0,1 detik, atau sekitar 40 HP per peristiwa `heal_tick`. Karena HP awal tepat di bawah batas 32-bit yang ditandatangani, menyembuhkannya melewati `2^31` akan menghasilkan nilai negatif. Klien juga memiliki kemampuan untuk mengirim acara `flag_flip`, sehingga tanda `avoid_heal_zones` bos dapat dimatikan.

`watchdog.gdc` sisi klien hanya membandingkan properti bos lokal dengan nilai awalnya. Mengirim flip sebagai acara server yang ditandatangani, tanpa mengubah properti lokal, sepenuhnya menghindari pengawas.

## Penyelesaian

Rute terpendek adalah membalik `avoid_heal_zones`, memberi tahu server bahwa bos berada di zona penyembuhan, dan memberikan kutu penyembuhan yang cukup untuk melewati batas 32-bit yang ditandatangani. Pemuatan otomatis `NetworkClient` asli dapat menghasilkan peristiwa bertanda tangan HMAC yang diperlukan:

```gdscript
NetworkClient.send_flip("avoid_heal_zones", false)
NetworkClient.send_zone(true)
for i in range(24):
    NetworkClient.send_heal(40)
```

Server menerima flip dan entri zona; HP bos kemudian naik 40 per tick penyembuhan:

```text
2147483040
2147483080
...
2147483600
2147483640
-2147483616
```

Pada langkah terakhir nilai 32-bit yang ditandatangani dibungkus dengan negatif. Arena melihat `hp < 0` dan meminta penanda finis. Titik akhir finish mengembalikan flag tantangan, yang diterima oleh halaman pengiriman. Flag yang diterima adalah `████████████████`.