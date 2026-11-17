# The Tower of Babel

| | |
|---|---|
| **Bagian** | wave 6 |
| **Kategori** | misc |
| **Jumlah Solve** | 159 |

## Ikhtisar

`babel.txt` adalah lembar referensi ditambah pesan 399 karakter yang diambil hanya dari tujuh huruf `A E I O U X Y`. Lembar ini menjelaskan lima operasi — empat pengkodean basis-N yang dikunci ke bilangan bulat yang diwakilinya, dan satu transposisi kolom dengan lebar 7 — diterapkan dalam urutan yang tidak ditentukan. Hanya rangkaian karakter di setiap tahap yang memberi tahu Anda operasi mana yang menghasilkannya, sehingga pengelupasan dilakukan dengan inspeksi, bukan dengan menebak urutannya.

Pengkodeannya adalah: base 3 (`0`,`1`,`2`), Bitcoin base 58 (58 alfanumerik tanpa `0`, `I`, `O`, `l`), RFC 1924 base 85 (the varian yang alfabetnya mengandung `{`, `}`, `` ` ``, `<`, `>`), and base 7 over `A E I O U Y X` in that digit order. All four are big-integer conversions: the whole string is a numeral, so decoding means evaluating it as one integer and re-emitting the digits in the target base.

## Penyelesaian

The outermost layer announces itself immediately: only seven distinct characters, exactly the base-7 alphabet. Evaluating the message as a base-7 numeral and writing the integer as bytes yields 140 printable ASCII characters.

Those 140 characters use digits, letters, `|`, `` ` ``, `$`, `~`, `;`, `<`, `>` — a mix that fits RFC 1924 base 85 but not Ascii85 (which stops at `u`). Decoding is again a single big-integer conversion, not the 5-char-to-4-byte block form: the result is 112 characters that look like Bitcoin base 58, except for three `0XQ ZKEEP31XQZ0` is the tell. Base 58 deliberately has no `0`, and the transposition is documented to pad the last row with `'0'` before reading columns. So the next layer is the transposition. Writing the 112 characters into a 16 × 7 grid column by column and reading the rows back gives the original string followed by `000`.

Stripping that padding leaves a 109-character base-58 numeral. Decoding it as a big integer yields 80 digits drawn from `0`, `1`, `2` — lapisan terner.

Basis 3 dengan 80 digit adalah sekitar 126 bit, yang diterjemahkan menjadi 16 byte ASCII, dan 16 karakter heksadesimal tersebut adalah nilai nonce yang diminta oleh lembar referensi.

Nonce yang dipulihkan adalah ████████████████.