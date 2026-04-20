## Commit 1 Reflection

Kode pada commit pertama merupakan implementasi sederhana dari sebuah server HTTP menggunakan Rust. Program ini bekerja dengan membuat server TCP yang berjalan pada alamat lokal 127.0.0.1 di port 7878. Setiap kali ada koneksi yang masuk, server akan menerima data tersebut dan memprosesnya melalui sebuah fungsi khusus.

Di dalam fungsi tersebut, koneksi yang diterima diperlakukan sebagai stream yang bisa dimodifikasi. Untuk membaca data dengan lebih efisien, digunakan mekanisme buffering sehingga data dapat diproses baris demi baris. Setiap baris yang diterima kemudian diambil hingga mencapai baris kosong, yang menandakan akhir dari header HTTP. Seluruh baris ini kemudian dikumpulkan menjadi sebuah struktur data berbentuk list.

Setelah semua data permintaan HTTP berhasil dibaca, informasi tersebut ditampilkan ke console. Dari hasil yang diperoleh, terlihat bahwa server menerima request HTTP standar yang berisi metode GET untuk mengakses halaman utama. Selain itu, terdapat berbagai header yang menjelaskan detail permintaan, seperti alamat tujuan, jenis koneksi, preferensi cache, informasi browser yang digunakan, tipe konten yang dapat diterima, serta preferensi encoding dan bahasa. Header-header ini memberikan gambaran lengkap mengenai bagaimana client (browser) berkomunikasi dengan server.

## Commit 2 Reflection
![alt text](/2.png)
Pada commit kedua, fungsi handle_connection telah diperbarui sehingga tidak lagi hanya membaca HTTP request, tetapi juga mengirimkan respons kembali ke klien. Perubahan ini menambahkan kemampuan untuk menyusun respons HTTP secara lengkap, dimulai dari status line yang menunjukkan keberhasilan permintaan, kemudian membaca isi file HTML dari sistem, menghitung panjang konten, dan menggabungkan seluruh komponen tersebut menjadi satu respons HTTP yang valid. Respons yang sudah terbentuk kemudian dikirimkan ke klien melalui koneksi TCP, sehingga server kini berfungsi tidak hanya sebagai penerima request, tetapi juga sebagai pemberi respons berupa halaman HTML.

Saat program dijalankan, proses kompilasi berhasil dilakukan, namun muncul sebuah peringatan. Peringatan tersebut menunjukkan bahwa terdapat variabel yang dideklarasikan tetapi tidak digunakan dalam kode. Compiler memberikan saran untuk menambahkan awalan underscore jika variabel tersebut memang sengaja tidak dipakai. Meskipun demikian, peringatan ini tidak menghalangi proses kompilasi maupun eksekusi, sehingga program tetap dapat berjalan dengan normal dalam mode pengembangan.

## Commit 3 Reflection
![alt text](/3.png)
Pada commit ketiga, fungsi handle_connection kembali diperbarui agar server dapat membedakan jenis request yang diterima. Server kini hanya mengambil baris pertama dari HTTP request, lalu mengecek apakah permintaan tersebut merupakan akses ke root (GET / HTTP/1.1). Jika sesuai, server akan mengirimkan respons dengan status berhasil beserta isi halaman utama. Sebaliknya, jika request tidak sesuai, server akan mengembalikan respons dengan status halaman tidak ditemukan dan mengirimkan halaman error. Dengan perubahan ini, server menjadi lebih “cerdas” karena dapat memberikan respons berbeda tergantung permintaan klien, tidak lagi selalu mengirimkan satu jenis respons seperti sebelumnya.

