# 4. Quality & Reliability
### Kualitas & Keandalan / Quality & Reliability

Bagaimana memastikan kode yang Anda tulis tidak hanya *jalan*, tapi juga *bisa diandalkan* di production? Jawabannya: testing yang sistematis, kode yang bersih, penanganan edge case yang sadar, dan exception handling yang tegas.

How do you ensure the code you write doesn't just *work* but is also *reliable* in production? The answers: systematic testing, clean code, deliberate edge case handling, and disciplined exception handling.

---

## Testing Management

### Apa ini? / What is this?

**ID:** Testing bukan satu hal — ia keluarga praktik dengan tujuan berbeda:

- **Unit test** — uji satu unit kecil (function/class) terisolasi.
- **Integration test** — uji interaksi beberapa komponen nyata.
- **Contract test** — pastikan service-A & service-B sepakat tentang format komunikasi mereka.
- **E2E test (End-to-End)** — uji alur dari sudut pandang user, di environment lengkap.
- **BDD** — menulis test dalam bahasa bisnis agar BA & developer paham.
- **Performance / Load / Stress / Chaos** — menguji aplikasi di bawah beban & gangguan.

**EN:** Testing isn't one thing — it's a family of practices with different goals (unit, integration, contract, E2E, BDD, performance/load/stress/chaos).

Analogi piramida test: Banyak unit test (cepat, murah) di dasar, sedikit integration test di tengah, sangat sedikit E2E test di puncak (lambat, mahal).

### Mengapa penting? / Why it matters?

- Tanpa test, setiap perubahan = lotere. Bug yang sama kembali (regresi).
- Test = dokumentasi yang selalu update — perilaku kode terbaca dari test.
- Test cepat → developer berani refactor → kode tetap sehat seiring waktu.

### Use Case

> **Skenario:** Anda menambah fitur diskon. Setelah deploy, ternyata diskon ter-double untuk member VIP. Tanpa test → bug terdeteksi user, harga rugi. Dengan unit test untuk hitung diskon + integration test untuk flow checkout + BDD scenario "Member VIP dapat diskon X" → bug tertangkap di CI.
{ .usecase }

### Istilah & Konsep / Glossary

- **AAA Pattern (Arrange-Act-Assert)** — pola struktur test: siapkan data → jalankan operasi → cek hasil.
- **Mock / Stub / Fake** — pengganti dependency saat test. *Mock*: cek interaksi. *Stub*: kembalikan nilai tertentu. *Fake*: implementasi sederhana yang berfungsi (mis. in-memory repository).
- **Test Coverage** — persentase baris kode yang dijalankan saat test. Bukan jaminan kualitas, tapi indikator.
- **Flaky Test** — test yang kadang lulus kadang gagal tanpa kode berubah. Lebih buruk dari tidak ada test.
- **BDD (Behavior-Driven Development)** — menulis test sebagai skenario behavior dalam bahasa bisnis.
- **Gherkin** — sintaks BDD: `Given <kondisi awal> When <aksi> Then <hasil>`.
- **Feature File** — file `.feature` yang berisi skenario Gherkin.
- **Testcontainers** — library untuk menjalankan dependency nyata (DB, queue) di container saat test → tidak perlu mock infrastruktur.
- **Consumer-Driven Contract** — consumer menulis ekspektasinya tentang API; provider memverifikasi. Tool: Pact.
- **Pact Broker** — repository pusat untuk menyimpan kontrak antar service.
- **p50 / p95 / p99** — *median*, *95th percentile*, *99th percentile* latency. p95 = "95% request lebih cepat dari ini".
- **Baseline** — angka performa acuan untuk dibandingkan dengan build berikutnya.
- **Steady-state Hypothesis** — di chaos engineering: hipotesis "sistem dalam kondisi normal X" yang diuji apakah tetap berlaku saat ada gangguan.
- **Blast Radius** — luas dampak sebuah experiment chaos.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Test yang share state global → urutan eksekusi mempengaruhi hasil.
- ❌ Mock yang terlalu ketat → setiap refactor pecahkan banyak test (rapuh).
- ❌ E2E test untuk segala hal → suite lambat, flaky, susah maintain.
- ❌ Disable flaky test tanpa investigasi → bug nyata tersembunyi.
- ❌ Coverage 100% target → bisa dikejar dengan test sampah; fokus pada *meaningful coverage*.
</div>

