# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

#### **1. In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model struct is enough?**  
Dalam **Observer Pattern**, `Subscriber` biasanya didefinisikan sebagai sebuah **interface** (atau trait dalam Rust) agar memungkinkan berbagai jenis subscriber dengan perilaku berbeda. Namun, dalam BambangShop, setiap subscriber hanya memiliki **dua atribut utama: `url` dan `name`**, dan tidak ada variasi perilaku antara satu subscriber dengan yang lain.  

Karena itu, **penggunaan satu model struct sudah cukup** untuk menangani semua subscriber. Jika di masa depan sistem berkembang dan membutuhkan berbagai jenis subscriber dengan perilaku berbeda (misalnya, subscriber yang menggunakan WebSocket atau memiliki format notifikasi berbeda), barulah kita perlu mempertimbangkan penggunaan trait agar lebih fleksibel.  

---

#### **2. id in Program and url in Subscriber is intended to be unique. Explain based on your understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently use is necessary for this case?**  
Dalam implementasi daftar subscriber, kita perlu menyimpan dan mengakses subscriber berdasarkan **URL** sebagai identifier unik. Jika kita menggunakan `Vec<Subscriber>`, maka setiap operasi pencarian, penambahan, atau penghapusan harus dilakukan dengan **iterasi manual** (`O(n)` complexity), yang tidak efisien jika jumlah subscriber bertambah banyak.  

Sebaliknya, **DashMap** menggunakan struktur **hash-based key-value storage**, yang memungkinkan **akses langsung ke subscriber berdasarkan URL dalam waktu O(1)**. Selain itu, DashMap **mencegah duplikasi**, sehingga tidak perlu memeriksa secara manual apakah URL sudah ada sebelum menambah subscriber baru.  

---

#### **3. When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we used the DashMap external library for thread safe HashMap. Explain based on your understanding of design patterns, do we still need DashMap or we can implement Singleton pattern instead?**  
Meskipun kita dapat menggunakan **Singleton Pattern** dengan `Mutex<HashMap<..>>` atau `RwLock<HashMap<..>>`, pendekatan ini memiliki keterbatasan dalam lingkungan **multi-threaded**:  
- **`Mutex<HashMap<..>>`**: Mengunci seluruh `HashMap` saat ada satu thread yang menulis, membuat thread lain harus menunggu giliran.  
- **`RwLock<HashMap<..>>`**: Memungkinkan beberapa thread membaca secara bersamaan, tetapi tetap membatasi hanya satu thread yang bisa menulis pada satu waktu.  

Pendekatan ini bisa menyebabkan **bottleneck**, terutama jika banyak subscriber yang bertambah atau dihapus dalam waktu bersamaan.  

Sebagai alternatif, DashMap lebih optimal karena menggunakan teknik "sharding". Sharding dalam DashMap berarti bahwa data dipecah menjadi beberapa bagian (shards), dan setiap shard memiliki lock sendiri. Dengan cara ini:  
- **Beberapa thread dapat membaca dan menulis ke bagian (shard) yang berbeda tanpa saling mengganggu.**  
- **Tidak ada satu titik tunggal (global lock) yang menghambat seluruh operasi**, seperti yang terjadi pada `Mutex<HashMap<..>>`.  
- **Operasi parallel menjadi lebih efisien**, terutama untuk aplikasi yang memproses banyak subscriber dalam waktu bersamaan.  

Dengan kata lain, DashMap menghilangkan kelemahan utama dari pendekatan Singleton biasa, menjadikannya pilihan yang lebih efisien untuk mengelola daftar subscriber yang perlu diakses oleh banyak thread secara bersamaan.  

--- 

#### Reflection Publisher-2

#### **1. In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?**  
Dalam konsep **Model-View-Controller (MVC)**, model bertanggung jawab atas penyimpanan data dan logika bisnis. Namun, pendekatan ini bisa menyebabkan monolitik dan sulit dikelola seiring bertambahnya kompleksitas sistem.  

Dengan memisahkan **Service** dan **Repository**, kita mendapatkan:  
- **Repository**: Berfungsi sebagai lapisan akses data, menangani **query dan operasi database**.  
- **Service**: Mengelola **logika bisnis**, validasi, dan interaksi antara model tanpa terikat langsung ke database.  

