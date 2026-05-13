# 1. Foundation
### Fondasi Arsitektur / Architectural Foundation

Bab ini membahas hal paling dasar dalam menyusun proyek perangkat lunak: bagaimana folder ditata, bagaimana kode dipisah ke dalam *lapisan*, dan bagaimana kita mencatat keputusan-keputusan penting sehingga tim yang datang kemudian tahu *kenapa* sebuah pilihan dibuat.

This chapter covers the most fundamental aspects of setting up a software project: how folders are organized, how code is separated into *layers*, and how we record important decisions so future team members understand *why* a choice was made.

---

## Struktur Folder Dasar / Basic Folder Structure

### Apa ini? / What is this?

**ID:** Struktur folder adalah cara kita menata file & folder di sebuah proyek agar konsisten dan mudah ditemukan. Bayangkan seperti tata letak rak di perpustakaan — buku sains di rak A, novel di rak B. Tanpa tata letak yang disepakati, setiap orang akan menaruh barang di tempat berbeda, dan menemukan sesuatu jadi mimpi buruk.

**EN:** A folder structure is how we organize files & folders in a project so they're consistent and easy to find. Think of it like a library's shelf layout — science books on shelf A, novels on shelf B. Without an agreed layout, everyone puts things in different places, and finding anything becomes a nightmare.

### Mengapa penting? / Why it matters?

- Developer baru bisa cepat memahami isi proyek.
- Tools (CI/CD, linter, test runner) bisa menemukan file di lokasi yang dapat diprediksi.
- Mengurangi konflik saat banyak orang bekerja di proyek yang sama.
- Memisahkan kode aplikasi, dokumentasi, infrastruktur, dan skrip → tidak tercampur dalam satu folder besar.

### Use Case

> **Skenario:** Tim Anda menerima developer baru. Hari pertama, ia perlu menjawab: "Di mana saya menambahkan endpoint API baru? Di mana spec test ditulis? Bagaimana cara men-deploy ke staging?"
>
> Dengan struktur folder standar, ia bisa langsung melihat `src/` untuk kode, `specs/` untuk spec test, `infra/` untuk konfigurasi deploy, dan `docs/` untuk dokumentasi. Tanpa struktur jelas, ia harus tanya senior berulang kali → menghambat onboarding.
{ .usecase }

### Istilah & Konsep / Glossary

- **`src/`** — singkatan *source*, berisi kode aplikasi utama (production code).
- **`infra/`** — singkatan *infrastructure*, berisi file konfigurasi server, container, Kubernetes manifest, Terraform, dll.
- **`specs/`** — folder untuk *specification* — biasanya berisi file `.feature` (Gherkin) untuk BDD test, atau dokumen spesifikasi fungsional.
- **`adr/`** — *Architecture Decision Records*, catatan keputusan arsitektur yang pernah diambil. Akan dibahas di bagian berikutnya.
- **`scripts/`** — skrip bantu (mis. backup, seeder data, migrasi manual).
- **`.github/`** — folder khusus GitHub berisi workflow CI/CD, template issue, dll.

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Setiap proyek menggunakan struktur folder dasar yang disepakati sebelum coding dimulai.

<span class="badge badge-mandatory">Mandatory</span> Folder spesifik (`src/`, `infra/`, `docs/`, `specs/`) tidak dicampur isinya.

### Contoh Struktur / Example

| Folder / File | Deskripsi |
|---|---|
| `project-root/` | Root folder proyek |
| `├── .github/workflows/` | CI/CD pipeline (GitHub Actions) |
| `├── .github/instructions/` | Instruksi khusus untuk AI assistant / Copilot |
| `├── .github/skills/` | Skill definition untuk AI assistant |
| `├── docs/adr/` | Architecture Decision Records |
| `├── docs/api/` | API spec (OpenAPI / Swagger) |
| `├── src/` | Kode aplikasi utama (production code) |
| `├── scripts/` | Skrip bantu (backup, seeder, migrasi manual) |
| `├── specs/` | BDD spec / feature files (Gherkin) |
| `├── infra/` | Konfigurasi deploy & infrastruktur (Docker, K8s, Terraform) |
| `├── .vscode/` | Konfigurasi editor — opsional, sebagian tim `.gitignore` |
| `├── docker-compose.yml` | Definisi container untuk local development |
| `└── readme.md` | Dokumentasi utama proyek |