### Aturan / Rules

#### Mandatory — Testing Standards

<span class="badge badge-mandatory">Mandatory</span> Konvensi nama test: `MethodName_Scenario_ExpectedResult` (mis. `PlaceOrder_WithInsufficientStock_ThrowsBusinessException`).

<span class="badge badge-mandatory">Mandatory</span> Test harus independen — tidak ada shared mutable state antar test.

<span class="badge badge-mandatory">Mandatory</span> Test JANGAN akses data atau infrastruktur production.

<span class="badge badge-mandatory">Mandatory</span> Semua environment test reproducible dari code — tidak ada setup manual.

<span class="badge badge-mandatory">Mandatory</span> Flaky test harus diinvestigasi & diperbaiki segera — jangan di-disable tanpa tracked issue.

<span class="badge badge-mandatory">Mandatory</span> Coverage report di-generate & dipublikasikan setiap CI run.

#### Mandatory — BDD

<span class="badge badge-mandatory">Mandatory</span> Tulis skenario dalam format Gherkin: Given / When / Then.

<span class="badge badge-mandatory">Mandatory</span> Judul skenario pakai bahasa bisnis, bukan implementasi teknis.

<span class="badge badge-mandatory">Mandatory</span> Satu skenario per aturan bisnis.

<span class="badge badge-mandatory">Mandatory</span> Feature file dimiliki & di-review developer + business analyst.

<span class="badge badge-mandatory">Mandatory</span> Simpan semua `.feature` file di `/specs`.

<span class="badge badge-mandatory">Mandatory</span> Skenario harus eksekutabel & terhubung ke step definition otomatis.

<span class="badge badge-mandatory">Mandatory</span> JANGAN tulis skenario yang mendeskripsikan interaksi UI — fokus pada behavior bisnis.

#### Mandatory — Unit Test

<span class="badge badge-mandatory">Mandatory</span> Setiap public method punya minimal satu unit test.

<span class="badge badge-mandatory">Mandatory</span> Test terisolasi — tidak ada DB, network, file system. Mock semua external dependency.

<span class="badge badge-mandatory">Mandatory</span> Ikuti AAA: Arrange, Act, Assert.

<span class="badge badge-mandatory">Mandatory</span> Coverage minimum: 80% per service, 95% untuk domain layer.

<span class="badge badge-mandatory">Mandatory</span> Test harus deterministik — tidak ada random data, tidak bergantung urutan eksekusi.

<span class="badge badge-mandatory">Mandatory</span> Pakai builder pattern atau object mother untuk konstruksi data test.

<span class="badge badge-mandatory">Mandatory</span> JANGAN magic string / magic number — pakai named constant.

#### Mandatory — Integration Test

<span class="badge badge-mandatory">Mandatory</span> Uji integrasi antara ≥2 komponen nyata (service + DB, service + broker).

<span class="badge badge-mandatory">Mandatory</span> Pakai *Testcontainers* untuk dependency infra nyata — JANGAN mock infra di integration test.

<span class="badge badge-mandatory">Mandatory</span> Setiap test setup & teardown datanya sendiri. JANGAN andalkan data pre-existing.

<span class="badge badge-mandatory">Mandatory</span> Jalankan di environment terisolasi — JANGAN ke production atau staging shared.

<span class="badge badge-mandatory">Mandatory</span> Cakup minimum: happy path, edge case yang dikenal, failure scenario per integration point.

#### Mandatory — Contract Test

<span class="badge badge-mandatory">Mandatory</span> Setiap service yang konsumsi API punya consumer-driven contract test (mis. Pact).

<span class="badge badge-mandatory">Mandatory</span> Provider memverifikasi semua kontrak consumer pada setiap build.

