# 2. Application Patterns
### Pola-pola Aplikasi / Application Patterns

Bab ini membahas pola-pola yang muncul saat membangun aplikasi modern: bagaimana memecah sistem jadi banyak layanan (*microservice*), bagaimana menangani operasi yang lama (*async*), bagaimana mengakses data dengan benar, kapan butuh *cache*, kapan butuh komunikasi *event-driven*, dan bagaimana memastikan operasi *idempotent*.

This chapter covers patterns that emerge when building modern applications: how to split a system into multiple services (*microservice*), how to handle long-running operations (*async*), how to access data correctly, when to use *caching*, when to use *event-driven* communication, and how to keep operations *idempotent*.

---

## Microservice

### Apa ini? / What is this?

**ID:** Microservice adalah pendekatan membangun aplikasi sebagai sekumpulan layanan kecil yang berdiri sendiri, masing-masing fokus pada satu *kemampuan bisnis*. Berbeda dengan *monolith* (satu aplikasi besar yang melakukan semuanya), microservice memecah sistem jadi banyak unit independen yang berkomunikasi lewat jaringan.

**EN:** Microservice is an approach to building applications as a collection of small, independent services, each focused on one *business capability*. Unlike a *monolith* (one big app doing everything), microservices split the system into many independent units that communicate over the network.

Analogi: pabrik mobil dengan banyak bengkel kecil (engine, body, electronics), masing-masing tahu pekerjaannya sendiri, lebih mudah memperbaiki/upgrade satu bengkel tanpa menghentikan seluruh pabrik.

### Mengapa penting? / Why it matters?

- **Skala mandiri** — bagian yang sibuk (mis. payment) bisa di-scale tanpa scale seluruh sistem.
- **Deployment independen** — fix bug di service A tidak perlu redeploy service B.
- **Pemisahan tim** — tim berbeda bisa pegang service berbeda tanpa saling mengganggu.
- **Teknologi heterogen** — service A bisa pakai Python, service B pakai Go, tanpa konflik.

Tapi: microservice **bukan obat mujarab**. Kalau tim kecil & domain belum kompleks, monolith yang terstruktur sering lebih baik.

### Use Case

> **Skenario:** Aplikasi e-commerce dengan modul Catalog, Cart, Order, Payment, Shipping. Saat flash sale, traffic ke Catalog & Cart melonjak 50x lipat, tapi Payment tetap normal.
>
> Dengan microservice: scale Catalog & Cart ke 50 pod, Payment tetap 3 pod. Hemat biaya infrastruktur.
>
> Dengan monolith: harus scale seluruh aplikasi 50x lipat → 50x biaya, padahal sebagian besar pod idle.
{ .usecase }

### Istilah & Konsep / Glossary

- **Stateless** — service tidak menyimpan data di memorinya sendiri antar request. Jika butuh state, simpan di Redis / database eksternal. Dengan stateless, request bisa dilayani pod mana saja → mudah di-scale.
- **Bounded Context** (dari DDD) — batas logis sebuah area bisnis. "Order" di konteks penjualan ≠ "Order" di konteks dapur restoran. Setiap microservice idealnya = satu bounded context.
- **Ubiquitous Language** — istilah yang sama dipakai oleh developer, business analyst, dan dokumen. Tidak boleh kode menyebut "Cart" tapi dokumen bisnis menyebut "Basket".
- **Distributed Monolith** — anti-pattern: punya banyak service tapi mereka saling terkait erat sehingga harus di-deploy bersamaan. Punya semua kerumitan microservice tanpa manfaatnya.
- **Saga** — pola untuk mengelola transaksi yang melewati banyak service. Karena tidak ada *database transaction* lintas service, Saga membagi proses jadi langkah-langkah, masing-masing punya *compensating action* (langkah pembatalan).
- **Choreography vs Orchestration** — dua gaya Saga: choreography = setiap service "mendengarkan" event dan bereaksi (tanpa kontrol pusat). Orchestration = ada satu service yang memimpin alur.
- **Circuit Breaker** — pola seperti sekring listrik di rumah. Jika service downstream sering gagal, sirkuit "terbuka" → request langsung ditolak (tanpa menunggu timeout 30 detik) sampai service pulih.
- **2PC (Two-Phase Commit)** — protokol klasik untuk transaksi lintas database. Sangat rapuh di lingkungan terdistribusi dan jarang dipakai di microservice modern → diganti dengan Saga.
- **Rate Limiting** — pembatas jumlah request per satuan waktu (mis. 100 req/menit per user). Mencegah abuse & melindungi service dari overload.
- **API Gateway** — pintu masuk tunggal untuk semua request dari luar, sebelum diarahkan ke service yang tepat.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ **Database bersama** antar service → mereka jadi kopel erat di level data, ubah skema = pecah banyak service.
- ❌ **Library bisnis bersama** (mis. `shared-business-logic.jar`) → setiap update library memaksa redeploy semua service = distributed monolith.
- ❌ **Memanggil service lain di tengah transaksi DB** → kalau service lain lambat, transaksi DB juga lambat → contention.
- ❌ **2PC lintas service** → rapuh, tidak skalabel.
- ❌ **Tidak ada retry / circuit breaker** untuk panggilan eksternal → satu service lambat = semua service lambat (cascading failure).
</div>

