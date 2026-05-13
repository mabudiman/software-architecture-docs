# 3. Security
### Keamanan / Security

Keamanan bukan fitur yang ditambahkan di akhir — ia diintegrasikan dari awal. Bab ini menjelaskan tiga pilar keamanan aplikasi: **siapa yang boleh** (autentikasi & otorisasi), **bagaimana melindungi rahasia** (secret management & enkripsi), dan **bagaimana menulis kode yang aman** (secure coding).

Security isn't a feature bolted on at the end — it's built in from day one. This chapter covers three pillars of application security: **who is allowed** (auth), **how to protect secrets** (secret management & encryption), and **how to write secure code** (secure coding).

---

## Authentication & Authorization

### Apa ini? / What is this?

**ID:**

- **Authentication (AuthN)** — proses memastikan identitas: "Siapa Anda?" Biasanya via username/password, biometrik, OAuth, dll.
- **Authorization (AuthZ)** — proses memutuskan izin: "Apa yang boleh Anda lakukan?" Setelah identitas diketahui, sistem cek hak akses.

Analogi: di gedung kantor, kartu pegawai = autentikasi (membuktikan Anda karyawan). Tapi kartu Anda mungkin hanya bisa buka lantai 3, bukan lantai 10 (otorisasi).

**EN:**

- **Authentication (AuthN)** — confirming identity: "Who are you?" Via username/password, biometrics, OAuth, etc.
- **Authorization (AuthZ)** — deciding permission: "What can you do?" After identity is known, check access rights.

### Mengapa penting? / Why it matters?

- Tanpa autentikasi: siapa pun bisa pura-pura jadi siapa pun → bencana privasi & finansial.
- Tanpa otorisasi yang ketat: user biasa bisa akses fitur admin → *privilege escalation*.
- Password yang disimpan plain text = 1 leak DB cukup untuk hancurkan semua akun user.

### Use Case

> **Skenario:** API e-commerce. User A login → dapat token. Token ini perlu:
>
> 1. Membuktikan A adalah A (autentikasi) di setiap request.
> 2. Memastikan A hanya bisa lihat *order*-nya sendiri, bukan order user B (otorisasi).
> 3. Mencegah brute force password A (account lockout).
>
> Tanpa lockout: penyerang mencoba 10 juta password/detik. Dengan lockout setelah 5 percobaan gagal: brute force tidak praktis.
{ .usecase }

### Istilah & Konsep / Glossary

- **RBAC (Role-Based Access Control)** — akses ditentukan oleh *role* user (Admin, Manager, User). Mudah dikelola, cocok untuk kebanyakan kasus.
- **ABAC (Attribute-Based Access Control)** — akses ditentukan oleh *atribut* (mis. departemen, lokasi, jam akses). Lebih fleksibel tapi kompleks.
- **Access Token** — token jangka pendek (mis. 15 menit) yang dibawa client untuk membuktikan identitas. Biasanya JWT.
- **Refresh Token** — token jangka panjang untuk meminta access token baru tanpa minta user login ulang. Disimpan lebih aman.
- **JWT (JSON Web Token)** — format token yang berisi claim (mis. user ID, role) dan tanda tangan kriptografis.
- **MFA (Multi-Factor Authentication)** — autentikasi dengan ≥2 faktor berbeda: sesuatu yang Anda *tahu* (password), *punya* (HP/OTP), *miliki* (sidik jari).
- **bcrypt / argon2 / scrypt** — algoritma hashing password yang *lambat secara desain* — agar brute force mahal.
- **Cost Factor** — parameter di bcrypt/argon2 yang menentukan seberapa lambat hashing. Naikkan seiring CPU semakin cepat.
- **Account Lockout** — sementara mengunci akun setelah N percobaan login gagal.
- **API Gateway** — pintu masuk yang biasanya menangani autentikasi awal. Tapi **service tetap harus cek otorisasi sendiri** — gateway bisa dilewati.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Simpan password plain text di DB.
- ❌ Hash password dengan MD5 / SHA-1 / SHA-256 → terlalu cepat, mudah di-brute force dengan GPU.
- ❌ Access token jangka panjang (mis. 30 hari) → kalau bocor, penyerang punya akses lama.
- ❌ Hanya cek otorisasi di Gateway → request langsung ke service bypass cek.
- ❌ Cek otorisasi di Presentation tapi tidak di Application/Domain → endpoint baru mudah lupa cek.
</div>