<span class="badge badge-mandatory">Mandatory</span> Contract test jalan SEBELUM integration test di pipeline CI.

<span class="badge badge-mandatory">Mandatory</span> Memecah kontrak = build provider gagal.

<span class="badge badge-mandatory">Mandatory</span> Kontrak yang dipublikasikan diberi versi & disimpan di contract broker.

#### Mandatory — E2E Test

<span class="badge badge-mandatory">Mandatory</span> Cakup hanya critical user journey — JANGAN duplikasi unit/integration scenario.

<span class="badge badge-mandatory">Mandatory</span> Jalankan di environment terdeploy penuh (staging), bukan localhost.

<span class="badge badge-mandatory">Mandatory</span> Pakai akun & data test khusus & stabil — JANGAN pakai data user nyata.

<span class="badge badge-mandatory">Mandatory</span> E2E test idempotent — eksekusi berulang tidak menyebabkan side effect.

<span class="badge badge-mandatory">Mandatory</span> Durasi suite E2E maksimal 15 menit. Optimasi/paralelisasi jika lewat.

<span class="badge badge-mandatory">Mandatory</span> Kegagalan harus menghasilkan screenshot & log request/response untuk diagnosis.

#### Optional — Performance Test

<span class="badge badge-optional">Optional</span> Definisikan & dokumentasikan baseline target performa sebelum test pertama (mis. p95 < 300ms).

<span class="badge badge-optional">Optional</span> Simulasikan load & pola pemakaian realistis — bukan spike sintetis.

<span class="badge badge-optional">Optional</span> Jalankan di environment yang menyerupai production dalam skala.

<span class="badge badge-optional">Optional</span> Ukur: response time (p50, p95, p99), throughput (req/s), error rate.

<span class="badge badge-optional">Optional</span> Regresi performa (>10% lebih lambat dari baseline) memblok release.

<span class="badge badge-optional">Optional</span> Hasil disimpan & dibandingkan antar build — JANGAN dievaluasi terisolasi.

#### Optional — Load / Stress Test

<span class="badge badge-optional">Optional</span> *Load test*: validasi sistem di beban puncak yang diharapkan.

<span class="badge badge-optional">Optional</span> *Stress test*: cari breaking point — naikkan beban sampai sistem gagal.

<span class="badge badge-optional">Optional</span> Dokumentasikan maximum safe throughput & failure mode (graceful degradation vs crash).

<span class="badge badge-optional">Optional</span> Sistem harus pulih otomatis setelah beban turun — tanpa restart manual.

<span class="badge badge-optional">Optional</span> Jalankan minimal sekali sebelum setiap major release & setelah perubahan infra signifikan.

#### Optional — Chaos / Resilience Test

<span class="badge badge-optional">Optional</span> Definisikan katalog chaos experiment: pod kill, network partition, high CPU, disk full, dependency timeout.

<span class="badge badge-optional">Optional</span> Jalankan chaos di staging dulu; lanjut ke production hanya setelah steady-state hypothesis ditegakkan.

<span class="badge badge-optional">Optional</span> Setiap experiment punya: hypothesis, blast radius, rollback procedure, success criteria.

<span class="badge badge-optional">Optional</span> Sistem harus tetap observable (metrics, logs, traces) selama & setelah experiment.

<span class="badge badge-optional">Optional</span> Temuan diumpan-balik ke perbaikan resilience & di-test ulang setelah fix.

---

## Code Quality

### Apa ini? / What is this?

**ID:** Kode berkualitas = kode yang mudah dibaca, mudah diubah, dan minim bug. Bukan tentang *clever*, tapi tentang *clear*.

**EN:** Quality code = easy to read, easy to change, fewer bugs. Not about *clever*, but about *clear*.

### Mengapa penting? / Why it matters?

Developer **membaca kode 10x lebih banyak daripada menulisnya**. File 2000 baris atau function 200 baris yang mengerjakan 10 hal sekaligus = beban kognitif besar, tempat bug bersembunyi.