### Aturan / Rules

#### Mandatory

<span class="badge badge-mandatory">Mandatory</span> Setiap service harus *stateless*. Jika butuh state, gunakan external store (Redis, DB).

<span class="badge badge-mandatory">Mandatory</span> Setiap service memiliki data store-nya sendiri — tidak ada database bersama.

<span class="badge badge-mandatory">Mandatory</span> Service berkomunikasi via REST/gRPC sinkron (untuk query) atau messaging asinkron (untuk command/event).

<span class="badge badge-mandatory">Mandatory</span> Batas service sejajar dengan *bounded context* — jangan pecah satu bounded context jadi banyak service.

<span class="badge badge-mandatory">Mandatory</span> Hindari *distributed monolith* — service tidak berbagi library yang berisi logika bisnis.

<span class="badge badge-mandatory">Mandatory</span> Gunakan *Ubiquitous Language* secara konsisten dalam satu bounded context.

<span class="badge badge-mandatory">Mandatory</span> *Aggregate Root* adalah satu-satunya pintu untuk mengubah state aggregate.

<span class="badge badge-mandatory">Mandatory</span> Komunikasi lintas bounded context lewat *Domain Event*, bukan kopel langsung.

<span class="badge badge-mandatory">Mandatory</span> Setiap service bisa di-deploy secara independen tanpa rilis terkoordinasi.

#### Optional — Distributed Transaction (Saga)

<span class="badge badge-optional">Optional</span> JANGAN pakai 2PC lintas service — pakai Saga sebagai gantinya.

<span class="badge badge-optional">Optional</span> Pilih gaya Saga berdasarkan kompleksitas: Choreography (sederhana, event-based) atau Orchestration (mudah dilacak, ada koordinator pusat).

<span class="badge badge-optional">Optional</span> Setiap langkah Saga punya *compensating transaction* untuk rollback.

<span class="badge badge-optional">Optional</span> Saga state harus persisten — bisa bertahan setelah restart proses.

<span class="badge badge-optional">Optional</span> Desain untuk partial failure — sistem selalu bisa mencapai state konsisten akhirnya.

#### Optional — Circuit Breaker & Retry

<span class="badge badge-optional">Optional</span> Bungkus semua outbound call (HTTP, gRPC, DB, broker) dengan circuit breaker.

<span class="badge badge-optional">Optional</span> Status circuit breaker: **Closed** (normal), **Open** (gagal — tolak segera), **Half-Open** (uji pemulihan).

<span class="badge badge-optional">Optional</span> Retry: maksimal 3x dengan *exponential backoff* + *random jitter*.

<span class="badge badge-optional">Optional</span> JANGAN retry untuk: error 4xx, business rule violation, atau operasi non-idempotent.

<span class="badge badge-optional">Optional</span> Log setiap perubahan status circuit breaker sebagai warning.

<span class="badge badge-optional">Optional</span> Ekspos status circuit breaker via metrics.

#### Optional — Rate Limiting & Throttling

<span class="badge badge-optional">Optional</span> Terapkan rate limiting di API Gateway untuk semua endpoint publik.

