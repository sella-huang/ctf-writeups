# The Accused

| | |
|---|---|
| **Bagian** | wave 8 |
| **Kategori** | forensics |
| **Jumlah Solve** | 67 |

## Ikhtisar

Tantangannya menyediakan kotak keluar Marion Halberd, sebuah tarball yang berisi enam log sesi peralatan dan sebuah fragmen passwd, dan tangkapan telnet dari upaya penghapusan log. Nama file sesi langsung membawa tag subjek, tetapi rotasi log menghapus tag tersebut sebelum arsip ditulis. Oleh karena itu, log yang masih ada hanya mengidentifikasi kode operator dan kode subjek. Halaman tantangan secara eksplisit memperingatkan bahwa ini adalah satu-satunya tuduhan tim: token yang benar dipasangkan dengan terdakwa yang salah menutup kasus secara permanen, memberikan poin nol, kehilangan Good Gumshoe, dan tidak mengizinkan percobaan ulang. README menambahkan bahwa pencapaian penyelesaian kasus bergantung pada tuduhannya, bukan pada perolehan tanda pada percobaan ulang berikutnya. Tugasnya adalah mengidentifikasi sesi Elena Marchetti dan operator yang menjalankannya.

## Penyelesaian

Tarball berisi enam log. Hanya `session_QBRM7DGJ3WVZ3ZEX.log.1` yang memiliki `frame=IV`, `subject-selection-protocol=BYPASSED`, `safety-interlocks=OVERRIDE (operator)`, dan `subject=SBJ_14`; ini berlangsung dari 22:14 hingga 23:03 pada 28-06-2026. Lima sesi lainnya adalah sesi Frame I/STANDARD. Log 25/06 secara eksplisit diidentifikasi di kotak keluar sebagai subjek Weiss, SBJ_17, jadi ini adalah tindakan yang disengaja.

Email kotak keluar dari Halberd pada pukul 23:07 tanggal 28/06 mengatakan "masalah Marchetti" telah dicatat "di bawah katalog Frame IV dengan sebutan standar." Karena QBRM adalah satu-satunya log Frame IV, `SBJ_14` adalah Elena Marchetti.

Fragmen passwd memetakan kode operator QBRM ke sebuah nama: `APP_OPR_7` adalah `Marion Halberd`. Tangkapan telnet menguatkan hal ini: loginnya adalah `halberd`, dan sesi mencoba `rm session_*_marchetti.log*` dan `rm *marchetti*`. Keduanya gagal karena rotasi menghapus tag subjek, dan komentar penyerang berbunyi `# rotator stripped the tag. nothing to grep.` `ls` tangkapan juga mencantumkan QBRM di direktori langsung, menjadikannya satu-satunya ID sesi yang umum untuk tangkapan dan tarball yang bertahan; halaman menyatakan token yang diperlukan adalah `session_id` di dalam tarball.

Oleh karena itu, Marion Halberd adalah operator sesi Elena dan orang yang mencoba menghancurkan lognya. Jawaban yang diterima adalah token sesi `████████████████` dengan tuduhan `marion_halberd` (Prof. Marion Halberd).