### Aturan / Rules

#### Mandatory

<span class="badge badge-mandatory">Mandatory</span> Terapkan RBAC atau ABAC secara konsisten di semua service.

<span class="badge badge-mandatory">Mandatory</span> Cek otorisasi WAJIB di level service — jangan andalkan API Gateway saja.

<span class="badge badge-mandatory">Mandatory</span> Access token jangka pendek (maksimal 15 menit). Pakai refresh token untuk session berkelanjutan.

<span class="badge badge-mandatory">Mandatory</span> Account lockout setelah beberapa kali percobaan login gagal.

<span class="badge badge-mandatory">Mandatory</span> JANGAN simpan password plain text — pakai bcrypt, argon2, atau scrypt dengan cost factor yang sesuai.

#### Optional — Multi-factor Authentication

<span class="badge badge-optional">Optional</span> MFA wajib untuk semua akun privileged (admin, operator infra).

---

## Security: Secret Management, Encryption, Testing & Compliance

### Apa ini? / What is this?

**ID:** Kategori payung untuk hal-hal yang melindungi aplikasi dari pencurian data & serangan:

- **Secret Management** — menyimpan rahasia (password DB, API key) dengan aman.
- **Encryption** — mengenkripsi data baik saat transit (di jaringan) maupun saat rest (di disk).
- **Security Testing (SAST/DAST)** — memindai kode & aplikasi yang berjalan untuk celah keamanan.
- **Compliance** — kepatuhan terhadap regulasi (GDPR, PCI-DSS, OJK).

**EN:** Umbrella category for things that protect the app from data theft & attacks: secret storage, encryption, security testing, and compliance with regulations.

### Mengapa penting? / Why it matters?

- Hardcoded API key di Git history → bisa di-scan publik dalam hitungan menit.
- Database yang dicuri tapi terenkripsi at-rest = data masih aman.
- Tanpa SAST/DAST, kerentanan baru tidak terdeteksi sampai *pen-test* tahunan (terlalu lambat).
- Pelanggaran GDPR = denda hingga 4% pendapatan global.

### Use Case

> **Skenario:** Developer tanpa sengaja commit `db_password=secret123` ke repo. 5 menit kemudian, bot scanner GitHub menemukan key tersebut, dan akun DB Anda di-brute force.
>
> Dengan *secret scanner* otomatis di CI: commit ditolak sebelum sampai ke remote. Secret manager (Vault, AWS Secrets Manager) menyediakan secret saat runtime → tidak ada di kode.
{ .usecase }

### Istilah & Konsep / Glossary

- **Secret Manager** — sistem khusus untuk simpan secret (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault). Akses dengan IAM, audit log, rotasi otomatis.
- **TLS (Transport Layer Security)** — protokol enkripsi untuk komunikasi (HTTPS = HTTP + TLS).
- **mTLS (mutual TLS)** — TLS dua arah: client dan server saling membuktikan identitas dengan sertifikat.
- **AES-256** — algoritma enkripsi simetris standar industri. Cepat dan kuat.
- **KMS (Key Management Service)** — service untuk mengelola kunci enkripsi (AWS KMS, GCP KMS).
- **Key Rotation** — mengganti kunci enkripsi secara berkala — kalau bocor, dampaknya terbatas.
- **SAST (Static Application Security Testing)** — analisis kode source tanpa menjalankannya. Cari pola berbahaya (SQL injection, hardcoded secret).
- **DAST (Dynamic Application Security Testing)** — menguji aplikasi yang *running*: kirim payload jahat, lihat reaksi.
- **OWASP Dependency-Check / npm audit** — tool yang membandingkan dependency proyek dengan database CVE (kerentanan diketahui).
- **Penetration Testing (Pen-Test)** — simulasi serangan oleh tim independen.
- **GDPR / PCI-DSS / OJK** — regulasi: GDPR (privasi data UE), PCI-DSS (kartu kredit), OJK (jasa keuangan Indonesia).
- **PII (Personally Identifiable Information)** — data yang bisa mengidentifikasi orang (nama, alamat, no KTP, no HP).
- **Data Residency** — aturan tentang di negara mana data boleh disimpan.
- **Zero-trust** — paradigma keamanan: "jangan percaya siapa pun secara default" — semua komunikasi termasuk internal harus terotentikasi & terenkripsi.