<span class="badge badge-optional">Optional</span> Atur limit per: user, tenant, IP, dan global per endpoint.

<span class="badge badge-optional">Optional</span> Saat limit terlampaui, kembalikan `HTTP 429 Too Many Requests` dengan header `Retry-After`.

<span class="badge badge-optional">Optional</span> Whitelist panggilan internal antar service dari rate limit user-facing.

<span class="badge badge-optional">Optional</span> Log pelanggaran rate limit untuk monitoring abuse.

### Contoh Struktur / Example

```text
[root]
├── src/
│   ├── shared/
│   │   ├── contracts/      ← event schema, DTO antar service
│   │   └── common/         ← logger, http-client wrapper (NO business logic!)
│   └── service-name/
│       ├── src/
│       │   ├── presentation/
│       │   ├── application/
│       │   ├── domain/
│       │   └── infrastructure/
│       ├── tests/
│       │   ├── unit/
│       │   ├── integration/
│       │   └── contract/
│       └── Dockerfile
├── infra/
└── docker-compose.yml
```

---

## Async & Concurrency

### Apa ini? / What is this?

**ID:** "Async" (asynchronous) artinya operasi tidak langsung mengembalikan hasil — kita melepaskannya untuk dikerjakan di belakang sambil program lanjut melakukan hal lain. "Concurrency" artinya banyak hal berjalan "bersamaan" — bisa di banyak thread (multi-threading) atau di banyak CPU/core (parallelism).

**EN:** "Async" means an operation doesn't return a result immediately — we hand it off to be done in the background while the program continues. "Concurrency" means many things run "simultaneously" — either across many threads (multi-threading) or many CPUs/cores (parallelism).

Analogi: di restoran, koki tidak menunggu nasi matang baru goreng ayam — ia mulai keduanya bersamaan (concurrency). Sementara itu pelayan tidak menunggu satu meja selesai makan baru melayani meja lain (async).

### Mengapa penting? / Why it matters?

- **Pengalaman pengguna** — request HTTP yang butuh 30 detik (mis. generate laporan PDF) tidak boleh memblokir browser.
- **Throughput** — server bisa melayani lebih banyak request bila tidak menunggu I/O.
- **Tahan banting** — pekerjaan asinkron dengan queue tidak hilang meskipun server restart.

### Use Case

> **Skenario 1 (Async):** User klik "Generate Laporan Tahunan". Tanpa async, browser menunggu 2 menit dan timeout. Dengan async: server taruh task di queue → langsung kembalikan "Laporan sedang diproses, akan dikirim ke email Anda" → user lanjut bekerja.
>
> **Skenario 2 (Multi-threading):** Server menerima 1000 request paralel untuk fetch data. Jika single-thread, request ke-1000 menunggu 999 request selesai. Dengan thread pool ukuran 100, sampai 100 request bisa diproses bersamaan.
>
> **Skenario 3 (Parallelism):** Anda memproses 10 juta record. Single-core butuh 1 jam. Dengan partitioning + 10 worker paralel di 10 core, selesai dalam 6 menit.
{ .usecase }

### Istilah & Konsep / Glossary

- **Thread Pool** — sekumpulan thread yang sudah disiapkan, dipakai berulang. Lebih efisien daripada membuat thread baru setiap task.
- **I/O-bound vs CPU-bound** — task yang menunggu disk/network = I/O-bound (perlu banyak thread, CPU idle). Task hitung-hitungan berat = CPU-bound (jumlah thread ≤ core, kalau lebih malah pelan).
- **Dead-letter Queue (DLQ)** — antrian khusus untuk pesan yang gagal diproses beberapa kali. Diperiksa manual/otomatis untuk investigasi.
- **Race Condition** — bug ketika dua thread mengakses data yang sama tanpa sinkronisasi, hasilnya bergantung "siapa duluan" — sulit di-debug.
- **Deadlock** — dua thread saling menunggu lock yang dipegang yang lain → keduanya berhenti selamanya.
- **Cancellation Token** — mekanisme untuk membatalkan operasi async (mis. user pencet "Cancel").
- **Backpressure** — saat consumer lebih lambat dari producer, sistem perlu mekanisme untuk "menahan" producer agar tidak overload.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Blok thread HTTP server untuk operasi yang lama → throughput jatuh, request lain antri.
- ❌ Membuat thread tanpa batas (`new Thread()` di setiap request) → OutOfMemory.
- ❌ Tidak menangani error di task async → silently swallowed, bug tidak terdeteksi.
- ❌ Lock bersarang dengan urutan tidak konsisten → deadlock.
- ❌ Asumsi "parallel pasti lebih cepat" — overhead konteks switch & koordinasi bisa lebih besar dari gain.
</div>