### Use Case

> **Skenario:** Function `processOrder()` 500 baris, nesting 6 level (`if` di dalam `for` di dalam `if`...). Saat ditugaskan menambah fitur diskon, developer butuh 2 hari hanya untuk memahami flow → tinggi risiko bug.
>
> Setelah refactor: function jadi 30 baris yang memanggil 5 function lebih kecil (validateStock, applyDiscount, ...). Tambah fitur baru hanya butuh 2 jam.
{ .usecase }

### Istilah & Konsep / Glossary

- **Single Responsibility** — satu function/class punya satu alasan untuk berubah.
- **Guard Clause / Early Return** — `if (invalid) return;` di awal function → hindari nesting dalam.
- **Magic Number / String** — angka/string literal di tengah kode tanpa penjelasan (mis. `if (status == 7)` — kenapa 7?). Gantikan dengan constant bernama.
- **Cyclomatic Complexity** — ukuran kompleksitas alur. Banyak branch = sulit di-test & dipahami.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Function 500 baris yang melakukan validasi, perhitungan, dan kirim email sekaligus.
- ❌ Nesting `if` 6 level dalam.
- ❌ `if (type == "A")` — apa itu "A"? Magic string.
- ❌ Variable bernama `x`, `tmp`, `data2`.
</div>

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Setiap file maksimal 500 baris. Lebih dari itu = sinyal harus refactor.

<span class="badge badge-mandatory">Mandatory</span> Setiap function/method melakukan satu hal — single responsibility di level function.

<span class="badge badge-mandatory">Mandatory</span> Function maksimal 40 baris.

<span class="badge badge-mandatory">Mandatory</span> Nesting maksimal 3 level. Flatten dengan *early return* atau ekstraksi function.

<span class="badge badge-mandatory">Mandatory</span> Ikuti konvensi penamaan yang disepakati secara konsisten di seluruh codebase.

<span class="badge badge-mandatory">Mandatory</span> JANGAN ada magic string / magic number — pakai named constant.

---

## Edge Case Handling

### Apa ini? / What is this?

**ID:** *Edge case* = kondisi tidak biasa di pinggir input space: nilai null, batas atas/bawah, list kosong, payload rusak, akses bersamaan. Mereka jarang muncul tapi saat muncul = bug serius.

**EN:** *Edge cases* = unusual conditions at the boundary of the input space: null values, min/max bounds, empty lists, malformed payloads, concurrent access. Rare but when they happen = serious bugs.

### Mengapa penting? / Why it matters?

90% bug produksi datang dari skenario yang **tidak terbayangkan** saat coding. Mengidentifikasi edge case di *fase desain* (bukan setelah implementasi) jauh lebih murah.

### Use Case

> **Skenario:** API checkout. Test bekerja sempurna untuk cart berisi 1-10 item. Production: ada user yang men-checkout cart KOSONG (0 item) → NullPointerException, 500 error. Edge case sederhana yang tidak dipikirkan.
{ .usecase }

### Istilah & Konsep / Glossary

- **Boundary Value** — nilai di batas (min, max, kosong, nol, negatif, sangat besar).
- **Concurrent Modification** — dua proses ubah data yang sama bersamaan → siapa menang? Pakai *optimistic locking* atau *pessimistic locking*.
- **Defensive Programming** — menulis kode dengan asumsi input bisa salah, jaringan bisa putus, dependency bisa down.

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Identifikasi & dokumentasikan edge case saat *desain* — bukan setelah implementasi.

<span class="badge badge-mandatory">Mandatory</span> Semua input divalidasi di *boundary* (API layer) sebelum masuk Application layer.

<span class="badge badge-mandatory">Mandatory</span> JANGAN asumsikan input eksternal valid, well-formed, atau dalam range yang diharapkan.

<span class="badge badge-mandatory">Mandatory</span> Tangani: null/empty, boundary value (min/max), enum value tak terduga, payload rusak, modifikasi bersamaan.

<span class="badge badge-mandatory">Mandatory</span> Skenario edge case punya unit/integration test terkait.