### Aturan / Rules

#### Mandatory — Secret Management

<span class="badge badge-mandatory">Mandatory</span> Semua secret disimpan di secret manager khusus (Vault, AWS Secrets Manager, Azure Key Vault).

<span class="badge badge-mandatory">Mandatory</span> Secret dirotasi sesuai jadwal & segera saat dicurigai bocor.

<span class="badge badge-mandatory">Mandatory</span> Akses secret mengikuti *least-privilege* — tiap service hanya akses secret-nya sendiri.

<span class="badge badge-mandatory">Mandatory</span> Secret scanner otomatis berjalan di setiap commit & PR (truffleHog, git-secrets).

#### Optional — Encryption

<span class="badge badge-optional">Optional</span> Semua data in-transit dienkripsi TLS 1.2 minimum. TLS 1.3 preferred.

<span class="badge badge-optional">Optional</span> Data sensitif at-rest dienkripsi AES-256 atau setara.

<span class="badge badge-optional">Optional</span> Kunci enkripsi dikelola via KMS — JANGAN hardcoded atau disimpan di config aplikasi.

<span class="badge badge-optional">Optional</span> Rotasi kunci dilakukan sesuai jadwal & saat dicurigai bocor.

<span class="badge badge-optional">Optional</span> JANGAN implementasi kriptografi sendiri — pakai library teraudit.

<span class="badge badge-optional">Optional</span> Komunikasi internal service-to-service pakai mTLS di lingkungan zero-trust.

#### Optional — Security Test (SAST / DAST)

<span class="badge badge-optional">Optional</span> SAST jalan di setiap PR sebagai bagian CI.

<span class="badge badge-optional">Optional</span> DAST jalan di environment staging pada setiap release pipeline.

<span class="badge badge-optional">Optional</span> Semua finding critical & high WAJIB diselesaikan sebelum deploy ke production.

<span class="badge badge-optional">Optional</span> Dependency vulnerability scan jalan di setiap build (OWASP Dependency-Check, `npm audit`).

<span class="badge badge-optional">Optional</span> Penetration test minimal sekali per tahun oleh pihak independen.

<span class="badge badge-optional">Optional</span> Hasil security test dilacak & di-review tiap release cycle.

#### Optional — Compliance

<span class="badge badge-optional">Optional</span> Identifikasi regulasi yang berlaku di awal proyek (GDPR, PCI-DSS, ISO 27001, OJK).

<span class="badge badge-optional">Optional</span> Persyaratan data residency didokumentasikan & ditegakkan di level infra.

<span class="badge badge-optional">Optional</span> Data PII diidentifikasi, diklasifikasi, dan ditangani sesuai regulasi.

<span class="badge badge-optional">Optional</span> Kebijakan retensi & penghapusan data dapat ditegakkan on-demand.

<span class="badge badge-optional">Optional</span> Bukti compliance (log, audit trail, hasil test) disimpan & dapat diakses untuk audit.

---

## Secure Coding

### Apa ini? / What is this?

**ID:** Praktik menulis kode yang minim celah keamanan dari awal. Banyak kerentanan terjadi karena kebiasaan menulis kode "yang penting jalan" tanpa memikirkan input jahat.

**EN:** The practice of writing code with minimal security holes from the start. Many vulnerabilities happen because of habits like "just make it work" without thinking about hostile input.

