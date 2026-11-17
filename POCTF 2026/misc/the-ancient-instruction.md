# The Ancient Instruction

| | |
|---|---|
| **Bagian** | wave 8 |
| **Kategori** | misc |
| **Jumlah Solve** | 66 |

## Ikhtisar

Artefak tersebut adalah program Brainfuck yang ditulis dengan delapan trigram `☰☱☲☳☴☵☶☷`. Brainfuck memiliki delapan perintah, jadi teka-tekinya adalah memulihkan permutasi yang memetakan trigram ke `><+-.,[]`. Output pertama program ini adalah header pendek yang memberi nama alfabet perintah, diikuti dengan bendera.

## Penyelesaian

Indeks trigram dalam urutan `☰☱☲☳☴☵☶☷`, ubah artefak menjadi aliran instruksi, dan coba 8! kemungkinan permutasi perintah. Pemetaan yang benar adalah:

```text
☰ ☱ ☲ ☳ ☴ ☵ ☶ ☷
] < > - , . + [
```

Seorang penerjemah kecil sudah cukup:

```python
symbols = "☰☱☲☳☴☵☶☷"
index = {c: i for i, c in enumerate(symbols)}
command = "]<>-,.+["

code = [command[index[c]] for c in open("program.txt", encoding="utf-8").read()
        if c in index]
```

Jalankan program Brainfuck yang dihasilkan. Ini mencetak alfabet perintah `+-><.,[]` sebagai pemeriksaan mandiri dan kemudian bendera. Flag yang diterima adalah `████████████████`.