Setelah itu dilakukan refactoring untuk merapikan kode. Pendekatan yang digunakan adalah dengan menentukan status respons dan nama file sekaligus dalam satu langkah menggunakan pasangan nilai (tuple). Dengan cara ini, proses membaca file, menghitung panjang konten, dan mengirimkan respons tidak perlu ditulis dua kali. Hasilnya, kode menjadi lebih singkat, lebih bersih, dan lebih mudah dipahami. Perubahan ini juga mencerminkan penerapan prinsip DRY (Don't Repeat Yourself), yaitu menghindari duplikasi kode agar program lebih efisien dan maintainable.

## Commit 4 Reflection
Pada commit keempat, fungsi handle_connection mengalami pengembangan lebih lanjut dengan beberapa peningkatan penting. Struktur percabangan yang sebelumnya menggunakan if-else kini diganti dengan pendekatan pattern matching, sehingga penanganan berbagai jenis request menjadi lebih rapi dan mudah dibaca. Selain itu, server tidak hanya menangani rute utama dan rute tidak valid, tetapi juga ditambahkan rute baru yaitu /sleep. Untuk semua request yang tidak dikenali, digunakan pola default sebagai penangkap umum yang akan mengembalikan respons 404.

Rute /sleep dibuat untuk mensimulasikan masalah pada server yang berjalan secara single-threaded. Pada rute ini ditambahkan jeda selama beberapa detik sebelum server memberikan respons. Akibatnya, ketika ada satu request yang membutuhkan waktu lama untuk diproses, seluruh server menjadi tidak responsif terhadap request lain selama proses tersebut berlangsung.

Hal ini terjadi karena server hanya memproses satu koneksi dalam satu waktu. Setiap koneksi yang masuk ditangani secara berurutan dan harus selesai terlebih dahulu sebelum koneksi berikutnya diproses. Simulasi ini menunjukkan keterbatasan arsitektur single-threaded dan menegaskan pentingnya penggunaan multi-threading agar server dapat menangani banyak request secara bersamaan tanpa saling menghambat.

## Commit 5 Reflection
Pada commit kelima, terjadi perubahan signifikan pada arsitektur server dengan diperkenalkannya konsep multithreading menggunakan thread pool. Pada bagian utama program, server kini membuat sejumlah worker thread (misalnya empat) yang akan digunakan untuk menangani koneksi masuk. Setiap koneksi yang diterima tidak lagi diproses secara langsung dan berurutan, melainkan didistribusikan ke thread pool melalui sebuah mekanisme eksekusi tugas. Dengan pendekatan ini, server dapat menangani banyak request secara bersamaan tanpa saling memblokir.

Sementara itu, file baru berfungsi sebagai implementasi dari thread pool itu sendiri. Di dalamnya terdapat struktur yang mengelola kumpulan worker serta sistem komunikasi antar thread menggunakan channel. Setiap worker berjalan dalam loop yang terus menunggu tugas, lalu mengeksekusinya ketika menerima pekerjaan baru. Untuk memungkinkan banyak worker mengakses sumber tugas yang sama secara aman, digunakan mekanisme berbagi data dengan sinkronisasi.

Saat sebuah tugas dikirim, tugas tersebut dikemas dan dimasukkan ke dalam channel, lalu akan diambil oleh salah satu worker yang tersedia. Pendekatan ini memanfaatkan sistem kepemilikan dan keamanan memori Rust, sehingga concurrency dapat dilakukan tanpa risiko data race. Hasilnya, server menjadi jauh lebih responsif, terbukti ketika mengakses rute yang memiliki delay tidak lagi menghambat request lain.

## Commit Bonus Reflection
Pada commit bonus ini, dilakukan peningkatan pada mekanisme pembuatan thread pool dengan menambahkan penanganan error yang lebih baik. Sebelumnya, pembuatan thread pool dilakukan secara langsung tanpa validasi, namun kini diperkenalkan sebuah method baru yang mengikuti pola builder. Method ini menerima jumlah thread sebagai parameter dan mengembalikan hasil dalam bentuk Result, sehingga kemungkinan kegagalan dapat ditangani secara eksplisit.

Untuk mendukung hal tersebut, dibuat sebuah struktur khusus yang merepresentasikan error ketika pembuatan thread pool gagal. Struktur ini dilengkapi dengan implementasi formatting standar sehingga pesan error yang dihasilkan menjadi lebih informatif dan mudah dipahami, baik untuk debugging maupun untuk ditampilkan ke pengguna.

Selain itu, terdapat validasi tambahan untuk memastikan bahwa jumlah thread yang diberikan harus bernilai positif dan tidak nol. Jika kondisi ini tidak terpenuhi, maka proses pembuatan thread pool akan gagal dan mengembalikan error yang sesuai. Perubahan ini membuat sistem menjadi lebih aman dan robust karena mencegah konfigurasi yang tidak valid sejak awal.

Di sisi penggunaan, cara inisialisasi thread pool juga disesuaikan untuk menangani nilai Result yang dikembalikan. Dengan demikian, keseluruhan perubahan ini menunjukkan penerapan prinsip error handling yang lebih baik dalam Rust, sekaligus meningkatkan keandalan program.