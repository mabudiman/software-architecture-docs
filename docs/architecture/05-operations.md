# 5. Operations
### Operasional / Operations

Setelah aplikasi rilis, kerjaan tidak selesai — justru baru dimulai. Bab ini membahas hal-hal yang membuat aplikasi *bisa dipantau, didiagnosis, dan dipertanggungjawabkan* di production: logging, health check, application performance monitoring (APM), dan audit trail.

Once an application is released, the work isn't done — it's just beginning. This chapter covers what makes an application *monitorable, diagnosable, and accountable* in production: logging, health checks, APM, and audit trail.

---

## Logging

### Apa ini? / What is this?

**ID:** Log adalah catatan kronologis tentang apa yang terjadi di aplikasi: request masuk, error, business event. Tanpa log yang baik, debugging di production = mencari jarum di tumpukan jerami sambil mata tertutup.

**EN:** Logs are chronological records of what happens in the app: incoming requests, errors, business events. Without good logs, production debugging = finding a needle in a haystack blindfolded.

### Mengapa penting? / Why it matters?

- Production tidak punya debugger. Log = satu-satunya jendela ke perilaku aplikasi.
- Structured log (JSON) bisa di-query: "tampilkan semua error dari user 12345 dalam 1 jam terakhir".
- Plain text log = grep manual = lambat & rawan miss.

### Use Case

> **Skenario:** User melaporkan "saya bayar tapi pesanan tidak masuk". Anda perlu menelusuri request user itu di jam tertentu lintas service: API Gateway → Order → Payment → Notification.
>
> Tanpa correlation ID: harus tebak-tebakan, susun timeline manual.
>
> Dengan structured log + correlation ID: `kibana_query: correlation_id="abc-123"` → semua log dari semua service terkait muncul, urut waktu.
{ .usecase }

### Istilah & Konsep / Glossary

- **Structured Logging** — log dalam format yang machine-readable (biasanya JSON), bukan free text. Field bisa di-index & di-query.
- **Log Level** — tingkat kepentingan: **Trace** (sangat detail), **Debug** (debugging), **Info** (event normal), **Warning** (anomali yang tidak fatal), **Error** (gagal).
- **Correlation ID / Trace ID** — pengenal unik untuk satu request yang merambat lewat banyak service → semua log terkait bisa dikaitkan.
- **Log Aggregator** — sistem terpusat yang menerima log dari banyak service: ELK Stack (Elasticsearch + Logstash + Kibana), Grafana Loki, Datadog, Splunk.
- **PII (Personally Identifiable Information)** — data pribadi (nama, email, KTP). Tidak boleh muncul di log mentah.
- **Log Retention** — berapa lama log disimpan. Bergantung regulasi & biaya storage.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ `log("Error")` tanpa konteks → tidak bisa di-debug.
- ❌ Log password / token / nomor kartu kredit.
- ❌ Pakai log level `Info` untuk semua — banjir log, hal penting tertimbun.
- ❌ Plain text log: `"User abc did xyz at 2025"`. Susah dipotong field.
- ❌ Log retention 1 hari di production → kalau bug muncul Senin, log Kamis sudah hilang.
</div>

### Aturan / Rules

#### Mandatory

<span class="badge badge-mandatory">Mandatory</span> Pakai *structured logging* (JSON) — JANGAN plain text string.

<span class="badge badge-mandatory">Mandatory</span> Setiap log entry berisi: timestamp (UTC), log level, service name, correlation/trace ID, message.

<span class="badge badge-mandatory">Mandatory</span> Log level: Error, Warning, Info, Debug, Trace. Level dapat dikonfigurasi runtime via setting tanpa recompile.

#### Confirmation — Log Retention Policy

<span class="badge badge-optional">Optional</span> Definisikan kebijakan retensi log per environment: production minimum 90 hari, development minimum 30 hari.

#### Optional — Prevent Logging Sensitive Data

<span class="badge badge-optional">Optional</span> JANGAN log data sensitif: PII, password, token, nomor kartu kredit.

#### Optional — Centralized Log Aggregator

<span class="badge badge-optional">Optional</span> Semua log dikirim ke log aggregator terpusat (ELK Stack, Grafana Loki, Datadog).

---

## Health Check & Readiness Probe

### Apa ini? / What is this?

