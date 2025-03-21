# Module 6 Tutorial: Concurrency

Advanced Programming (Even Semester 2024/2025) Tutorial Module 6

Khansa Khairunisa - 2306152462

## Commit 1 Reflection

Dalam implementasi single-threaded web server ini, function `handle_connection` berperan penting untuk mengolah *HTTP request* yang masuk melalui `TcpStream`. Fungsi ini memanfaatkan `BufReader` yang membungkus *mutable* `TcpStream` untuk membaca data secara efisien. Melalui penggunaan metode `.lines()`, sistem dapat memproses data `stream` secara bertahap per baris, lalu mengaplikasikan `.map` untuk melakukan ekstraksi nilai dengan `unwrap()`, serta menerapkan `.take_while` sebagai kondisi penghentian proses pembacaan saat terdeteksi baris kosong yang mengindikasikan berakhirnya *header HTTP request*. Keseluruhan informasi ini lalu diakumulasikan dalam struktur data `Vec<String>` dan diperlihatkan pada konsol, memberikan visibilitas terhadap komponen-komponen request seperti metode HTTP, informasi host, *user-agent*, dan berbagai *header* lainnya. Tahapan ini merupakan bagian esensial dalam proses pengembangan web server untuk menginterpretasikan dan kemudian memberikan respons yang tepat terhadap request dari browser secara sistematis.

## Commit 2 Reflection

Screenshot:

![Commit 2 screen capture](/assets/images/commit2.png)

Fungsi `handle_connection` telah dikembangkan untuk merespons *HTTP request* dengan mengirimkan konten HTML. Implementasi baru ini menggunakan `fs::read_to_string("hello.html")` untuk membaca file HTML dan `contents.len()` untuk menghitung panjang kontennya. Respons HTTP dibentuk dengan `format!()` yang mencakup status line `"HTTP/1.1 200 OK"`, *header* `Content-Length`, yang akan memberi tahu browser ukuran data yang akan diterima agar dapat memprosesnya dengan benar, serta isi file HTML dengan pemisah baris sesuai standar HTTP. Respons tersebut kemudian dikonversi ke bytes menggunakan `as_bytes()` dan dikirim ke browser melalui `stream.write_all()`. Dengan perubahan ini, browser dapat menerima dan menampilkan halaman web ketika mengakses alamat server.

## Commit 3 Reflection

Screenshot:

![Commit 3 screen capture](/assets/images/commit3.png)

Fungsi `handle_connection` dimodifikasi dengan beberapa perubahan penting. Perubahan ini dilakukan untuk menambahkan kemampuan server merespons berbagai jenis URL dengan konten yang sesuai. Pertama, saya mengambil baris pertama request dengan `buf_reader.lines().next().unwrap().unwrap()` untuk mendapatkan `request_line`. Kemudian menggunakan struktur kondisional if-else dengan *pattern matching* untuk memeriksa isi `request_line`, menghasilkan tuple `(status_line, file_path)` yang sesuai. Jika `request_line` adalah `"GET / HTTP/1.1"`, server mengembalikan status `"HTTP/1.1 200 OK"` dan file `hello.html`, sedangkan request lainnya menghasilkan `"HTTP/1.1 404 NOT FOUND"` dan `404.html`, Pembacaan file dilakukan dengan `fs::read_to_string(file_path)` yang mengembalikan konten HTML sesuai kondisi. Respons kemudian diformat menggunakan `format!()` dengan sintaks `{status_line}` dan `{length}` yang lebih ringkas dan dikirim ke browser dengan `stream.write_all()`.

## Commit 4 Reflection

Fungsi `handle_connection` dimodifikasi dengan menambahkan penanganan untuk path `/sleep` yang mensimulasikan permintaan yang membutuhkan waktu lama untuk diproses. Perubahan utama terlihat pada blok *pattern matching* yang memeriksa `request_line` dengan menambahkan kondisi `"GET /sleep HTTP/1.1"`, yang ketika diakses memicu `thread::sleep(Duration::from_secs(10))` untuk menunda respons selama 10 detik sebelum mengirimkan halaman `hello.html` dengan status `"HTTP/1.1 200 OK"`. Simulasi ini menunjukkan kelemahan fundamental server *single-thread*, dimana ketika satu permintaan sedang diproses (terutama yang lambat), semua permintaan lain harus menunggu hingga permintaan tersebut selesai, sehingga ketika beberapa jendela browser mencoba mengakses server secara bersamaan, permintaan kedua akan tetap menunggu sampai permintaan `/sleep` yang pertama selesai. 