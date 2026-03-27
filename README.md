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
1. Dalam Observer pattern misal di Java, Observer dijadikan interface agar Publisher bisa berinteraksi dengan berbagai jenis Subscriber tanpa tahu implementasi spesifiknya. Misal kita punya 2 subscriber, yaitu EmailSubscriber dan WebhookSubscriber yg keduanya implement interface Observer, tapi cara mereka update() berbeda, maka kita butuh interface untuk mengabstraksikan perilaku keduanya.

Sementara itu pada BambangShop, single Model struct sudah cukup karena jika kita lihat struct Subscriber:

pub struct Subscriber {
    pub url: String,
    pub name: String,
}

Struct Subscriber hanya punya field url sebagai satu2nya cara semua subscriber menerima notifikasi, yaitu melalui HTTP request ke URL mereka. Tidak ada field seperti email, phone, atau subscriber_type yang menandakan adanya variasi perilaku antar subscriber. sementara field nama hanya label untuk indentitas subscriber.

Kalau misal ada variasi perilaku, strukturnya mungkin seperti berikut

pub struct Subscriber {
    pub url: Option<String>,
    pub email: Option<String>,
    pub subscriber_type: String, // "webhook", "email", "sms"
}

Kalau seperti itu, mungkin trait baru diperlukan karena perilaku update() akan berbeda tergantung tipenya. Namun karena semua subscriber di BambangShop melakukan hal yang sama persis, interface tidak diperlukan.

2. Vec adalah list/array dinamis yg menyimpan data secara berurutan berdasarkan index, untuk cari elemen tertentu, harus cek satu per satu dari depan, sedangkan DashMap adalah HashMap yg menyimpan data dalam bentuk key-value, Operasi pencarian dan penghapusan cukup O(1), dan karena key harus unik maka duplikasi otomatis dicegah.

Di BambangShop, operasi yg sering dilakukan adalah mencari, menghapus, dan memastikan tidak ada duplikasi subscriber berdasarkan urlnya. Karena url dan id bersifat unik, mereka sangat cocok dijadikan key di DashMap, hal ini sama seperti di Java kita pakai HashMap<String, Subscriber> bukan ArrayList<Subscriber> untuk data yang punya unique identifier. Vec tidak cukup karena tidak menjamin keunikan dan operasinya tidak efisien untuk kasus ini.

3. Tetap perlu DashMap krn Singleton dan DashMap menyelesaikan masalah yang berbeda.

Singleton pattern memastikan bahwa hanya ada satu instance dari suatu objek di seluruh program, tanpa Singleton, setiap request bisa membuat instance SUBSCRIBERS yang berbeda-beda, sehingga data subscriber terfragmentasi. Dengan Singleton, semua request mengacu ke instance yang sama sehingga data terpusat di satu tempat. Di BambangShop, lazy_static sudah berperan sebagai Singleton.

Adapun Singleton menimbulkan masalah baru, karena hanya ada satu instance dan diakses oleh banyak thread sekaligus, sehingga bisa saja terjadi race condition di mana dua thread menulis ke data yg sama di waktu yang sama dan saling menimpa. Di sini DashMap berperan untuk memastikan akses ke data tersebut aman saat diakses oleh banyak thread secara bersamaan. Rocket menjalankan setiap request di thread yang berbeda, sehingga tanpa DashMap, operasi subscribe dan unsubscribe yang terjadi bersamaan bisa menyebabkan data corrupt.

Jadi keduanya harus ada dan tidak bisa saling menggantikan. Singleton tanpa DashMap berarti data konsisten tapi rawan race condition. DashMap tanpa Singleton berarti thread safe tapi data terfragmentasi di banyak instance.

#### Reflection Publisher-2
1. Berdasarkan Single Responsibility Principle, setiap class atau module hanya bertanggungjawab atas satu hal.
Misal kalau Model menangani semuanya, maka satu class akan bertanggung jawab atas tiga hal sekaligus:

- Struktur data, yaitu seperti apa bentuk datanya
- Business logic, seperti validasi, kalkulasi, dst
- Akses data, yaitu cara menyimpan dan mengambil data dari database

Hal ini melanggar SRP, karena kalau database diganti, atau business logic berubah, atau struktur data berubah, semuanya mengubah satu class yang sama. Dengan memisahkan Model, Repository, dan Service berdasarkan tanggungjawab berikut:

- Model, hanya mendefinisikan struktur data
- Repository, hanya menangani akses dan penyimpanan data
- Service, hanya menangani business logic

Setiap lapisan bisa berubah secara independen tanpa mempengaruhi lapisan lain.

2. Jika kita hanya menggunakan Model, akibatnya adalah Model Product, Subscriber, dan Notification harus saling berinteraksi langsung tanpa Service dan Repository.

Product harus tahu cara menyimpan dirinya sendiri ke database, harus tahu cara mengambil daftar Subscriber yg follow, jga harus tahu cara membuat Notification dan cara mengirimkannya.

Subscriber harus tahu cara menyimpan dirinya sendiri, juga harus tahu cara mengambil Product yg dia ikuti.

Notification harus tahu cara membuat dirinya sendiri berdasarkan event dari Program, sekaligus cara mengirimkan dirinya sendiri ke Subscriber.

Akibatnya setiap Model menjadi sangat kompleks dan saling bergantung satu sama lain, Perubahan kecil di Subscriber bisa merusak Program karena Program langsung bergantung ke Subscriber. Kode jd sulit dibaca, sulit ditest, dan sulit dimaintain.

3. Ya, menurut saya Postman sangat membantu dalam testing BambangShop karena kita bisa mengirim HTTP request (GET, POST, DELETE) ke endpoint yang tersedia tanpa perlu membuat frontend terlebih dahulu. Beberapa fitur yg berguna menurut saya adalah:
- Environment Variables, kita bisa menyimpan base URL seperti http://127.0.0.1:8000 sebagai variable, sehingga kalau port berubah cukup ubah di satu tempat tanpa harus update semua endpoint satu2.
- Request History, semua request yang pernah dikirim tersimpan, mudah untuk mengulang test yang sama.
- Response Viewer, response dari server ditampilkan dengan JSON terformat sehingga mudah dibaca.


#### Reflection Publisher-3

