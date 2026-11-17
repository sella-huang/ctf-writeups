# An Invented Language

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 256 |

## Ikhtisar

Arsip berisi `prog.bin` untuk VM berbasis tumpukan yang tidak terdokumentasi dan `vm.c`, penerjemah referensinya.

## Penyelesaian

`vm.c` mendefinisikan set instruksi lengkap: `0x01` push imm32, `0x02` pop, `0x03` dup, `0x04` swap, `0x05` add, `0x06` sub, `0x07` xor, `0x08` dan, `0x09` memutar byte, memuat `0x0a`, menyimpan `0x0b`, mencetak `0x0c`, `0x0d` berhenti.

VM referensi tidak menghasilkan keluaran. Penerjemah Python untuk ISA yang sama menunjukkan bahwa program mendorong pasangan langsung, meng-XOR-nya, dan menyimpan 59 byte ke dalam `m[0..58]`; itu tidak pernah mencapai instruksi cetak. Membuang memori terakhir akan mengungkapkan flag-nya.

Flag yang dikirimkan adalah POCTF{123.1814.████████████████.████████████████} dan diterima.