---

## Clean Architecture

### Apa ini? / What is this?

**ID:** Clean Architecture adalah cara membagi kode menjadi *lapisan-lapisan* (layer) dengan aturan ketat: lapisan dalam tidak boleh tahu apa-apa tentang lapisan luar. Bayangkan seperti bawang — di tengah ada *core* bisnis (paling penting), dikelilingi lapisan-lapisan teknis (framework, database, UI).

**EN:** Clean Architecture is a way of dividing code into *layers* with strict rules: inner layers must not know anything about outer layers. Imagine an onion — the business *core* sits at the center (most important), surrounded by technical layers (framework, database, UI).

Lapisan utamanya / The main layers:

1. **Domain** (inti) — aturan & entitas bisnis murni, tanpa framework.
2. **Application** — orkestrasi *use case* (alur cerita aplikasi).
3. **Infrastructure** — implementasi teknis (database, HTTP client, queue).
4. **Presentation** — yang dilihat pengguna atau di-expose ke dunia luar (REST API, gRPC, UI).

Aturan emas: **arah ketergantungan selalu menuju ke dalam.** Presentation tahu Application, Application tahu Domain. Tapi Domain TIDAK tahu apa-apa di luar dirinya.

### Mengapa penting? / Why it matters?

- **Domain bisa diuji tanpa database, tanpa HTTP, tanpa framework** → unit test cepat & deterministik.
- **Mengganti teknologi tidak meledakkan seluruh kode.** Migrasi dari MySQL ke PostgreSQL? Cukup ganti adapter di Infrastructure — Domain tidak tersentuh.
- **Logika bisnis tidak "bocor" ke controller atau ke SQL query** → mudah dilacak & dirawat.

### Use Case

> **Skenario:** Aplikasi e-commerce Anda awalnya pakai REST API + MySQL. Setahun kemudian, mobile team minta dukungan gRPC + ada kebutuhan migrasi ke PostgreSQL.
>
> Dengan Clean Architecture: tambahkan adapter gRPC baru di Presentation, ganti driver database di Infrastructure. **Domain & Application tidak berubah** karena aturan bisnis tetap sama.
>
> Tanpa Clean Architecture (kode bisnis bercampur di controller & SQL): perubahan ini bisa jadi proyek 3 bulan dengan banyak bug, karena logika "diskon untuk member" tersebar di puluhan file controller.
{ .usecase }

### Istilah & Konsep / Glossary

- **Entity** — objek bisnis utama yang punya *identitas* (mis. `User` dengan ID, `Order` dengan order number). Dua entity berbeda meskipun isinya sama jika ID-nya beda.
- **Value Object** — objek yang ditentukan oleh *nilainya*, bukan identitas. Mis. `Money(amount=100, currency=USD)`. Dua value object dengan nilai sama = sama. Selalu *immutable* (tidak berubah setelah dibuat).
- **Aggregate Root** — entity utama yang menjadi pintu masuk untuk memodifikasi sekumpulan objek terkait. Mis. untuk mengubah `OrderItem`, harus melalui `Order` (aggregate root). Ini mencegah data inkonsisten.
- **Domain Event** — peristiwa penting di domain (mis. `OrderPlaced`, `PaymentReceived`). Layanan lain bisa "mendengarkan" event ini dan bereaksi. Selalu dinamai dalam bentuk *past tense* dan bersifat *immutable*.
- **Repository** — abstraksi untuk menyimpan & mengambil entity. Di Domain hanya didefinisikan *interface*-nya; implementasinya (pakai SQL, MongoDB, file) ada di Infrastructure. Repository bekerja dengan *Aggregate Root*, bukan entity internal.
- **Domain Service** — logika domain yang tidak pas masuk ke entity tertentu (mis. `TransferService` yang melibatkan dua `Account`, atau `PricingService` yang menghitung diskon + voucher + loyalty).
- **DDD (Domain-Driven Design)** — pendekatan desain yang menempatkan model bisnis (Domain) sebagai pusat.
- **ArchUnit / NetArchTest** — library untuk menulis *test* yang memverifikasi aturan arsitektur (mis. "Domain tidak boleh mengimport package Infrastructure").
- **Inversion of Control (IoC)** — Domain mendefinisikan *interface* yang dibutuhkan; Infrastructure *mengimplementasikan* interface tersebut. Dependensi konkret di-inject oleh DI container di Composition Root.