### Mengapa penting? / Why it matters?

- 90% celah keamanan web masuk dalam **OWASP Top 10** yang sama dari tahun ke tahun: SQL injection, XSS, CSRF, broken auth, dll.
- Mencegah jauh lebih murah daripada memperbaiki setelah breach (denda, kehilangan kepercayaan, biaya forensik).

### Use Case

> **Skenario SQL Injection klasik:**
>
> ```python
> # Salah ❌
> query = "SELECT * FROM users WHERE name='" + user_input + "'"
> db.execute(query)
> ```
>
> Jika `user_input = "' OR '1'='1"`, query jadi `SELECT * FROM users WHERE name='' OR '1'='1'` → semua user di-dump. Bencana.
>
> ```python
> # Benar ✅
> query = "SELECT * FROM users WHERE name = ?"
> db.execute(query, [user_input])  # parameterized
> ```
{ .usecase }

### Istilah & Konsep / Glossary

- **OWASP Top 10** — daftar 10 kerentanan keamanan web paling umum, di-update OWASP secara berkala. Wajib dipahami semua developer web.
- **SQL Injection** — penyerang menyisipkan SQL ke input → database dimanipulasi.
- **XSS (Cross-Site Scripting)** — penyerang menyisipkan JavaScript ke halaman → dijalankan di browser korban.
- **CSRF (Cross-Site Request Forgery)** — penyerang membuat browser korban mengirim request tidak sengaja ke situs lain (sambil korban masih login).
- **Parameterized Query** — query SQL dengan placeholder yang isinya dibungkus oleh driver DB → input tidak diinterpretasi sebagai SQL.
- **Output Encoding** — meng-*encode* data sebelum ditampilkan sesuai konteks (HTML, JSON, URL). Mencegah injection.
- **Least Privilege** — setiap akun/proses hanya punya hak minimum yang dibutuhkan untuk tugasnya.
- **Input Validation** — memeriksa & menolak input yang tidak sesuai aturan (format, panjang, tipe) sebelum diproses.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ String concatenation untuk SQL: `"WHERE id=" + id` → SQL injection.
- ❌ `innerHTML = userInput` → XSS.
- ❌ Service account DB punya hak `DROP TABLE` padahal tidak butuh.
- ❌ Log seluruh request body → bisa berisi password, token, kartu kredit.
- ❌ API key di file `config.json` yang ter-commit.
</div>

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Validasi & sanitasi semua input di boundary — anggap semua input bermusuhan.

<span class="badge badge-mandatory">Mandatory</span> Pakai parameterized query / ORM untuk semua interaksi DB — JANGAN concat user input ke SQL.

<span class="badge badge-mandatory">Mandatory</span> Encode output sesuai konteks (HTML, JSON, URL) untuk mencegah injection.

<span class="badge badge-mandatory">Mandatory</span> Prinsip *least privilege* untuk service account, DB user, API credential.

<span class="badge badge-mandatory">Mandatory</span> JANGAN log data sensitif: PII, token, password, no kartu kredit.

<span class="badge badge-mandatory">Mandatory</span> Review OWASP Top 10 sebagai checklist minimum setiap rilis.

<span class="badge badge-mandatory">Mandatory</span> JANGAN ada secret, credential, atau API key di source code, config file, atau Docker image — pakai secret manager.

---

## Ringkasan / Summary

- **AuthN** = identitas, **AuthZ** = izin. Keduanya wajib & dilakukan di setiap service.
- **Secret manager** menjauhkan rahasia dari kode. Token pendek + refresh token mengurangi blast radius.
- **Enkripsi** in-transit (TLS) dan at-rest (AES) melindungi data dari pencurian.
- **SAST/DAST + dependency scan** menangkap kerentanan otomatis sebelum sampai produksi.
- **OWASP Top 10** = peta jalan minimum untuk setiap web developer.

> **Selanjutnya:** [4. Quality & Reliability](04-quality-reliability.md) — bagaimana memastikan kode yang sudah aman juga reliable & berkualitas.