**ID:** *Health check* = endpoint khusus yang dipanggil orchestrator (Kubernetes, load balancer) untuk memeriksa apakah aplikasi: (1) masih hidup, (2) siap menerima traffic.

- **Liveness probe** (`/health/live`) — Apakah proses masih hidup & responsif? Jika gagal, orchestrator restart container.
- **Readiness probe** (`/health/ready`) — Apakah aplikasi siap melayani request? (DB tersambung, cache hangat, dll.) Jika gagal, traffic tidak diarahkan ke instance ini.

**EN:** Health check = a dedicated endpoint called by orchestrators (Kubernetes, load balancer) to check if the app is (1) alive and (2) ready for traffic. Liveness → restart if failing. Readiness → don't route traffic if failing.

### Mengapa penting? / Why it matters?

- Saat pod baru start, mungkin butuh 30 detik untuk warming up cache & connect DB. Tanpa readiness, traffic masuk duluan → user lihat error.
- Aplikasi bisa "deadlock" — proses jalan tapi tidak proses request. Tanpa liveness, pod stuck selamanya.

### Use Case

> **Skenario:** Deploy versi baru di Kubernetes. Pod baru start dalam 5 detik, tapi butuh 30 detik untuk inisialisasi connection pool ke DB.
>
> Dengan readiness probe yang mengecek DB: pod baru tidak menerima traffic sampai DB siap → zero downtime.
>
> Tanpa readiness: traffic masuk 5 detik setelah start → error 500 untuk user beruntung yang request di window 25 detik berikutnya.
{ .usecase }

### Istilah & Konsep / Glossary

- **Kubernetes Probe** — Kubernetes secara berkala memanggil endpoint health untuk memutuskan: restart pod (liveness) atau matikan/hidupkan routing (readiness).
- **Startup Probe** — varian khusus untuk aplikasi yang start-nya lama; tidak ditrigger liveness sampai aplikasi "dewasa".
- **Deep Health Check** — periksa semua dependency (DB, cache, queue) → akurat tapi mahal. Jangan terlalu sering.
- **Shallow Health Check** — hanya cek "proses hidup" tanpa cek dependency. Murah, kurang informatif.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Liveness yang mengecek DB → DB hang sebentar = semua pod di-restart oleh K8s = total outage.
- ❌ Health endpoint kembalikan `200 OK` dengan body "FAIL" → orchestrator menganggap sehat (hanya baca status code).
- ❌ Health endpoint butuh autentikasi → orchestrator tidak bisa cek.
- ❌ Satu endpoint `/health` untuk liveness dan readiness → DB maintenance = semua pod di-restart (harusnya cukup stop routing traffic).
</div>

### Perbedaan Liveness vs Readiness

```
Liveness probe (/health/live):              Readiness probe (/health/ready):
┌────────────────────────────────┐           ┌────────────────────────────────┐
│ Cek proses hidup & responsif   │           │ Cek proses + SEMUA dependency │
│ JANGAN cek external dependency │           │ DB, Redis, Message Broker     │
│                                │           │                                │
│ Gagal → Kubernetes RESTART pod │           │ Gagal → STOP routing traffic  │
│                                │           │ Pod tetap hidup, tunggu pulih │
└────────────────────────────────┘           └────────────────────────────────┘
```

**Skenario: DB maintenance window**

```
Dengan satu endpoint /health:
  DB offline → /health 503 → K8s restart SEMUA pod → total outage ❌

Dengan endpoint terpisah:
  DB offline → /health/live 200 (proses OK) → pod tetap hidup
             → /health/ready 503 (DB unreachable) → stop traffic
             → DB kembali → /health/ready 200 → traffic resume ✅
```

### Dependency Health Checking

Readiness probe harus memeriksa semua dependency yang dipakai:

| Dependency | Cara Cek | Timeout |
|-----------|----------|---------|
| Database | `SELECT 1` (lightweight query) | 2-3 detik |
| Redis | `PING` → expect `PONG` | 2-3 detik |
| Message Broker | Cek TCP connection / management API | 2-3 detik |

**Status model per dependency:**

| Status | Arti | Pengaruh HTTP Code |
|--------|------|--------------------|
| `healthy` | Dependency beroperasi normal | Kontribusi ke 200 |
| `degraded` | Reachable tapi lambat/parsial | 200 (dengan warning) atau 503 |
| `unhealthy` | Tidak reachable / error | Kontribusi ke 503 |

**Aturan derivasi status keseluruhan:** status ditentukan oleh dependency terburuk.

