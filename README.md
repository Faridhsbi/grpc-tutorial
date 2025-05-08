# Reflection - Module 8

> 1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?

Unary RPC adalah komunikasi satu permintaan-satu respons, cocok untuk operasi sederhana seperti autentikasi atau pengambilan data tunggal.

Server Streaming memungkinkan server mengirim banyak respons terhadap satu permintaan, ideal untuk pengiriman data berkelanjutan, contohnya pada notifikasi real-time atau pengiriman file besar. 

Bi-Directional Streaming memfasilitasi komunikasi dua arah secara asinkron, cocok untuk aplikasi seperti chat atau game multi-pemain. Pemilihan metode bergantung pada kebutuhan latensi, volume data, dan interaktivitas.

> 2. What are the potential security considerations involved in implementing a gRPC service inRust, particularly regarding authentication, authorization, and data encryption?

Autentikasi dapat diimplementasikan menggunakan TLS untuk enkripsi data, JWT/OAuth untuk validasi token, atau sertifikat mutual. Otorisasi diperlukan untuk membatasi akses ke endpoint tertentu misalnya RBAC. Enkripsi data wajib menggunakan TLS untuk mencegah sniffing. Validasi input dan sanitasi data juga penting untuk menghindari serangan injeksi.

> 3. What are the potential challenges or issues that may arise when handling bidirectionalstreaming in Rust gRPC, especially in scenarios like chat applications?

Dalam implementasi bidirectional streaming dengan Rust gRPC untuk aplikasi chat, tantangan utama meliputi manajemen state yang kompleks akibat interaksi yang terus menerus antara klien dan server, serta kebutuhan untuk operasi asyncrhonus untuk mengirim dan menerima pesan secara bersamaan tanpa blokir.

> 4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?

Dalam penggunaan `tokio_stream::wrappers::ReceiverStream` memudahkan konversi channel `(mpsc::Receiver)` menjadi stream yang bisa dikembalikan oleh Tonic, sehingga memisahkan logic pembuatan data dan pengiriman. Namun buffer size-nya statis dan tidak ada backpressure otomatis, sehingga developer harus mengatur ukuran buffer dan mekanisme throttling atau drop sendiri untuk menghindari kelebihan beban.

> 5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

Untuk mendukung reuse dan modularitas pada layanan gRPC di Rust, pisahkan kode ke modul‑modul terpisah menurut tanggung jawab: satu modul untuk definisi Protobuf, satu untuk implementasi masing‑masing service, dan satu lagi untuk konfigurasi server (TLS, logging, middleware). Dengan memisahkan concern seperti ini, setiap bagian bisa dikembangkan, diuji, dan diperluas secara mandiri, sehingga kode menjadi lebih mudah dipelihara dan di-scale di masa mendatang.

> 6. In the MyPaymentService implementation, what additional steps might be necessary tohandle more complex payment processing logic?

Selain membalas status sukses, MyPaymentService sebaiknya melakukan validasi menyeluruh terhadap data pembayaran, batas nilai, dan format sebelum memproses. Kemudian, integrasi dengan gateway eksternal misalnya Stripe dan Midtrans, diperlukan untuk mengeksekusi transaksi nyata, termasuk menangani callback atau webhook dari penyedia. Transaksi database harus dijalankan secara atomik dengan idempotency key agar duplikasi terhindar dan konsistensi terjaga. Pada kegagalan, mekanisme retry dan kompensasi (rollback) wajib diimplementasikan untuk recovery. Terakhir, audit trail dan notifikasi penting demi keamanan, pelacakan, dan kepatuhan regulasi.

> 7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

Penggunaan gRPC mendorong arsitektur terdistribusi ke model contract‑first dengan .proto sebagai sumber kebenaran, memanfaatkan HTTP/2 dan Protobuf untuk throughput tinggi, latensi rendah, dan stub multi‑bahasa, sehingga layanan mikro dapat dibangun dalam bahasa apa pun dan tetap saling terhubung secara efisien. Akan tetapi, karena gRPC “binary‑native” dan HTTP/2‑only, integrasi dengan klien atau sistem legacy yang hanya memahami REST/HTTP 1.1 memerlukan gateway atau adapter untuk menerjemahkan antara JSON/HTTP dan Protobuf/gRPC, sehingga desain harus memasukkan lapisan integrasi ekstra untuk menjamin interoperabilitas lintas platform dan teknologi.

> 8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

HTTP/2 menjadi dasar gRPC menggunakan framing biner, kompresi header (HPack), dan multiplexing sejati sehingga banyak RPC dapat berjalan paralel pada satu koneksi TCP tanpa head‑of‑line blocking, menghasilkan latensi lebih rendah dan throughput lebih tinggi, serta mendukung server push untuk pengiriman data proaktif. Sedangkan HTTP/1.1 dengan WebSocket membutuhkan koneksi terpisah atau satu kanal duplex tanpa kompresi header ataupun sub‑stream multiplexing, sehingga tidak seefisien HTTP/2 dalam kontrol aliran dan pemanfaatan bandwidth. Di sisi lain, HTTP/2 lebih kompleks di implementasi dan debugging, serta beberapa proxy atau browser lawas mungkin kurang mendukung, sementara HTTP/1.1 dengan WebSocket punya kompatibilitas lebih luas dan tooling yang lebih sederhana.

> 9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

REST API kurang cocok untuk komunikasi real-time karena model request-response pada REST API bersifat sinkron dan terbatas pada pola satu permintaan satu respons. Sebaliknya, gRPC dengan kemampuan streaming dua arah memungkinkan klien dan server untuk mengirim dan menerima data secara simultan sehingga meningkatkan responsivitas dan efisiensi dalam aplikasi yang memerlukan komunikasi real-time

> 10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?
>
Pendekatan berbasis skema pada gRPC menggunakan protocol buffers yang menyediakan struktur data yang menghasilkan struktur data yang terstruktur dan konsisten di seluruh sistem, komunikasi yang lebih efisien dan kinerja yang lebih baik. Akan tetapi, hal tersebut juga dapat mengurangi fleksibilitas karena setiap perubahan pada skema memerlukan update dan recompile di kedua sisi klien dan server. Sedangkan, pendekatan schema-less dari JSON dalam REST API memberikan fleksibilitas lebih dalam manipulasi dan penggunaan data, sehingga dapat memudahkan integrasi dan iterasi, tetapi juga dapat meningkatkan kompleksitas dan risiko kesalahan dalam pengelolaan data.