### Anti-pattern

<div class="antipattern" markdown>
**Yang sering salah:**

- ❌ Domain mengimport library ORM (mis. `from sqlalchemy import Column` di entity) → Domain tergantung framework.
- ❌ Controller menulis SQL langsung → logika bisnis bocor ke Presentation.
- ❌ Service di Application memanggil class konkret dari Infrastructure → dependency rule terbalik.
- ❌ Folder dibagi per *jenis file* (`controllers/`, `models/`, `services/`) → struktur teknis, bukan arsitektur. Untuk proyek besar, lebih baik dibagi per *fitur* atau per *layer*.

**Yang benar:** Domain hanya berisi class murni (POJO / POCO / dataclass). Akses database lewat *interface* Repository yang implementasinya ada di Infrastructure.
</div>

### Contoh Domain Modeling / Domain Modeling Examples

Berikut contoh lengkap setiap building block Domain:

#### Entity

Entity punya identitas unik. Dua entity berbeda meskipun atributnya sama, jika ID-nya beda.

```csharp
// ✅ Entity dengan identitas kuat dan factory method
public class Order : Entity<OrderId>
{
    public CustomerId CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    public Money TotalAmount { get; private set; }
    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    // Konstruktor private — hanya bisa dibuat melalui factory method
    private Order(OrderId id, CustomerId customerId) : base(id)
    {
        CustomerId = customerId;
        Status = OrderStatus.Draft;
    }

    // Factory method memastikan invariant terpenuhi saat pembuatan
    public static Order Create(CustomerId customerId, IEnumerable<Product> products, Address shippingAddress)
    {
        if (!products.Any())
            throw new DomainException("Order harus memiliki minimal satu item.");

        var order = new Order(OrderId.NewId(), customerId);
        foreach (var product in products)
            order._items.Add(OrderItem.Create(product));

        order.TotalAmount = order.CalculateTotal();
        order.AddDomainEvent(new OrderCreatedEvent(order.Id, customerId));
        return order;
    }

    // Metode bisnis yang memproteksi invariant
    public void Confirm()
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Hanya order berstatus Draft yang dapat dikonfirmasi.");
        Status = OrderStatus.Confirmed;
        AddDomainEvent(new OrderConfirmedEvent(Id));
    }
}
```

#### Value Object

Value Object ditentukan oleh nilainya, bukan identitas. Selalu *immutable*.

```csharp
// ✅ Value Object — immutable, equality berdasarkan nilai
public sealed class Money : ValueObject
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        if (amount < 0) throw new DomainException("Jumlah uang tidak boleh negatif.");
        if (string.IsNullOrWhiteSpace(currency)) throw new DomainException("Kode mata uang wajib diisi.");
        Amount = amount;
        Currency = currency.ToUpperInvariant();
    }

    // Operasi bisnis menghasilkan Value Object baru (immutable)
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException($"Tidak dapat menjumlahkan {Currency} dengan {other.Currency}.");
        return new Money(Amount + other.Amount, Currency);
    }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Amount;
        yield return Currency;
    }
}
```

#### Aggregate Root

Aggregate Root = satu-satunya pintu untuk memodifikasi state aggregate. Objek di luar aggregate hanya boleh referensi Aggregate Root.