### Aturan / Rules

#### Mandatory — Async

<span class="badge badge-mandatory">Mandatory</span> Operasi yang tidak butuh hasil sinkron harus diproses async.

<span class="badge badge-mandatory">Mandatory</span> Task lama dioffload ke background worker / message queue — jangan blokir request thread.

<span class="badge badge-mandatory">Mandatory</span> Operasi async harus *observable* — punya endpoint status atau emit event untuk completion & failure.

<span class="badge badge-mandatory">Mandatory</span> Selalu handle failure async secara eksplisit — *dead-letter queue* wajib dikonfigurasi.

#### Mandatory — Multi-threading

<span class="badge badge-mandatory">Mandatory</span> Shared mutable state dilindungi mekanisme sinkronisasi yang sesuai.

<span class="badge badge-mandatory">Mandatory</span> Lebih baik pakai struktur data immutable jika memungkinkan — hilangkan kebutuhan locking.

<span class="badge badge-mandatory">Mandatory</span> Ukuran thread pool dikonfigurasi eksplisit — jangan andalkan default.

<span class="badge badge-mandatory">Mandatory</span> Hindari thread starvation: pisahkan thread pool untuk I/O-bound dan CPU-bound.

<span class="badge badge-mandatory">Mandatory</span> Pola rawan deadlock (nested lock, urutan lock tidak konsisten) di-review di code review.

#### Mandatory — Parallelism

<span class="badge badge-mandatory">Mandatory</span> Gunakan parallelism hanya jika operasi benar-benar independen dan overhead-nya sepadan.

<span class="badge badge-mandatory">Mandatory</span> Partisi workload secara eksplisit — jangan andalkan implicit parallelism.

<span class="badge badge-mandatory">Mandatory</span> Operasi paralel punya timeout & cancellation strategy yang jelas.

<span class="badge badge-mandatory">Mandatory</span> Error di satu cabang paralel tidak boleh menelan error cabang lain — agregasi & surface semua failure.

<span class="badge badge-mandatory">Mandatory</span> Lakukan benchmark sebelum-sesudah untuk memastikan parallelism benar-benar meningkatkan throughput.

---

## Data Access

### Apa ini? / What is this?

**ID:** Cara aplikasi membaca & menulis data ke penyimpanan (database, file, dll). Pola yang dipakai menentukan: kebersihan kode, performa, dan kemudahan testing.

**EN:** How the application reads & writes data to storage (database, files, etc.). The patterns used determine code cleanliness, performance, and testability.

### Mengapa penting? / Why it matters?

- Tanpa abstraksi, SQL/ORM tersebar di seluruh kode → ganti DB = pekerjaan masif.
- Query tanpa pagination = bom waktu (saat data tumbuh, query menggantung server).
- N+1 query = penyakit klasik yang membuat aplikasi pelan tanpa terlihat di awal.

### Use Case

> **Skenario N+1:** API `/orders` mengembalikan 100 order, masing-masing dengan list `items`. Kode naif: 1 query ambil 100 order, lalu 100 query lagi (satu per order) untuk ambil items → **101 query** untuk 1 endpoint! Database overloaded saat traffic tinggi.
>
> Solusi: *eager loading* atau *batch fetching* → 2 query saja (1 untuk orders, 1 untuk semua items di orders tersebut, lalu join di memory).
{ .usecase }

### Istilah & Konsep / Glossary