### Response Format (IETF `application/health+json`)

```json
{
  "status": "healthy",
  "version": "1.4.2",
  "releaseId": "3f2a1b",
  "checks": {
    "database": [{
      "componentType": "datastore",
      "status": "healthy",
      "time": "2025-11-20T08:45:12Z",
      "responseTime": "12ms"
    }],
    "cache": [{
      "componentType": "datastore",
      "status": "healthy",
      "time": "2025-11-20T08:45:12Z",
      "responseTime": "2ms"
    }],
    "messageBroker": [{
      "componentType": "messagebus",
      "status": "unhealthy",
      "time": "2025-11-20T08:45:12Z",
      "output": "Connection refused: rabbitmq:5672"
    }]
  }
}
```

| Field | Wajib | Deskripsi |
|-------|-------|-----------|
| `status` | ✅ | Overall: `healthy`, `degraded`, atau `unhealthy` |
| `checks` | ✅ | Object dengan satu key per dependency |
| `checks[name][].status` | ✅ | Status per dependency |
| `checks[name][].time` | ✅ | ISO 8601 timestamp |
| `checks[name][].responseTime` | ✅ | Durasi pengecekan |
| `version` | Disarankan | Versi aplikasi |

**JANGAN expose di response:** connection string, password, IP internal, API key, full stack trace.

### HTTP Status Code — Jangan Bohong dengan 200

```
❌ SALAH: HTTP 200 tapi body "unhealthy" → K8s anggap sehat, tetap routing traffic
✅ BENAR: unhealthy → HTTP 503, healthy → HTTP 200
```

| Health Status | HTTP Code |
|---------------|-----------|
| `healthy` | `200 OK` |
| `degraded` | `200 OK` atau `503` (sesuai policy) |
| `unhealthy` | `503 Service Unavailable` |

### Mengapa Tanpa Autentikasi?

Health endpoint **harus** tanpa auth karena:

1. **Circular dependency** — jika auth service down, health check gagal → K8s restart pod yang sebenarnya sehat.
2. **Infrastructure tidak bisa authenticate** — K8s kubelet, load balancer, uptime monitor hanya cek HTTP status code.

**Alternatif security yang direkomendasikan:**

- Network policy / firewall — restrict akses ke IP cluster internal saja.
- Separate port — health di port 8081 (internal), app traffic di port 8080 (via Ingress).
- Response data scoping — hanya info operasional, tanpa data sensitif.

### Startup Probe (Kubernetes)

Untuk service yang start-nya lama (load ML model, migrasi DB), gunakan startup probe agar pod tidak di-kill prematur:

```
Tanpa startup probe:
t=0s  │ Container starts
t=10s │ Liveness probe → FAIL (belum siap) → K8s restart ← SALAH

Dengan startup probe (failureThreshold=30, periodSeconds=10):
t=0s  │ Container starts
t=30s │ Startup probe → success
t=30s │ Liveness & Readiness probe take over → normal
```

### Contoh Implementasi ASP.NET Core

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy(), tags: ["live"])
    .AddSqlServer(connectionString, name: "database", tags: ["ready"])
    .AddRedis(redisConnectionString, name: "cache", tags: ["ready"])
    .AddRabbitMQ(rabbitConnectionString, name: "messageBroker", tags: ["ready"]);

// Map endpoint — pisahkan liveness dan readiness berdasarkan tag
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("live"),
    ResponseWriter = WriteHealthResponse
}).AllowAnonymous();  // Tanpa autentikasi!

app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready"),
    ResponseWriter = WriteHealthResponse
}).AllowAnonymous();
```

### Contoh Implementasi Node.js / Express

```javascript
// Health routes SEBELUM auth middleware
app.use('/health', healthRouter);
app.use(authMiddleware); // Auth diterapkan setelah health routes

// GET /health/live
router.get('/live', (req, res) => {
    res.status(200).json({ status: 'healthy' });
});