```csharp
// ✅ Order sebagai Aggregate Root — OrderItem hanya bisa dimanipulasi melalui Order
public class Order : AggregateRoot<OrderId>
{
    private readonly List<OrderItem> _items = new();
    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();

    public void AddItem(ProductId productId, string productName, Money price, int quantity)
    {
        if (Status == OrderStatus.Completed)
            throw new DomainException("Tidak dapat menambah item ke order yang sudah selesai.");

        var existingItem = _items.FirstOrDefault(i => i.ProductId == productId);
        if (existingItem != null)
            existingItem.IncreaseQuantity(quantity);
        else
            _items.Add(OrderItem.Create(productId, productName, price, quantity));

        RecalculateTotal();
    }
}

// ❌ SALAH — mengakses OrderItem langsung dari luar aggregate
var orderItem = orderItemRepository.GetById(itemId);
orderItem.ChangeQuantity(5);

// ✅ BENAR — semua perubahan melalui Aggregate Root
var order = orderRepository.GetById(orderId);
order.AddItem(productId, productName, price, quantity);
orderRepository.Save(order);
```

#### Domain Event

Domain Event = fakta yang sudah terjadi di domain, past tense, immutable.

```csharp
// ✅ Domain Event — merepresentasikan fakta yang telah terjadi
public sealed class OrderCreatedEvent : DomainEvent
{
    public OrderId OrderId { get; }
    public CustomerId CustomerId { get; }
    public Money TotalAmount { get; }
    public DateTime OccurredAt { get; }

    public OrderCreatedEvent(OrderId orderId, CustomerId customerId, Money totalAmount)
    {
        OrderId = orderId;
        CustomerId = customerId;
        TotalAmount = totalAmount;
        OccurredAt = DateTime.UtcNow;
    }
}

// Event di-raise di dalam Aggregate Root, di-dispatch oleh Application layer
public class CreateOrderCommandHandler
{
    public async Task HandleAsync(CreateOrderCommand command)
    {
        var order = Order.Create(command.CustomerId, products);
        await _orderRepository.AddAsync(order);
        await _unitOfWork.CommitAsync();

        // Dispatch semua domain events yang terakumulasi
        foreach (var domainEvent in order.DomainEvents)
            await _eventDispatcher.DispatchAsync(domainEvent);
    }
}
```

#### Domain Service

Domain Service = logika bisnis yang melibatkan beberapa domain objects dan tidak pas di satu Entity.

```csharp
// ✅ Domain Service — menghitung harga akhir dengan diskon + voucher + loyalty
public class PricingDomainService
{
    public Money CalculateFinalPrice(
        IEnumerable<OrderItem> items, Voucher? voucher, LoyaltyAccount loyaltyAccount)
    {
        var subtotal = items.Aggregate(
            new Money(0, "IDR"),
            (total, item) => total.Add(item.SubTotal));

        var afterVoucher = voucher != null ? voucher.Apply(subtotal) : subtotal;
        var loyaltyDiscount = loyaltyAccount.CalculateDiscount(afterVoucher);
        return afterVoucher.Subtract(loyaltyDiscount);
    }
}
```

#### Repository Interface & Implementation

Interface di Domain, implementasi di Infrastructure. Repository bekerja dengan Aggregate Root.

```csharp
// ✅ Interface di Domain layer
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(OrderId id, CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
    Task UpdateAsync(Order order, CancellationToken ct = default);
}

// ✅ Implementasi di Infrastructure layer
public class OrderRepository : IOrderRepository
{
    private readonly OrderDbContext _dbContext;
    public OrderRepository(OrderDbContext dbContext) => _dbContext = dbContext;

    public async Task<Order?> GetByIdAsync(OrderId id, CancellationToken ct = default)
        => await _dbContext.Orders.Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id, ct);

    public async Task AddAsync(Order order, CancellationToken ct = default)
        => await _dbContext.Orders.AddAsync(order, ct);
}

// Composition Root menghubungkan keduanya
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
```

