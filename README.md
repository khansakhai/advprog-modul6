# Module 6 Tutorial: Concurrency

Advanced Programming (Even Semester 2024/2025) Tutorial Module 6

Khansa Khairunisa - 2306152462

## Commit 1 Reflection

Dalam implementasi single-threaded web server ini, function `handle_connection` berperan penting untuk mengolah *HTTP request* yang masuk melalui `TcpStream`. Fungsi ini memanfaatkan `BufReader` yang membungkus *mutable* `TcpStream` untuk membaca data secara efisien. Melalui penggunaan metode `.lines()`, sistem dapat memproses data `stream` secara bertahap per baris, lalu mengaplikasikan `.map` untuk melakukan ekstraksi nilai dengan `unwrap()`, serta menerapkan `.take_while` sebagai kondisi penghentian proses pembacaan saat terdeteksi baris kosong yang mengindikasikan berakhirnya *header HTTP request*. Keseluruhan informasi ini lalu diakumulasikan dalam struktur data `Vec<String>` dan diperlihatkan pada konsol, memberikan visibilitas terhadap komponen-komponen request seperti metode HTTP, informasi host, *user-agent*, dan berbagai *header* lainnya. Tahapan ini merupakan bagian esensial dalam proses pengembangan web server untuk menginterpretasikan dan kemudian memberikan respons yang tepat terhadap request dari browser secara sistematis.

## Commit 2 Reflection

Screenshot:

![Commit 2 screen capture](/assets/images/commit2.png)

Fungsi `handle_connection` telah dikembangkan untuk merespons *HTTP request* dengan mengirimkan konten HTML. Implementasi baru ini menggunakan `fs::read_to_string("hello.html")` untuk membaca file HTML dan `contents.len()` untuk menghitung panjang kontennya. Respons HTTP dibentuk dengan `format!()` yang mencakup status line `"HTTP/1.1 200 OK"`, *header* `Content-Length`, yang akan memberi tahu browser ukuran data yang akan diterima agar dapat memprosesnya dengan benar, serta isi file HTML dengan pemisah baris sesuai standar HTTP. Respons tersebut kemudian dikonversi ke bytes menggunakan `as_bytes()` dan dikirim ke browser melalui `stream.write_all()`. Dengan perubahan ini, browser dapat menerima dan menampilkan halaman web ketika mengakses alamat server.