- **Repository Pattern** — abstraksi seperti "koleksi" entity di memori. Kode bisnis cukup `orderRepo.findById(id)`, tidak tahu apakah belakangnya SQL, MongoDB, atau in-memory dictionary.
- **ORM (Object-Relational Mapping)** — library yang memetakan tabel ke class (mis. Hibernate, Entity Framework, SQLAlchemy, Prisma).
- **Query Object** — pola untuk merangkai filter query kompleks tanpa SQL hardcoded di service.
- **N+1 Problem** — bug performa: 1 query parent + N query child = N+1 round-trip ke DB.
- **Connection Pool** — kumpulan koneksi DB yang dipakai bergantian; jauh lebih cepat daripada buka-tutup koneksi tiap request.
- **CQRS (Command Query Responsibility Segregation)** — pola memisahkan model & path untuk operasi *write* (command) dan *read* (query). Cocok kalau pola baca jauh berbeda dari pola tulis (mis. tulis transaksional, baca laporan analitik).
- **Pagination** — membatasi jumlah data yang dikembalikan per request (mis. 50 baris per halaman).

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Inline SQL string di controller / service: `db.execute("SELECT * FROM users WHERE id=" + userId)` → rawan SQL injection + sulit di-test.
- ❌ `findAll()` tanpa pagination di endpoint API publik.
- ❌ Connection di-open di setiap query tanpa pool.
- ❌ Loop dalam loop yang memicu query DB di setiap iterasi.
</div>

### Aturan / Rules

#### Mandatory

<span class="badge badge-mandatory">Mandatory</span> Pakai *Repository Pattern* untuk semua akses data domain.

<span class="badge badge-mandatory">Mandatory</span> Jangan tulis SQL mentah inline di logika bisnis — pakai ORM, query builder, atau stored procedure secara konsisten.

<span class="badge badge-mandatory">Mandatory</span> Pagination wajib untuk semua list query — tidak boleh ada result set tanpa batas.

<span class="badge badge-mandatory">Mandatory</span> Connection DB dikelola via *connection pool* dengan min/max yang eksplisit.

<span class="badge badge-mandatory">Mandatory</span> Pakai *Query Object* untuk read yang kompleks.

<span class="badge badge-mandatory">Mandatory</span> Hindari N+1 query — gunakan eager loading atau batch fetching.

#### Optional

<span class="badge badge-optional">Optional</span> Pisahkan read model dari write model (*CQRS*) jika beban baca & tulis sangat berbeda.

---

## Cache Management

### Apa ini? / What is this?

**ID:** Cache adalah penyimpanan sementara yang lebih cepat diakses daripada sumber aslinya. Bayangkan menyimpan jawaban yang sering ditanya di sticky note di meja, daripada bolak-balik buka buku tebal.

**EN:** A cache is temporary storage that's faster to access than the original source. Like sticky notes on your desk with frequent answers, instead of opening a thick book every time.

### Mengapa penting? / Why it matters?

- Database sering jadi *bottleneck* — cache memindahkan beban ke memory.
- Saat flash sale, halaman produk di-hit jutaan kali — tanpa cache, DB tewas.
- Latensi turun drastis (mis. dari 80ms ke 2ms).

### Use Case

> **Skenario flash sale:** Halaman detail produk best-seller di-hit 100.000x/menit. Tanpa cache: 100.000 query identik ke DB → DB overload. Dengan cache TTL 60 detik: hanya ~1 query/menit ke DB (saat cache expire).
{ .usecase }

### Istilah & Konsep / Glossary

- **TTL (Time-To-Live)** — masa berlaku entry cache. Setelah TTL habis, entry dianggap kadaluwarsa.
- **Cache Hit / Miss** — *hit* = data ada di cache. *Miss* = tidak ada, harus ambil dari sumber.
- **Cache-Aside (Lazy Loading)** — aplikasi cek cache dulu; jika miss, baca dari DB & isi cache. Pola paling umum.
- **Write-Through** — setiap tulis ke DB, sekaligus update cache.
- **Write-Behind** — tulis ke cache dulu, async tulis ke DB.
- **Cache Invalidation** — menghapus/mengganti entry cache saat data sumbernya berubah. Salah satu *hardest problem in computer science*.
- **Cache Stampede / Thundering Herd** — saat cache satu key expire dan ribuan request bersamaan miss → semua hit DB sekaligus.
- **L1 / L2 Cache** — L1: in-process (di memori aplikasi, paling cepat tapi tidak shared antar instance). L2: Redis / Memcached, shared antar instance.
- **Redis** — in-memory data store yang umum dipakai untuk cache (dan banyak hal lain: queue, pub/sub).

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Cache tanpa TTL → data basi selamanya.
- ❌ Lupa invalidasi saat update → user lihat data lama.
- ❌ Cache key tidak konsisten (`user-123`, `users:123`, `User_123`) → cache miss tanpa sadar.
- ❌ Cache data sensitif (token, password) tanpa enkripsi → bocor jika Redis ter-dump.
</div>

