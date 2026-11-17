# The Oracle's Weight

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 116 |

## Ikhtisar

Oracle adalah jaringan saraf ONNX yang menerima vektor 8 elemen di `[0,1]` dan mengembalikan skalar mendekati 1 hanya untuk kunci 8 byte tim, dinormalisasi dengan membagi setiap byte dengan 255.

## Penyelesaian

Modelnya adalah MLP tiga lapis, `8 → 64 → 64 → 1`, dengan LeakyReLU (kemiringan 0,1) di kedua lapisan tersembunyi. Saya mengimplementasikan kembali forward pass di NumPy, memperoleh gradien analitik, dan menjalankan pendakian gradien Adam pada input, memotong ke `[0,1]`.

Tiga puluh benih acak dioptimalkan untuk 3000 langkah; hasil berkelanjutan terbaik dibulatkan menjadi byte dan disempurnakan dengan pencarian grid ±1 pada kedelapan komponen. Input terbaik mendapat skor `0.9605`, di atas yang disyaratkan 0,75. Tantangannya meminta 16 karakter hex huruf kecil dari 8 byte yang dihasilkan.

Flag yang dikirimkan adalah `████████████████` dan diterima.