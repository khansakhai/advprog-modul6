# Module 6 Tutorial: Concurrency

Advanced Programming (Even Semester 2024/2025) Tutorial Module 6

Khansa Khairunisa - 2306152462

## Commit 1 Reflection

Dalam implementasi single-threaded web server ini, function `handle_connection` berperan penting untuk mengolah *HTTP request* yang masuk melalui `TcpStream`. Fungsi ini memanfaatkan `BufReader` yang membungkus *mutable* `TcpStream` untuk membaca data secara efisien. Melalui penggunaan metode `.lines()`, sistem dapat memproses data `stream` secara bertahap per baris, lalu mengaplikasikan `.map` untuk melakukan ekstraksi nilai dengan `unwrap()`, serta menerapkan `.take_while` sebagai kondisi penghentian proses pembacaan saat terdeteksi baris kosong yang mengindikasikan berakhirnya *header HTTP request*. Keseluruhan informasi ini lalu diakumulasikan dalam struktur data `Vec<String>` dan diperlihatkan pada konsol, memberikan visibilitas terhadap komponen-komponen request seperti metode HTTP, informasi host, *user-agent*, dan berbagai *header* lainnya. Tahapan ini merupakan bagian esensial dalam proses pengembangan web server untuk menginterpretasikan dan kemudian memberikan respons yang tepat terhadap request dari browser secara sistematis.