### Aturan / Rules

#### Mandatory

<span class="badge badge-mandatory">Mandatory</span> Data yang diambil by primary key harus di-cache. Invalidate saat update/delete.

<span class="badge badge-mandatory">Mandatory</span> Tentukan strategi caching per use case: *Cache-Aside*, *Write-Through*, atau *Write-Behind*.

<span class="badge badge-mandatory">Mandatory</span> Setiap entry cache punya TTL eksplisit. TTL minimum 24 jam untuk cache database-backed, kecuali ada aturan bisnis lain.

<span class="badge badge-mandatory">Mandatory</span> Cache key mengikuti konvensi konsisten: `{service}:{entity}:{id}`.

#### Optional

<span class="badge badge-optional">Optional</span> Bedakan tier cache: L1 (in-process) & L2 (Redis, shared).

<span class="badge badge-optional">Optional</span> Lindungi dari cache stampede: probabilistic early expiration atau mutex lock saat miss.

<span class="badge badge-optional">Optional</span> Jangan cache data sensitif (PII, kredensial) tanpa enkripsi at-rest.

---

## Service Bus & Event-Driven

### Apa ini? / What is this?

**ID:** *Service Bus* adalah jalur komunikasi (message broker) yang dipakai service untuk saling kirim pesan tanpa langsung memanggil satu sama lain. *Event-driven* berarti service bereaksi pada *kejadian* (event) — bukan menunggu di-perintah.

**EN:** A *service bus* is a communication channel (message broker) services use to exchange messages without calling each other directly. *Event-driven* means services react to *events* — rather than waiting to be told.

### Mengapa penting? / Why it matters?

- **Loose coupling** — service A tidak perlu tahu siapa yang konsumsi event-nya.
- **Tahan banting** — jika consumer down, pesan menunggu di broker, tidak hilang.
- **Mudah menambah consumer baru** — tambahkan subscriber tanpa mengubah producer.

### Use Case

> **Skenario:** Saat `Order` dibuat di Order Service, ada banyak yang perlu bereaksi: Inventory kurangi stok, Email kirim konfirmasi, Loyalty hitung poin, Analytics catat metrik.
>
> Tanpa event: Order Service harus memanggil 4 service satu per satu (HTTP). Jika salah satu lambat/down → Order ikut bermasalah. Penambahan service ke-5 = ubah Order Service.
>
> Dengan event: Order Service publish `OrderPlaced` ke broker → 4 service (& service ke-5 nanti) berlangganan event itu independen.
{ .usecase }

### Istilah & Konsep / Glossary

- **Event** — fakta yang sudah terjadi (mis. `OrderPlaced`, `PaymentReceived`). *Past tense*.
- **Command** — perintah untuk melakukan sesuatu (mis. `PlaceOrder`). *Imperative*.
- **Producer / Publisher** — service yang mengirim pesan.
- **Consumer / Subscriber** — service yang menerima & memproses pesan.
- **Topic / Queue** — saluran tempat pesan ditampung. *Queue*: satu pesan dikonsumsi satu consumer. *Topic* (pub/sub): satu pesan bisa diterima banyak subscriber.
- **Idempotent Consumer** — consumer aman menerima pesan yang sama berkali-kali tanpa efek berbeda (penting karena broker bisa redeliver).
- **Outbox Pattern** — pola untuk menjamin event terkirim setelah DB transaction commit. Event ditulis ke tabel `outbox` di transaksi yang sama dengan data bisnis; proses relay membaca tabel ini dan publish ke broker.
- **Message Broker** — middleware yang menampung & meneruskan pesan (RabbitMQ, Kafka, NATS, Azure Service Bus).

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Publish event di awal transaksi, baru DB commit di akhir → event terkirim, DB rollback = data tidak konsisten.
- ❌ Consumer tidak dedup pesan → pesan dobel = data dobel.
- ❌ Event tanpa schema versioning → ubah field memecah semua consumer.
</div>

