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

## Commit 5 Reflection

`main.rs` dimodifikasi dengan menerapkan model *multithreaded* server menggunakan ThreadPool, sedangkan `lib.rs` dibuat baru untuk mendukung fungsionalitas ini. Pada `main.rs`, perubahan utama terjadi dengan penambahan `use hello::ThreadPool` untuk mengimpor struktur ThreadPool dari *library* yang dibuat, kemudian inisialisasi *thread pool* dengan `Let pool = ThreadPool::new(4)` yang menciptakan 4 *worker thread*. Selanjutnya, metode `pool.execute()` digunakan untuk mendelegasikan penanganan koneksi ke *thread* yang tersedia dari *pool* dengan *closure* `|| { handle_connection(stream); }`. Sementara itu, `lib.rs` yang baru dibuat berfungsi sebagai modul terpisah yang mendefinisikan implementasi ThreadPool dengan komponen-komponen penting seperti struktur Worker, tipe Job, dan kanal MPSC (*multiple producer, single consumer*) untuk komunikasi antar *thread*. Perubahan ini dilakukan untuk mengatasi keterbatasan server *single-thread* yang tidak dapat menangani permintaan secara bersamaan, sehingga server dapat memproses beberapa koneksi secara paralel tanpa harus menunggu permintaan yang membutuhkan waktu lama (seperti `/sleep`) selesai diproses, meningkatkan responsivitas dan *throughput* server secara signifikan.


## Commit Bonus Reflection

Saya melakukan perubahan pada implementasi ThreadPool dengan mengganti function `new` menjadi function `build` yang lebih aman. Tujuan dari perubahan ini adalah untuk menghindari *panic* saat *size* bernilai nol atau angka negatif, dan sebagai gantinya mengembalikan *result* yang lebih sesuai dengan cara kerja Rust. Caranya dengan mengganti `assert!(size > 0)` menjadi pengecekan kondisional `if size <= 0` yang mengembalikan `Err("The size of ThreadPool must be a positive integer!")` ketika ukuran *thread pool* tidak *valid*. Sementara itu, di `main.rs` saya menambahkan *import* `process` dan memodifikasi inisialisasi *thread pool* menggunakan `ThreadPool::build(4).unwrap_or_else()` dengan *handler error* yang lebih informatif, yang mencetak pesan *error* dan keluar dari program dengan kode status 1 jika terjadi kegagalan. Perubahan ini membuat kode lebih *robust* karena *error* ditangani dengan cara yang lebih eksplisit dan terkendali, serta memberikan pesan yang lebih jelas kepada pengguna.