Pendekatan ini membuat kode lebih modular, mudah diuji, dan scalable, karena logika bisnis tidak bercampur dengan kode akses data.  

---

#### **2. What happens if we only use the Model? Explain your imagination on how the interactions between each model (Program, Subscriber, Notification) affect the code complexity for each model?**  
Jika kita hanya menggunakan Model tanpa Service dan Repository, setiap model akan menangani akses data dan logika bisnis sekaligus, menyebabkan beberapa masalah:  
- **Ketergantungan antar Model meningkat**, misalnya `Notification` harus mengetahui detail `Subscriber` dan `Product`, sehingga perubahan kecil dalam satu model bisa berdampak besar pada model lainnya.  
- **Duplikasi kode**, karena logika bisnis akan tersebar di berbagai model, membuat perawatan kode lebih sulit.  
- **Kesulitan dalam pengujian**, karena unit test harus selalu berinteraksi dengan database, bukan hanya menguji logika bisnis secara independen.  

Memisahkan Service dan Repository mengurangi **coupling**, membuat kode lebih bersih, dan lebih fleksibel untuk dikembangkan.  

---

#### **3. Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects**  
Postman sangat membantu dalam menguji API tanpa perlu membuat frontend atau menggunakan command line. Dalam implementasi ini, Postman mempermudah:  
- **Mengirim request HTTP (POST, GET, DELETE, dll.)** untuk menguji `subscribe` dan `unsubscribe` pada `NotificationService`.  
- **Melihat response JSON secara langsung**, termasuk error handling.  
- **Menyimpan dan mengorganisasi request API**, berguna untuk dokumentasi dan pengujian di masa depan.  

Fitur menarik yang menurut saya akan berguna untuk proyek selanjutnya ada, **Automated testing**,
**Environment variables**, **Collection runner**: 

Dengan Postman, pengujian API menjadi lebih efisien, terstruktur, dan mudah diulang, mendukung pengembangan berbasis TDD dan CI/CD.  

---

#### Reflection Publisher-3 

#### **1. Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?**  
Kita menggunakan **Push Model**, di mana **publisher (sistem)** secara langsung mengirimkan **notifikasi** ke **subscribers** saat terjadi perubahan pada produk. Setiap kali produk dibuat, dipromosikan, atau dihapus, notifikasi langsung dikirim melalui HTTP POST ke URL subscriber.  

#### **2. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)**  
Jika kita menggunakan **Pull Model**, subscriber harus secara berkala **memeriksa (polling)** apakah ada pembaruan pada produk.  

**Keuntungan Pull Model:**  
- Subscriber memiliki **kontrol penuh** kapan harus mengambil data.  
- Menghindari **overhead komunikasi** jika subscriber tidak selalu membutuhkan pembaruan.  

**Kekurangan Pull Model:**  
- **Tidak real-time**, karena subscriber harus melakukan polling secara berkala.  
- **Beban server meningkat** jika banyak subscriber melakukan polling berulang.  
- **Inefisiensi jaringan**, karena polling tetap dilakukan meskipun tidak ada perubahan data.  

Dalam kasus ini, **Push Model lebih efektif** karena perubahan produk langsung dikirimkan ke subscriber tanpa perlu polling.  

#### **3. Explain what will happen to the program if we decide to not use multi-threading in the notification process.**  
Tanpa **multi-threading**, notifikasi ke subscriber akan dikirim **secara berurutan (synchronous execution)**.  

**Dampaknya:**  
- **Latensi meningkat**, karena satu subscriber harus menunggu yang lain selesai menerima notifikasi sebelum lanjut ke berikutnya.  
- **Bottleneck dalam sistem**, terutama jika ada banyak subscriber atau server mereka lambat merespons.  
- **Kurang scalable**, karena satu request bisa memblokir keseluruhan proses notifikasi.  

Dengan **multi-threading**, setiap notifikasi dikirim dalam thread terpisah, memungkinkan pengiriman paralel yang lebih cepat dan responsif.  

---