---

## Exception Handling

### Apa ini? / What is this?

**ID:** *Exception* (pengecualian) = sinyal bahwa kondisi tidak normal terjadi. *Exception handling* = strategi untuk menangani sinyal ini agar aplikasi tidak crash sembarangan & user tidak melihat error mentah.

**EN:** An *exception* signals that something abnormal happened. *Exception handling* = strategy for dealing with these signals so the app doesn't crash carelessly and users don't see raw errors.

### Mengapa penting? / Why it matters?

- Stack trace ke client = mengungkap struktur internal aplikasi → keamanan!
- Tanpa global handler → setiap exception bisa muncul dengan format berbeda → client sulit menangani.
- Error message yang berinformasi untuk developer (di log) tapi aman & generik untuk user.

### Use Case

> **Skenario:** User mencoba ambil resource yang tidak ada (`GET /orders/999`). Tanpa handling: framework mengembalikan HTML stack trace dengan path file, versi library, query SQL → penyerang dapat informasi berharga.
>
> Dengan handling: server mengembalikan `404` dengan body Problem Details:
> ```json
> { "type": "https://example.com/errors/not-found",
>   "title": "Order not found",
>   "status": 404,
>   "detail": "Order with ID 999 does not exist." }
> ```
> Dan log di server menyimpan detail lengkap untuk debugging.
{ .usecase }

### Istilah & Konsep / Glossary

- **Global Exception Handler / Middleware** — satu tempat di aplikasi yang menangkap semua exception yang tidak ditangani secara lokal, mengubahnya jadi response standar.
- **Problem Details (RFC 7807)** — standar format JSON untuk error API: field `type`, `title`, `status`, `detail`, `instance`.
- **Stack Trace** — daftar pemanggilan function yang menuju ke error. Sangat berguna untuk developer, sangat tidak boleh untuk dilihat user.
- **Custom Exception** — kelas exception khusus untuk kasus bisnis (mis. `InsufficientStockException`). Memudahkan handling spesifik.
- **Domain vs Infrastructure Exception** — pisahkan exception bisnis (validation, business rule) dari teknis (DB down, network error). Penanganannya beda.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ `catch (Exception e) {}` (swallow exception) → bug tersembunyi.
- ❌ Return stack trace ke client.
- ❌ Throw `Exception` umum tanpa info — log "Error" saja.
- ❌ Format error berbeda di setiap endpoint.
</div>

### Aturan / Rules

#### Mandatory

<span class="badge badge-mandatory">Mandatory</span> Semua unhandled exception ditangkap *global exception handler / middleware* — JANGAN biarkan raw exception sampai ke client.

<span class="badge badge-mandatory">Mandatory</span> JANGAN ekspos stack trace ke client. Log di server-side; kembalikan response error terstruktur.

<span class="badge badge-mandatory">Mandatory</span> Pakai format *Problem Details* (RFC 7807) untuk semua API error response.

<span class="badge badge-mandatory">Mandatory</span> Pesan exception informatif untuk developer (di log) & aman/generik untuk end user (di response).

#### Optional — Use Custom Exception

<span class="badge badge-optional">Optional</span> Definisikan hirarki custom exception: `BaseException` → `DomainException`, `InfrastructureException`, `ApplicationException`.

<span class="badge badge-optional">Optional</span> Di bawah `DomainException`: `BusinessRuleException`, `NotFoundException`, `ValidationException`.

---

## Ringkasan / Summary

- **Test piramida**: banyak unit (dasar) → integration → sedikit E2E (puncak).
- **BDD** menjembatani developer & business analyst dengan bahasa Gherkin.
- **Code quality**: file ≤500 baris, function ≤40 baris, nesting ≤3 level, no magic.
- **Edge case** dipikirkan di *desain*, bukan setelah implementasi.
- **Exception handling** terstandar lewat global handler + Problem Details.

> **Selanjutnya:** [5. Operations](05-operations.md) — bagaimana mengoperasikan & memantau aplikasi setelah deploy.