### Aturan / Rules

#### Mandatory — Event-Driven

<span class="badge badge-mandatory">Mandatory</span> Setiap event membawa: event ID, event type, aggregate ID, payload, timestamp.

<span class="badge badge-mandatory">Mandatory</span> Event harus idempotent di sisi consumer — consumer dedup berdasarkan event ID.

#### Optional — Outbox Pattern

<span class="badge badge-optional">Optional</span> Semua domain event ditulis ke tabel *outbox* dalam transaksi yang sama dengan operasi bisnis.

<span class="badge badge-optional">Optional</span> Proses relay khusus membaca outbox dan publish ke broker — memisahkan *write* dari *publish*.

---

## Idempotency

### Apa ini? / What is this?

**ID:** Operasi *idempotent* adalah operasi yang aman dijalankan berkali-kali — hasilnya sama seperti dijalankan sekali. Bayangkan tombol lift: ditekan 5x atau 1x sama saja, lift tetap dipanggil sekali.

**EN:** An *idempotent* operation is safe to run multiple times — the result is the same as running it once. Like a lift button: pressing 5 times or 1 time, the lift is still called once.

### Mengapa penting? / Why it matters?

Di jaringan, request bisa: gagal di tengah jalan, timeout, atau client tidak yakin sudah sampai → client retry. Tanpa idempotency, retry = efek dobel (mis. tagihan dobel, email terkirim dua kali).

### Use Case

> **Skenario:** User klik "Bayar Rp 500.000". Browser kirim request, server mulai proses, tapi koneksi putus sebelum response sampai ke browser. User panik, klik "Bayar" lagi.
>
> Tanpa idempotency: dua payment ter-charge → user marah, refund manual.
>
> Dengan idempotency: client kirim header `Idempotency-Key: abc-123` di kedua request. Server kedua kalinya mendeteksi key yang sama → tidak charge ulang, kembalikan response yang sama.
{ .usecase }

### Istilah & Konsep / Glossary

- **Idempotency Key** — pengenal unik yang diberikan client untuk setiap operasi. Biasanya UUID per *intent* user.
- **At-least-once delivery** — broker bisa mengirim pesan lebih dari satu kali → consumer wajib idempotent.
- **Exactly-once** — sangat sulit dicapai di sistem terdistribusi. *Idempotent consumer* + at-least-once = efek seperti exactly-once.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Mengandalkan client untuk tidak men-retry → pasti gagal cepat atau lambat.
- ❌ Cek key di memory aplikasi → restart proses = key hilang.
- ❌ Simpan key tanpa atomic dengan operasi bisnisnya → race condition.
</div>

### Aturan / Rules

<span class="badge badge-optional">Optional</span> Semua POST dan PUT yang mungkin di-retry harus idempotent.

<span class="badge badge-optional">Optional</span> Client mengirim header `Idempotency-Key` untuk operasi write.

<span class="badge badge-optional">Optional</span> Server menyimpan & menghormati idempotency key minimal 24 jam.

<span class="badge badge-optional">Optional</span> Request duplikat dengan key sama → kembalikan response yang sama dengan request asli, tanpa mengeksekusi side-effect ulang.

<span class="badge badge-optional">Optional</span> Penyimpanan idempotency key bersifat *atomic* dengan operasi yang dijaganya.

---

## Ringkasan / Summary

- **Microservice** memberi keleluasaan scale & deploy mandiri, dengan biaya kompleksitas terdistribusi.
- **Async & concurrency** memungkinkan satu server melayani banyak hal sekaligus, tapi butuh disiplin sinkronisasi.
- **Data access** yang baik = abstraksi (Repository) + pagination + hindari N+1.
- **Cache** memindahkan beban dari DB ke memory; kuncinya TTL & invalidasi.
- **Event-driven** memberi loose coupling; pakai *Outbox Pattern* untuk reliability.
- **Idempotency** melindungi sistem dari retry → operasi penting jangan double-charge.

> **Selanjutnya:** [3. Security](03-security.md) — bagaimana melindungi aplikasi dari ancaman.
