# The Apparatus, Invocation

| | |
|---|---|
| **Bagian** | wave 1 |
| **Kategori** | misc |
| **Jumlah Solve** | 633 |

## Ikhtisar

Tantangannya adalah kotak lilin berukuran 7x7: empat puluh sembilan sel, masing-masing menyala atau gelap. Pers akan mengaktifkan candle yang ditekan dan empat tetangga ortogonalnya; tujuannya adalah untuk menggelapkan setiap lilin. Catatan tersebut membuat aljabar menjadi eksplisit: urutan penekanan apa pun diperbolehkan, urutannya tidak penting, hanya set yang penting, dan pengaturan ulang akan memulihkan pola awal tim. Menekan sel yang sama dua kali akan membatalkan, sehingga setiap sel ditekan paling banyak satu kali. Ini adalah varian Lights Out, dan diselesaikan sebagai sistem linier melalui GF(2), bukan dengan pencarian.

Halaman ini juga memperlihatkan titik akhir `complete` yang menerima serangkaian penekanan dan mengembalikan tanda. Pemecah di bawah membaca papan dari tangkapan layar, menyelesaikan sistem, dan memposting klik yang dihasilkan ke titik akhir tersebut.

## Penyelesaian

Bacalah papan baris-mayor, dengan `1` untuk lilin yang menyala dan `0` untuk lilin yang gelap. Papan yang ditangkap adalah:

```
1 1 1 0 0 1 1
1 0 1 1 1 1 1
1 1 0 1 0 0 0
1 0 0 1 0 1 1
0 1 1 0 0 1 1
1 1 0 0 1 0 1
1 0 0 1 1 0 1
```

Diratakan, itulah vektor 49-bit `b`. Misalkan `x` adalah vektor 49-bit dimana `x[i] = 1` berarti sel `i` ditekan. Biarkan `A` menjadi matriks 49×49 dengan `A[i][j] = 1` saat menekan `j` akan mengaktifkan sel `i`, yaitu ketika `j == i` atau `j` adalah tetangga ortogonal dari `i`. Setiap pers adalah XOR dari tanda silang ke papan, jadi keadaan akhirnya adalah `b XOR (A x)`. Tujuannya adalah keadaan serba nol:

```
A x = b   over GF(2)
```

Dua kali penekanan pada sel yang sama dibatalkan, tepatnya `1 + 1 = 0` di GF(2); itulah mengapa hanya set yang penting dan mengapa kekerasan terhadap `2^49` tidak diperlukan. Eliminasi Gaussian melalui GF(2) — eliminasi biasa dengan XOR alih-alih pengurangan — menyelesaikan sistem 49×49 dalam operasi 49³ dan bersifat eksak. Matriks 7×7 Lights Out memiliki peringkat penuh 49 di atas GF(2), sehingga solusinya unik untuk setiap papan.

Solver membangun `A` dari aturan sakelar, menambahnya dengan `b`, menguranginya dengan Gauss-Jordan, dan membaca kembali `x`. Kode tersebut adalah payload yang dijalankan di konsol browser pada halaman tantangan:

```javascript
(async function () {
  const N = 7;
  const size = N * N;
  const initialBits = [
    1,1,1,0,0,1,1,
    1,0,1,1,1,1,1,
    1,1,0,1,0,0,0,
    1,0,0,1,0,1,1,
    0,1,1,0,0,1,1,
    1,1,0,0,1,0,1,
    1,0,0,1,1,0,1,
  ];

  // A[i][j] = 1 iff pressing j toggles cell i: self plus orthogonal neighbours.
  const A = Array.from({ length: size }, () => new Array(size).fill(0));
  for (let r = 0; r < N; r++) {
    for (let c = 0; c < N; c++) {
      const i = r * N + c;
      A[i][i] = 1;
      for (const [dr, dc] of [[-1,0],[1,0],[0,-1],[0,1]]) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < N && nc >= 0 && nc < N) {
          A[i][nr * N + nc] = 1;
        }
      }
    }
  }

  // Gauss-Jordan on [A | b] over GF(2).
  const M = A.map((row, i) => [...row, initialBits[i]]);
  let rank = 0;
  for (let col = 0; col < size; col++) {
    let pivot = -1;
    for (let r = rank; r < size; r++) {
      if (M[r][col] === 1) { pivot = r; break; }
    }
    if (pivot === -1) continue;
    [M[rank], M[pivot]] = [M[pivot], M[rank]];
    for (let r = 0; r < size; r++) {
      if (r !== rank && M[r][col] === 1) {
        for (let c = col; c <= size; c++) M[r][c] ^= M[rank][c];
      }
    }
    rank++;
  }

  // Back-substitute; free variables (if any) stay 0.
  const x = new Array(size).fill(0);
  for (let r = rank - 1; r >= 0; r--) {
    let first = -1;
    for (let c = 0; c < size; c++) {
      if (M[r][c] === 1) { first = c; break; }
    }
    if (first === -1) continue;
    let val = M[r][size];
    for (let c = first + 1; c < size; c++) {
      if (M[r][c] === 1) val ^= x[c];
    }
    x[first] = val;
  }

  const clicks = [];
  for (let i = 0; i < size; i++) {
    if (x[i] === 1) clicks.push({ r: Math.floor(i / N), c: i % N });
  }

  console.log('clicks', clicks);
  const res = await fetch(window.location.pathname + 'complete', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ clicks }),
  });
  const data = await res.json();
  console.log(data);
  if (data.flag) alert('FLAG: ' + data.flag);
})();
```

Untuk papan yang ditangkap, eliminasi menghasilkan 15 penekanan:

```
(0,0) (0,3) (0,4) (1,6) (2,2) (3,0) (3,2) (3,6)
(5,0) (5,1) (5,5) (6,0) (6,1) (6,2) (6,5)
```Menerapkan persilangan tersebut ke `b` menghasilkan nol di setiap sel. Skrip kemudian memposting `{"clicks":[{"r":0,"c":0}, ...]}` ke `window.location.pathname + 'complete'` dan server mengembalikan `{flag}`. Jalur halaman diakhiri dengan `/`, sehingga titik akhir relatif menentukan rute tantangan `complete`; menjalankan skrip dari asal lain akan kehilangan sesi tim dan permintaan gagal.

Jalan buntu instruktif sebagian besar merupakan kesalahan pemodelan. Brute force tidak ada: `2^49` tidak dapat dicari, dan struktur linier membuatnya tidak diperlukan. Memasukkan diagonal, atau menghilangkan sel yang ditekan, mengubah `A` dan menghasilkan himpunan yang membuat papan salah. Membalikkan konvensi bit (`0` untuk menyala, bukan `1`) menyelesaikan masalah komplemen dan menyalakan setiap candle, bukan mematikannya. Membaca gambar kolom-utama alih-alih baris-utama mengubah urutan `b` relatif terhadap `A` dan menghasilkan kumpulan klik yang tampak masuk akal tetapi tidak valid. Mengirim array datar atau koordinat 1 indeks tidak cocok dengan skema `{clicks:[{r,c}]}`. Perintah menekan tidak relevan, tetapi duplikat dibatalkan, sehingga kumpulan harus berisi setiap sel satu kali.

Peralatan tersebut mencetak namanya:

```
POCTF{████…}
```