// GET /health/ready
router.get('/ready', async (req, res) => {
    const checks = await runReadinessChecks(); // Cek DB, Redis, Broker
    const status = deriveOverallStatus(checks);
    res.status(status === 'unhealthy' ? 503 : 200).json({ status, checks });
});
```

### Kubernetes Deployment Configuration

```yaml
# k8s/deployment.yaml
spec:
  containers:
    - name: order-service
      image: order-service:1.4.2
      ports:
        - containerPort: 8080

      # Startup probe — beri waktu container untuk inisialisasi
      startupProbe:
        httpGet:
          path: /health/live
          port: 8080
        failureThreshold: 30
        periodSeconds: 10

      # Liveness — restart jika proses freeze
      livenessProbe:
        httpGet:
          path: /health/live
          port: 8080
        periodSeconds: 30
        timeoutSeconds: 5
        failureThreshold: 3

      # Readiness — stop traffic jika dependency gagal
      readinessProbe:
        httpGet:
          path: /health/ready
          port: 8080
        periodSeconds: 10
        timeoutSeconds: 5
        failureThreshold: 3
        successThreshold: 1
```

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Setiap service ekspos: `/health/live` (liveness) dan `/health/ready` (readiness).

<span class="badge badge-mandatory">Mandatory</span> *Liveness*: konfirmasi proses hidup & tidak deadlock.

<span class="badge badge-mandatory">Mandatory</span> *Readiness*: konfirmasi semua dependency (DB, cache, broker) terjangkau sebelum menerima traffic.

<span class="badge badge-mandatory">Mandatory</span> Endpoint health TIDAK memerlukan autentikasi.

<span class="badge badge-mandatory">Mandatory</span> Response berisi status dependency dalam format terstruktur.

<span class="badge badge-mandatory">Mandatory</span> JANGAN kembalikan HTTP 200 dengan body error — status code harus akurat mencerminkan health.

---

## Application Performance Monitoring (APM)

### Apa ini? / What is this?

**ID:** APM adalah praktik & alat untuk memantau performa aplikasi secara real-time: berapa cepat, berapa banyak error, di mana bottleneck. Mencakup *metrics*, *tracing*, dan kadang *profiling*.

**EN:** APM is the practice & tooling for monitoring application performance in real-time: how fast, how many errors, where the bottlenecks are. Covers *metrics*, *tracing*, sometimes *profiling*.

### Mengapa penting? / Why it matters?

- Tanpa APM, Anda tahu "ada yang lambat" tapi tidak tahu *di mana*.
- Alert otomatis = tahu masalah sebelum user komplain.
- Distributed tracing = lihat request "naik turun" lewat banyak service → temukan service yang lambat.

### Use Case

> **Skenario:** User komplain "checkout lambat". Aplikasi terdiri dari 8 service. Tanpa tracing: tim debug satu per satu service → 3 hari.
>
> Dengan distributed tracing (OpenTelemetry): buka trace ID di Jaeger / Tempo → lihat timeline: Order Service 50ms, Payment Service 200ms, **Inventory Service 4500ms** (akar masalah). Issue dipinpoint dalam menit.
{ .usecase }

### Istilah & Konsep / Glossary

- **Metrics** — angka deret-waktu (mis. request/detik, CPU%, memory usage).
- **RED Method** — tiga metrik dasar untuk service: **R**ate (req/s), **E**rrors (error rate), **D**uration (latency).
- **USE Method** — untuk resource: **U**tilization, **S**aturation, **E**rrors. Cocok untuk node/disk/network.
- **Prometheus** — sistem metrics open-source paling populer. Aplikasi ekspos endpoint `/metrics` text, Prometheus *scrape* berkala.
- **Grafana** — UI untuk visualisasi metrics (dari Prometheus, Loki, dll).
- **Distributed Tracing** — melacak satu request lewat banyak service, lengkap dengan timing setiap hop.
- **OpenTelemetry** — standar industri & SDK untuk instrumentasi metrics, traces, logs. Vendor-agnostic.
- **Trace ID / Span ID** — Trace = satu request end-to-end. Span = satu operasi dalam trace. Span punya parent (span pemanggil).
- **`traceparent`** — header HTTP standar (W3C) untuk membawa trace info antar service.
- **Sampling Rate** — persentase request yang di-trace. 100% di staging, mungkin 5-10% di production (mahal jika 100%).
- **Runbook** — panduan operasional langkah-demi-langkah untuk respon insiden tertentu (mis. "kalau alert DB-Connection-Pool-Full menyala, lakukan ini").
- **SLI / SLO / SLA** — Service Level Indicator / Objective / Agreement. Indikator yang diukur (latency), target internal (p95 < 300ms), janji ke pelanggan.

### Aturan / Rules

#### Optional — Observability (APM / Metrics)

<span class="badge badge-optional">Optional</span> Instrumentasi setiap service dengan RED metrics: Rate, Errors, Duration.

<span class="badge badge-optional">Optional</span> Ekspos metrics dalam format standar (mis. Prometheus `/metrics`).

<span class="badge badge-optional">Optional</span> Definisikan threshold alerting untuk setiap metrik kritis sebelum production.

<span class="badge badge-optional">Optional</span> Dashboard dirawat sebagai code (mis. Grafana provisioning) — JANGAN konfigurasi manual.

<span class="badge badge-optional">Optional</span> Runbook on-call ter-link dari setiap alert definition.

#### Optional — Distributed Tracing

<span class="badge badge-optional">Optional</span> Implementasi OpenTelemetry di semua service.

<span class="badge badge-optional">Optional</span> Setiap request inbound generate trace ID. Propagasi ke downstream lewat header standar (`traceparent`).

<span class="badge badge-optional">Optional</span> Injeksi correlation/trace ID ke setiap log entry.

<span class="badge badge-optional">Optional</span> Trace sampling rate dapat dikonfigurasi per environment (100% staging, sampled production).

<span class="badge badge-optional">Optional</span> Backend tracing menyimpan trace minimum 7 hari.

---

## Audit Trail

### Apa ini? / What is this?

**ID:** Audit trail = catatan permanen tentang **siapa melakukan apa, kapan, dan dengan hasil apa**. Berbeda dari log biasa, audit trail tidak boleh dimanipulasi & disimpan jauh lebih lama (regulasi).

**EN:** Audit trail = permanent record of **who did what, when, and with what result**. Unlike regular logs, audit trail must be tamper-resistant and kept much longer (regulation).

### Mengapa penting? / Why it matters?

- Forensik insiden: "siapa yang menghapus tabel `orders`?" → audit trail wajib.
- Compliance: GDPR, PCI-DSS, OJK butuh audit trail.
- Investigasi penipuan internal.

### Use Case

> **Skenario:** Sejumlah saldo wallet user dilaporkan hilang. Investigator perlu tahu:
>
> 1. Siapa user/admin yang melakukan operasi?
> 2. Operasi apa? (transfer, refund, manual adjustment?)
> 3. Kapan persis?
> 4. Dari IP / endpoint mana?
> 5. Berhasil atau gagal?
>
> Dengan audit trail lengkap → kasus selesai dalam jam. Tanpa → kasus tidak terlacak.
{ .usecase }

### Istilah & Konsep / Glossary

- **Immutable Store** — tempat penyimpanan yang tidak mengizinkan UPDATE atau DELETE. Biasanya implementasi: append-only DB, atau permission-level constraint.
- **Append-only** — hanya bisa tambah data, tidak bisa ubah/hapus.
- **Tamper Evidence** — bukti bahwa data tidak diubah (mis. hash chain, signature).
- **Before/After Value** — saat data berubah, catat nilai lama dan nilai baru → bisa rekonstruksi.

### Aturan / Rules

<span class="badge badge-optional">Optional</span> Catat untuk setiap operasi signifikan: **who** (user/service), **what** (action & resource), **when** (UTC timestamp), **where** (resource ID, endpoint), **result** (success/failure), **origin** (IP, correlation ID).

<span class="badge badge-optional">Optional</span> Audit record ditulis ke *immutable store* — TIDAK boleh UPDATE / DELETE pada audit row.

<span class="badge badge-optional">Optional</span> Audit trail dapat di-query berdasarkan: user, resource, time range, action type.

<span class="badge badge-optional">Optional</span> Nilai field sensitif (before/after perubahan data) dimasukkan jika relevan.

<span class="badge badge-optional">Optional</span> Retensi audit log mengikuti regulasi yang berlaku (rekomendasi minimum 1 tahun).

---

## Ringkasan / Summary

- **Structured logging + correlation ID** = debugging tetap mungkin di production multi-service.
- **Liveness vs Readiness** punya peran berbeda — jangan campur (terutama jangan masukkan cek DB ke liveness).
- **RED metrics + distributed tracing** = peta untuk menemukan bottleneck dalam menit, bukan hari.
- **Audit trail** = catatan untuk forensik & compliance, beda dari log operasional biasa.

> **Selanjutnya:** [6. User Interface](06-user-interface.md) — standar untuk membangun UI yang konsisten & dapat diakses semua orang.