### Domain Layer Tanpa Framework Dependency

Domain layer hanya menggunakan *pure language constructs* — class, interface, enum, exception bawaan bahasa.

```csharp
// ❌ SALAH — Domain Entity bergantung pada framework
using System.ComponentModel.DataAnnotations;  // EF Core
using Newtonsoft.Json;                         // JSON library

public class Product
{
    [Key]                    // ❌ EF Core attribute
    public int Id { get; set; }
    [Required]               // ❌ Data Annotations dari ASP.NET
    public string Name { get; set; }
    [JsonIgnore]             // ❌ Newtonsoft.Json attribute
    public decimal CostPrice { get; set; }
}
```

```csharp
// ✅ BENAR — Domain Entity murni tanpa framework dependency
public class Product : Entity<ProductId>
{
    public string Name { get; private set; }
    public Money Price { get; private set; }

    public void UpdatePrice(Money newPrice)
    {
        if (newPrice.Amount <= 0)
            throw new DomainException("Harga harus lebih dari nol.");
        Price = newPrice;
    }
}
```

| Jika Domain bergantung pada framework | Dampaknya |
|---------------------------------------|-----------|
| ASP.NET attributes (`[Required]`) | Domain terikat pada cara serialisasi HTTP |
| EF Core attributes (`[Column]`, `[Table]`) | Domain terikat pada cara data disimpan |
| Logging framework (`ILogger<T>`) | Domain terikat pada cara logging dilakukan |
| DI container abstractions | Domain tidak bisa diuji tanpa container |

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Batas antar layer ketat: Presentation → Application → Domain → Infrastructure. Inner layer tidak boleh tahu outer layer.

<span class="badge badge-mandatory">Mandatory</span> Domain harus bebas dari dependency framework — hanya konstruk bahasa pemrograman murni.

<span class="badge badge-mandatory">Mandatory</span> Domain dimodelkan dengan: Entity, Value Object, Aggregate Root, Domain Event, Repository (interface), Domain Service.

<span class="badge badge-mandatory">Mandatory</span> Komunikasi antar layer melalui *interface*, bukan class konkret.

<span class="badge badge-mandatory">Mandatory</span> Logika domain TIDAK bocor ke Application atau Presentation.

<span class="badge badge-mandatory">Mandatory</span> Aturan layer divalidasi dengan architecture unit test (ArchUnit, NetArchTest, atau setara).

<span class="badge badge-mandatory">Mandatory</span> Struktur folder mencerminkan layer arsitektur, bukan jenis file.

<span class="badge badge-mandatory">Mandatory</span> Kode shared antar service diletakkan di `/src/shared/` — tidak ada duplikasi cross-cutting concern.

### Contoh Struktur / Example

| Folder | Deskripsi |
|---|---|
| `src/` | Root kode aplikasi |
| `├── shared/contracts/` | Interface / DTO yang dipakai bersama antar service |
| `├── shared/common/` | Utilitas umum (logger wrapper, helper) |
| `└── service-name/` | Folder per service / bounded context |
| `    ├── src/presentation/` | REST / gRPC controller |
| `    ├── src/application/` | Use case, orchestrator |
| `    ├── src/domain/` | Entity, Value Object, Repository interface |
| `    ├── src/infrastructure/` | DB, HTTP client, queue implementation |
| `    ├── tests/unit/` | Test domain & application (no I/O) |
| `    ├── tests/integration/` | Test dengan DB / queue nyata |
| `    └── tests/contract/` | Consumer-driven contract test |

---

## Architecture Decision Record (ADR)

### Apa ini? / What is this?

**ID:** ADR adalah catatan singkat tentang keputusan arsitektur yang pernah diambil tim — apa keputusannya, mengapa, dan apa konsekuensinya. Mirip seperti "catatan rapat" tapi khusus untuk pilihan teknis penting.

**EN:** An ADR is a short note about an architectural decision the team has made — what was decided, why, and what the consequences are. Like meeting minutes, but specific to important technical choices.

### Mengapa penting? / Why it matters?

- **6 bulan kemudian, tidak ada yang ingat kenapa kita pilih RabbitMQ daripada Kafka.** Tanpa ADR, perdebatan terulang dan keputusan bisa dibalik tanpa alasan.
- **Onboarding lebih cepat** — anggota baru bisa baca ADR untuk memahami konteks historis.
- **Mencegah "argumen yang sama berulang kali"** — keputusan yang sudah dibuat punya catatan yang bisa dirujuk.

### Use Case

> **Skenario:** Setahun lalu tim memutuskan pakai *Outbox Pattern* untuk publikasi event. Hari ini ada developer baru yang melihat tabel `outbox` dan berpikir "ini overhead tidak perlu" dan ingin menghapusnya.
>
> Dengan ADR `0007-use-outbox-pattern.md`, dia bisa baca: "Outbox dipakai karena pernah terjadi event hilang saat broker down. Konsekuensi: write & publish jadi atomic. Alternatif yang ditolak: publish langsung (alasan: tidak transaksional)."
>
> Tanpa ADR: ia menghapus outbox, masalah lama muncul lagi, butuh post-mortem mahal.
{ .usecase }

### Istilah & Konsep / Glossary

- **Status ADR:**
    - *Proposed* — masih diusulkan, belum disetujui.
    - *Accepted* — sudah disepakati & diterapkan.
    - *Deprecated* — tidak relevan lagi tapi belum diganti.
    - *Superseded* — diganti dengan ADR lain yang lebih baru (mis. ADR 0007 superseded by 0015).
- **Context** — situasi/masalah yang melatari keputusan.
- **Consequences** — efek setelah keputusan diambil — yang baik dan yang buruk.

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Setiap keputusan arsitektur signifikan dicatat sebagai ADR.

<span class="badge badge-mandatory">Mandatory</span> Format ADR: **Title, Status, Context, Decision, Consequences**.

<span class="badge badge-mandatory">Mandatory</span> ADR disimpan di `/docs/adr/` dengan penomoran urut (`0001-...`, `0002-...`).

<span class="badge badge-mandatory">Mandatory</span> ADR yang sudah *Accepted* bersifat immutable — jika berubah, buat ADR baru yang meng-*supersede*, jangan edit yang lama.

<span class="badge badge-mandatory">Mandatory</span> ADR ditulis saat keputusan dibuat, BUKAN setelah keputusan berlaku berbulan-bulan.

### Contoh Struktur / Example

```markdown
# 0007. Use Outbox Pattern for Event Publishing

## Status
Accepted — 2025-03-14

## Context
Kami pernah mengalami event hilang saat message broker (RabbitMQ) sempat down 30 detik
saat order service melakukan commit transaksi. Hasilnya: order tersimpan di DB tapi
event `OrderPlaced` tidak terpublikasi → downstream service (inventory) tidak ter-update.

## Decision
Semua domain event ditulis ke tabel `outbox` dalam transaksi yang sama dengan
operasi bisnisnya. Background relay membaca tabel ini dan mempublikasikan ke broker.

## Consequences
+ Event tidak akan hilang meski broker down (akan di-retry).
+ Publish jadi *eventually consistent* (delay milidetik–detik).
- Tambahan kompleksitas: butuh relay process & monitoring tabel outbox.
- Tabel outbox tumbuh — perlu housekeeping.
```

---

## Ringkasan / Summary

Di bab ini kita belajar:

1. **Struktur folder dasar** memberi kerangka konsisten untuk seluruh proyek.
2. **Clean Architecture** memisahkan kode jadi layer dengan aturan ketat agar Domain bersih dari teknis.
3. **ADR** mencatat *mengapa* keputusan dibuat agar pengetahuan tidak hilang.

> **Selanjutnya:** [2. Application Patterns](02-application-patterns.md) — pola-pola umum untuk membangun aplikasi modern: microservice, async, caching, dan lainnya.
