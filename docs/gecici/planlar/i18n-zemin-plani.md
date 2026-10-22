---
tags: [plan, teknik/i18n]
date: 2026-10-23
status: taslak — Faruk onayı bekliyor
sahibi: oksis-tech-lead
---

# Çok dil (i18n) zemin aşaması — dilim planı

> **Bağlayıcı karar:** [[0029-cok-dil-altyapisi-ingilizce]] (accepted, 2026-10-23). Bu plan o kararı yeniden tartışmaz; yalnız ADR "Göç planı" **madde 1 (Zemin)** işini dilimlere böler.
> **Kapsam:** altyapı kurulur, **Türkçe görünüm değişmez**. Modül metinlerinin anahtara taşınması (madde 2) kapsam dışıdır; yalnız sıra önerisi §8'de.
> **İncelenen kod:** `oksis-api` origin/dev `40ad3bef`, `oksis-ui` origin/dev `b1c7bc76` (2026-10-23). Kod değiştirilmedi.
> **Onay gelmeden hiçbir masaya uygulama başlatılmaz.**

---

## 1. Koddan çıkan bulgular (planı şekillendiren)

### 1.1 ADR ile kod arasındaki ad farkları (kod kazanır)

| ADR'de | Kodda | Not |
|---|---|---|
| `UserAccount.PreferredLanguage` | `Account` (`Oksis.Domain/Modules/Identity/Entities/Account.cs`) — `TenantEntity`, okul başına hesap | Alan `Account.PreferredLanguage` olur. Hesap okula bağlı olduğundan tercih de okul başınadır. |
| `SubjectBranch` (branş adları çevrilmez) | `SubjectBranch` ders↔branş **bağlantı** tablosudur, ad taşımaz. Branş adları `MasterBranch` (`master.branches`) ve okulun `Branch` kaydındadır (`BranchCatalogImporter` ile kopyalanır) | Kararın niyeti `MasterBranch` + okul `Branch` için geçerli. ADR'de ad düzeltmesi önerilir. |
| Platform yüzeyi | Ayrı `PlatformAccount` (`Oksis.Domain/Modules/Platform/Entities/PlatformAccount.cs`) | ADR merkez yüzeyini anmıyor → §9 soru 5. |

### 1.2 Zemin için hazır olan dikiş yerleri

- **API hata sözlüğü zaten var:** `Oksis.Api/Errors/ErrorMessageCatalog.cs` (~290 anahtar→Türkçe cümle) `ResultExtensions.ToApiError` içinde `Humanize` ile API sınırında çalışıyor; `*Errors.cs` dosyalarının bir kısmı (`UserErrors`, `AccountErrors`) zaten `Error(Code, MessageKey)` kalıbında. Zemin bu sözlüğü `IStringLocalizer` arkasına taşır: `tr` içerik birebir aynı kalır, `en` boşsa `tr`'ye düşer. `ErrorMessageCatalogTests` (`tests/Oksis.Api.UnitTests/Errors/`) kapsam bekçisi olarak yeni kaynağa yönlendirilir.
- **Bildirim sözleşmesi tek noktadan geçiyor:** `NotificationRequest(EventId, SchoolId, RecipientAccountIds, NotificationMessage)` ve `NotificationMessage(Kind, Title, Body, DeepLink, Source)` (`Oksis.Application/Modules/Notifications/Abstractions/`). Metin bugün üretici tarafından hazır Türkçe dize olarak veriliyor; alıcı dili için kanca `NotificationDispatcher` + kanallarda (`InAppNotificationChannel`, `PushNotificationChannel`, `EmailNotificationChannel`) kurulur.
- **Alıcı çözümleme hesap kimliğiyle:** `NotificationRecipientResolver` kişi → `Person.LinkedAccountId` üzerinden `Account.Id` döndürüyor; alıcı dili `Account.PreferredLanguage ?? SchoolSettings.DefaultLanguage` olarak toplu sorgulanabilir.
- **ESLint için yeni paket gerekmez:** `packages/eslint-config` zaten `eslint-plugin-react` taşıyor (`react/jsx-no-literals` kuralı mevcut).

### 1.3 Zemini riskli yapan bulgular

1. **Süreç kültürü bilinçli olarak `tr-TR` (TB-271, `Program.cs` başı).** Sunucu araması `ToLower()` + LIKE ile Türkçe küçültmeye yaslanıyor (Application'da 39 `ToLower()`), sıralamalar `StringComparer.CurrentCulture` kullanıyor (22 yer). İstekte `CurrentCulture = en` yapılırsa "VELİ" araması ve Türkçe sıralama İngilizce kullanıcı için bozulur. **Öneri:** `RequestLocalization` yalnız **`CurrentUICulture`**'ı (metin dili) değiştirir; `CurrentCulture` (karşılaştırma/biçim) `tr-TR` kalır. Sunucuda tarih/sayı biçimleri modül göçünde açık kültürle ele alınır. (§9 soru 6.)
2. **Kültür dış ara katmana geri akmaz.** `ExceptionHandlingMiddleware` hattın başında; içeride atanan kültür `await` dönüşünde onda görünmez. Yakalanan istisnaların mesajı ayrıca çözülmeli (istek kültürü özelliğinden okunarak). Kimlik doğrulama/hız sınırı (401/403/429) kullanıcı çözülmeden üretildiği için yalnız `Accept-Language` → `tr` sırasını görür.
3. **Hangfire varsayılan olarak iş kuyruğa atan isteğin kültürünü yakalar** (`CaptureCulture` küresel süzgeci, Hangfire 1.8.23). İngilizce bir istekten kuyruğa giren iş İngilizce kültürle koşar — ADR "arka plan işi istek kültürü kullanmaz" diyor. `Oksis.Api/Extensions/HangfireSetup.cs`'de varsayılan kültür `tr-TR`'ye sabitlenmeli; Hangfire kapalıyken çalışan süreç içi yedek yol (`HangfireNotificationEnqueuer` fallback) da aynı kurala bağlanmalı.
4. **FluentValidation dili süreç genelinde `tr`'ye çivili** (`Oksis.Application/DependencyInjection.cs`, `EnsureProcessWideConfiguration`). ADR "kültüre bağlanır" diyor; bağlandığında İngilizce kullanıcı, göç bitene dek FluentValidation'ın kendi İngilizce varsayılan cümlelerini Türkçe alan adlarıyla karışık görür. Bu, dil seçicinin üretimde bayrak arkasında tutulması gerekçelerinden biridir (§9 soru 1).
5. **ADR "ortak tarih/sayı biçimleyicileri (`Intl`)" diyor; kod bilinçli olarak `Intl` kullanmıyor.** `packages/core/src/date/tr-date.ts` ve `format/tr-number.ts` başlıkları: Hermes derlemelerinde ICU verisi garanti değil, ay adları sessizce İngilizceye dönebiliyor. `i18next` çoğullama da `Intl.PluralRules` ister. Dilim 0'da cihazda ölçülmeden biçimleyici yolu seçilmez.
6. **Web, Cloudflare Workers Free sınırında.** `apps/web/middleware.ts` notu: Worker 3 MiB gzip sınırı, `proxy.ts` bu yüzden kullanılmıyor. `next-intl` + mesaj dosyaları boyutu ve kök layout'ta çerez okumanın tüm sayfaları dinamik yapması ölçülmeli.
7. **Bildirim metni kalıcı.** Uygulama içi bildirim satırı üretildiği anda metniyle yazılıyor; kullanıcı sonradan dil değiştirirse eski bildirimler eski dilde kalır. Kabul edilebilir, belgelenir.
8. **Sistem metni kullanıcı verisi olarak yazılan yerler var:** örn. `CreateSchoolCommandHandler` müdür profiline `"Okul Yönetimi"` / `"Müdür"` yazıyor. Zeminin işi değil; modül göçünde envantere alınır.
9. **Mobilde `expo-localization` yerel (native) modül:** yeni geliştirme/mağaza derlemesi ister, OTA ile gitmez.
10. **Eşzamanlı iş:** OYSS entegrasyonu 30 Ekim dev hedefinde; aynı API dosyalarına (`Program.cs`, `OksisDbContextModelSnapshot.cs`, OpenAPI şeması) dokunuyor. API dilimlerinin göç ve `Program.cs` adımları OYSS birleşmesinden sonra sıraya alınmalı (§9 soru 9).

---

## 2. Referans verisi: `NameKey` alacak tablolar (seed'den doğrulandı)

Kaynak: `Oksis.Infrastructure/Persistence/Seed/MasterData/*SeedData.cs` + `Configurations/**` içindeki `HasData` ve `ToMasterTable` çağrıları. Anahtar biçimi önerisi `reference.<tür>.<kalıcı kod>` (kod yoksa anlamsal ad); değer `tr` dosyasında bugünkü `Name` ile birebir aynıdır.

### 2.1 ADR listesi — doğrulama

| # | ADR'deki ad | Tablo / entity | Seed dosyası | Satır | Eklenecek kolon | Okula kopya? | Durum |
|---|---|---|---|---|---|---|---|
| 1 | Sınıf seviyesi | `master.grade_levels` / `GradeLevel` | `GradeLevelSeedData` | 14 | `NameKey` | Hayır (`SchoolGradeLevel` ad taşımaz) | Doğru |
| 2 | Sınav türü | `master.exam_types` / `MasterExamType` | `ExamTypeSeedData` | 7 | `NameKey`, `DescriptionKey` | **Evet** → `school.exam_types` / `ExamType` | Doğru; bkz. §2.3 |
| 3 | Resmî tatil | `master.official_holidays` / `OfficialHoliday` | `OfficialHolidaySeedData` | 7 sabit + 20 dinî (bayram + arife, 2026–2030); ~9 ayrı anahtar | `NameKey` | Hayır — okuma anında birleştirilir | Doğru; bkz. §2.3 |
| 4 | Nöbet yeri şablonu | `master.duty_location_templates` / `DutyLocationTemplate` | `DutyLocationTemplateSeedData` | 6 | `NameKey`, `DescriptionKey` | **Evet** → `DutyLocation.TemplateId` | Doğru; bkz. §2.3 |
| 5 | Yabancı dil | `master.foreign_languages` / `ForeignLanguage` | `ForeignLanguageSeedData` | 14 | `NameKey` | Hayır (`SchoolSettings` yalnız kimlik tutar) | Doğru — ama branş adıyla tutarsızlık notu §9 soru 4 |
| 6 | Rol | identity `SystemRole` | `SystemRoleSeedData` | 6 | `NameKey` (görünen alan `DisplayName`), `DescriptionKey` | Hayır | Doğru |
| 7 | İzin | identity `Permission` | `PermissionSeedData` | 138 | Yalnız `DescriptionKey` (ad kolonu yok; görünen metin `Description`) | Hayır | Doğru, kolon adı farklı |
| 8 | Bildirim türü | `master.notification_event_types` / `NotificationEventType` | `NotificationEventTypeSeedData` | 51 | `NameKey` | Hayır | Doğru |
| 9 | Sistem ayarı etiketi | `master.system_settings` / `SystemSetting` | `SystemSettingSeedData` | 12 | — | Hayır | **Fazla:** `Description` hiçbir uçta/ekranda gösterilmiyor (yalnız `Value` okunuyor: `StorageQuotaService`, `PersonUserCreationService`, `StudentEnrolledEventHandler`). Kolon eklenmez; bir ekran açılırsa o gün eklenir. |

### 2.2 ADR listesinde olmayan ama arayüzde görünen (eksik)

| Tablo / entity | Seed | Satır | Görüldüğü yer | Öneri |
|---|---|---|---|---|
| `master.academic_term_types` / `AcademicTermType` ("1. Dönem", "2. Dönem") | `AcademicTermTypeSeedData` | 2 | `AttendanceTermResolver`, `GetStudentSummaryQueryHandler` etiketleri, dönem seçici | `NameKey` ekle |
| `master.grade_scales` / `GradeScale` ("100'lük Sistem"…) | `GradeScaleSeedData` | 3 | `GET lookups/grade-scales`, akademik politika | `NameKey` + `DescriptionKey` ekle |

### 2.3 Karar gereken / çevrilmeyen

| Tablo | Durum |
|---|---|
| `master.consent_bundles` (KVKK aydınlatma metni başlığı) | Hukuki metin; çevirisi hukuk kararı → §9 soru 4 |
| `master.countries` (`Country.Name`: "Türkiye", "Almanya") | ADR il/ilçe/mahalleyi özel isim sayıyor, ülkeyi anmıyor → §9 soru 4 |
| `master.education_programs` ("Anadolu Lisesi"…) | MEB belgelerinden doğan katalog, ders kataloğuyla aynı nitelik → öneri: çevrilmez; §9 soru 4 |
| `master.subjects`, `subject_grade_levels`, `curriculum_*`, `meb_*` | ADR gereği çevrilmez (ders kataloğu) |
| `master.branches` (`MasterBranch`) + okul `Branch` | ADR gereği çevrilmez (branş adları) |
| `provinces`, `districts`, `neighborhoods` | ADR gereği çevrilmez |
| `master.plan_modules` | Ad kolonu yok |

### 2.4 Kopyalama ve boşaltma noktaları

**Sınav türü (`ExamType`, `school.exam_types`)**
- Taşıma: `ExamType.CreateFromMaster` → tek çağıran `Oksis.Application/Modules/Exams/Internal/ExamTypeCatalogImporter.ImportAsync`; onu çağıran iki yol `CreateSchoolCommandHandler` (okul açılışı) ve `ImportMasterExamTypesCommandHandler` ("çekirdekten getir"). `NameKey` ve `DescriptionKey` buradan kopyalanır.
- Boşaltma (ad): **gerekmez** — `ExamType.Update` çekirdekten gelen satırın kod/ad/dönemini değiştirmeyi `ExamType.MasterSourced.Immutable` ile reddediyor; ad hiç değişemediği için `NameKey` hiç boşalmaz. Bu kural değişirse boşaltma `ExamType.Update` içine yazılır.
- Boşaltma (açıklama): `ExamType.Update` açıklama değiştiğinde `DescriptionKey` boşaltılır (açıklama okulun kararı).
- `ExamType.Create` / `CreateCustom`: anahtar hiç yok.
- Geriye dönük doldurma: `master_exam_type_id` dolu mevcut okul satırlarına çekirdeğin anahtarı yazılır (ad donmuş olduğu için güvenli); açıklaması çekirdekten farklı satırlarda `DescriptionKey` boş bırakılır.

**Nöbet yeri (`DutyLocation`, `school.duty_locations`)**
- Taşıma: `CreateDutyLocationCommandHandler` → `DutyLocation.Create(..., templateId)`. Ad istemciden gelir (`request.Name`); `TemplateId` dolu **ve** ad şablonun `Name`'iyle (kırpılmış) aynıysa `NameKey` şablondan alınır, değilse boş. `DutyLocation`'da açıklama kolonu yok → `DescriptionKey` taşınmaz.
- Boşaltma: `DutyLocation.Update` ad değiştiğinde `NameKey` boşaltılır.
- Geriye dönük doldurma: `template_id` dolu ve adı şablon adına eşit satırlar.

**Resmî tatil** — okula satır yazılmıyor (`GetHolidaysQueryHandler` notu, `TB-177`): çekirdek satır okuma anında `OfficialHolidayResolver` ile birleştiriliyor. `NameKey`, `OfficialHolidayRow` ve çözümleyici çıktısına eklenmeli; tüketiciler: `GetHolidaysQueryHandler`, `GetSchoolHolidaysForSessionQueryHandler`, `GetOfficialHolidaysInRangeQueryHandler`, `Infrastructure/Calendars/HolidayCalendarReader`, `Infrastructure/Attendance/SchoolCalendarService`. Sezon sihirbazının (`OpenSeasonFromDraftCommandHandler`) yazdığı tatil satırları ve acil kapanış (`DeclareEmergencyClosureCommandHandler`) okul içeriğidir — anahtar almaz.

**Diğerleri** (sınıf seviyesi, yabancı dil, rol, izin, bildirim türü, dönem türü, not ölçeği) okula kopyalanmıyor; yalnız kimlikle referanslanıyor → boşaltma noktası yok.

---

## 3. Dilim haritası

Faruk'un kuralı gereği dilimler dikeydir: her dilim kendi API + web + mobil parçasını birlikte kapatır. Dilim içindeki alt işler (A1, A2…) masalara dağıtılır.

| Dilim | Ad | Masalar | Boy |
|---|---|---|---|
| 0 | Ölçüm ve doğrulama (spike) | mobil, web, devops | S |
| A | Dil tercihi uçtan uca | veri, backend, web, mobil | L |
| B | Sunucu metinleri yerelleştirilebilir | backend (+ web/mobil mock uyumu) | M |
| C | Referans verisi anahtarı | veri, backend (+ istemci çözümleyici gerekirse) | M |
| D | Çeviri yönetimi hattı (Tolgee + CI) | devops | M |
| E | Bekçiler, regresyon kanıtı ve kapanış | web, QA, teknik lider | S–M |
| S | Terim sözlüğü (kod dışı, paralel) | eğitim alan uzmanı, Faruk | S |

Dal adları (hepsi `dev`'den): `feature/i18n-zemin-<dilim>-<kısa>` (örn. `feature/i18n-zemin-a-dil-tercihi`). Masa worktree'leri `~/Repositories/oksis/` altında (`oksis-api-wt-backend`, `oksis-ui-wt-web`, `oksis-ui-wt-qa`; mobil/veri/devops için gerekirse aynı kökte açılır).

---

## 4. Dilimler

### Dilim 0 — Ölçüm ve doğrulama

**Amaç:** Geri dönüşü pahalı üç teknik belirsizliği kod yazılmadan kapatmak.

| Alt iş | Sorumlu | Ne ölçülür | Çıktı |
|---|---|---|---|
| 0.1 Hermes `Intl` | `oksis-mobile-developer` | iOS ve Android **cihaz/simülatör** derlemesinde (Expo web değil) `Intl.PluralRules`, `Intl.DateTimeFormat`, `Intl.NumberFormat` `tr` ve `en` için doğru mu; ay/gün adları, ondalık ayırıcı | Karar: `Intl` biçimleyici mi, saf TS tablo (`tr-date.ts` emsali + `en` tablosu) mı; `i18next` çoğullama için çoklu dolgu (polyfill) gerekir mi |
| 0.2 Worker boyutu | `oksis-web-developer` | Deneme dalında `next-intl` + iki dil mesaj dosyasıyla OpenNext Worker gzip boyutu (sınır 3 MiB); kök layout'ta çerez okumanın render kipine etkisi | Karar: sunucu istek yapılandırması mı, yalnız istemci sağlayıcısı mı |
| 0.3 Tolgee | `oksis-devops` | `tolgee/tolgee` imajı arm64'te (Oracle A1 test VM'i) çalışıyor mu; bellek tüketimi; ücretsiz sürümde **sözlük (glossary)**, ad alanı, CLI push/pull kullanılabilir mi (lisans sayfası bu özellikleri listelemiyor — kurulup denenmeli) | Barındırma kararı; sözlük Tolgee'de mi yoksa belge deposunda mı |

**Sonuçlar (2026-10-23):**

- **0.1 (iPhone, Static Hermes 250829098.0.14):** `DateTimeFormat`, `NumberFormat`, `Collator` `tr` ve `en` için doğru (22 Ekim 2026 Perşembe, 1.234.567,891, %26, ₺1.234,50; Türkçe sıralama ve `İ` doğru). `PluralRules`, `RelativeTimeFormat`, `ListFormat`, `DisplayNames`, `Segmenter` yok → `Intl.PluralRules` çoklu dolgusu (yalnız `tr` + `en` verisi) gerekli; göreli zaman ve liste mesaj anahtarıyla çözülür. Cihaz varsayılan locale'i `en-TR` çıktı — locale her çağrıda açık verilir. **Android henüz ölçülmedi**; `tr-date.ts`/`tr-number.ts` kaldırma kararı Android sonucuna bağlı.
- **0.2 (web, esbuild + gzip yaklaşık ölçüm, 9.000 anahtar):** bugün 2,49 MiB. Sunucu istek yapılandırması (`getRequestConfig`, kök layout'ta çerez) 3,18 MiB — sınırı aşıyor ve 83 rotanın tamamını dinamik yapıyor. Yalnız istemci sağlayıcısı (`tr` gömülü, `en` dinamik içe aktarım) 2,83 MiB, 64 statik / 19 dinamik korunuyor. Mesajları ad alanına bölmek Worker boyutunu küçültmüyor.
  - **Karar (Faruk):** istemci sağlayıcısı. Metin dil hazır olana dek iskelet arkasında kalır (Türkçe→İngilizce kayması görünmez), `<html lang>` açılışta küçük betikle çerezden ayarlanır. Kurulum, ileride web Oracle'da Node sunucusuna taşınırsa sunucu istek yapılandırmasına küçük değişiklikle geçebilecek biçimde yazılır; web'in nerede barındırılacağı üretim ortamı kararına bırakıldı. Kalan pay ~0,17 MiB; `en` büyüdükçe mesajların statik dosyaya taşınması gerekebilir.
- **0.3 (Tolgee, yerel amd64):** ücretsiz sürümde **sözlük yok** (`feature_not_enabled: GLOSSARY`); ad alanı çalışıyor ama projede `useNamespaces=true` açılmalı; CLI push/pull ve `JSON_ICU` içe/dışa aktarım çalışıyor; makine çevirisi kapalı. Bellek en az 1 GiB (768 MiB'da Metaspace OOM). Gömülü PostgreSQL 13 (EOL) — ayrı PG önerildi. İlk kurulum demo kullanıcıları açıyor, kapatılmalı. arm64 imajı var, Oracle'da çalıştırma doğrulanmadı.

- **Bağımlılık:** yok; hemen başlar.
- **Kabul ölçütleri:** üç soru için ölçülmüş yanıt; deneme dalları birleştirilmez, silinir.
- **Test kanıtı:** cihaz ekran görüntüleri (`tr`/`en` tarih, sayı, çoğul), Worker boyut çıktısı, Tolgee giriş ekranı + CLI push denemesi günlüğü. Bulgular SESSION-HANDOFF'a satır olarak yazılır.

---

### Dilim A — Dil tercihi uçtan uca

**Amaç:** Kullanıcı ve okul dil alanları, sunucuda istek kültürü, istemcilerde i18n kurulumu ve dil seçici. Bütün metinler `tr` kalır.

#### A1 — Veri modeli ve göç
- **Sorumlu:** `oksis-database-specialist` (göç), `oksis-backend-developer` (domain).
- **Dosyalar:** `Oksis.Domain/Modules/Identity/Entities/Account.cs` (`PreferredLanguage`, boş olabilir, `tr`|`en`), `Oksis.Domain/Modules/Schools/Entities/SchoolSettings.cs` (`DefaultLanguage`, zorunlu, varsayılan `tr`), ilgili `Configurations/Identity/*`, `Configurations/Schools/SchoolSettingsConfiguration.cs`, yeni göç + `OksisDbContextModelSnapshot.cs`.
- **Dikkat:** `SchoolSettings`'te zaten `EducationLanguage` (öğretim dili) var — ad karışıklığı olmasın; yeni alan arayüz dilidir. Dil değeri için ortak bir değer nesnesi/enum (`tr`, `en`) Domain'de tek yerde tanımlanır. `master.system_settings` içindeki `DEFAULT_LANGUAGE` bugün hiçbir yerde okunmuyor; dokunulmaz, son yedek sabit `tr`'dir.
- **Kabul:** göç ileri/geri temiz; mevcut okullarda `DefaultLanguage = tr`, mevcut hesaplarda `PreferredLanguage = null`.

#### A2 — İstek kültürü ve uçlar
- **Sorumlu:** `oksis-backend-developer`.
- **Dosyalar:** `Oksis.Api/Program.cs` (yerelleştirme kaydı ve hat sırası), yeni kültür sağlayıcı (`Oksis.Api/Localization/` altında), `Oksis.Api/Middleware/ExceptionHandlingMiddleware.cs`, `Oksis.Api/Extensions/HangfireSetup.cs`, `Oksis.Application/DependencyInjection.cs` (FluentValidation dil yöneticisi), `Identity/Queries/GetCurrentContext/ContextView.cs` + handler, `SelfController` (kendi dil tercihi), `SchoolSettingsController` (okul varsayılan dili; ayrı küçük uç önerilir, `basic-info` sözleşmesine dokunmadan), OpenAPI.
- **Kurallar:**
  - Sıra: kullanıcı tercihi → okul varsayılanı → `Accept-Language` → `tr`. Yalnız `tr` ve `en` kabul edilir; başka dil `tr`'ye düşer.
  - Yalnız `CurrentUICulture` istek kapsamında değişir; `DefaultThreadCurrentCulture` ve `CurrentCulture` `tr-TR` kalır (§1.3/1).
  - Kullanıcı/okul dili her istekte veritabanından okunmaz: hesap+okul anahtarlı önbellek; tercih ya da okul varsayılanı değişince geçersiz kılınır (okul değişikliği o okulun bütün girdilerini düşürür). Birden çok örnekte çalışmalı (Redis mevcut).
  - `me/context` yanıtına `preferredLanguage`, `schoolDefaultLanguage`, `effectiveLanguage` eklenir (ekleme; kırıcı değil).
  - Hangfire varsayılan kültürü `tr-TR`'ye sabitlenir; istek kültürü işe taşınmaz.
  - FluentValidation dil yöneticisi istek UI kültürüne bağlanır (`tr` için çıktı değişmez).
  - Okul varsayılanını değiştirme yetkisi okul ayarları yönetim izniyle aynı; kişinin tercihi yalnız kendisi tarafından.
- **Kabul:** `tr` kullanıcının bütün yanıtları birebir aynı; `Accept-Language: en` ile oturumsuz istek `en` kültürüne girer (metin yine `tr`, çünkü `en` kaynak yok → fallback); tercih değişince sonraki istek yeni kültürde; Hangfire işi `en` istekten kuyruğa girse bile `tr-TR` kültürüyle koşar.
- **Test kanıtı:** `./scripts/test-changed.sh` yeşil; yeni birim testleri (kültür sırası tablosu, geçersiz dil, önbellek geçersizleme); entegrasyon (`--integration`): `me/context` alanları, iki uç, yetki reddi (öğretmen okul dilini değiştiremez), tenant izolasyonu (A okulunun dili B'yi etkilemez). Güvenlik: yeni uçlar için `oksis-security-auditor` kısa denetimi.

#### A3 — `packages/i18n` ve istemci kurulumu
- **Sorumlu:** `oksis-web-developer` (paket + web), ardından `oksis-mobile-developer` (mobil). Kök `package-lock.json` ortak olduğundan **sıralı**.
- **Dosyalar:**
  - Yeni `packages/i18n`: mesaj dosyaları (§9 soru 3'e göre ya `tr.json`/`en.json` ya ad alanı başına dosya), tipli anahtar (TS modül genişletmesi; kod üretici aracı gerekmez), `tr` fallback, ortak biçimleyiciler (Dilim 0.1 kararına göre `Intl` ya da `packages/core` saf TS tabloları üzerine ince katman), referans adı çözümleyici (yalnız §9 soru 2 "istemci" çıkarsa). Başlangıçta yalnız iskelet ad alanları (`common`), içerik `tr`.
  - Web: `apps/web/package.json`, `apps/web/app/layout.tsx` (`<html lang>` etkin dilden), `apps/web/app/providers.tsx` (`next-intl` sağlayıcısı), `next-intl` istek yapılandırması (URL öneki yok; dil çerezden), `next.config.ts` eklentisi gerekirse. `apps/web/middleware.ts`'e dil yönlendirmesi **eklenmez** (önek yok).
  - Mobil: `apps/mobile/package.json`, `apps/mobile/src/app/_layout.tsx` (i18next başlatma), cihaz dili `expo-localization` ile, seçili dil kalıcı saklama (`apps/mobile/src/lib/` emsalleri: `remember-store.ts`).
  - `packages/api/src/client/config.ts` + `request.ts`: her isteğe etkin dili `Accept-Language` olarak ekleyen kanca (web ve mobil aynı).
- **Kabul:** web ve mobil açılışı ve bütün ekranlar görsel olarak değişmez; `typecheck`/`lint`/`test` yeşil; Worker boyutu sınır altında (Dilim 0.2 ölçüsüyle karşılaştırmalı).
- **Test kanıtı:** `packages/i18n` vitest (fallback, eksik anahtar, tip denetimi), web vitest; Playwright ile giriş + rol ana sayfaları önce/sonra ekran görüntüsü; mobil cihaz/simülatör ekran görüntüsü (Expo web yetmez — yerel modül).

#### A4 — Dil seçici ve istemci kültür akışı
- **Sorumlu:** `oksis-web-developer`, `oksis-mobile-developer` (paralel; ortak `packages/api` dosyalarında web önce).
- **Bağımlılık:** A2 sözleşmesi (OpenAPI) + A3.
- **Dosyalar:** `packages/api/src/generated/*` (codegen; yalnız bu alt iş üretir), `packages/api` self/school-settings uçları ve sorguları, `packages/api-mocks` `me/context` + iki yeni uç işleyicisi (web ve mobil mock kaydı), web: `apps/web/components/app-shell.tsx` kullanıcı menüsü (kişisel dil), `apps/web/features/settings/general-tab.tsx` (okul varsayılan dili), giriş ekranı (oturumsuz çerez seçimi); mobil: `apps/mobile/src/app/profile.tsx` ve giriş ekranı.
- **Kurallar:** Oturum açınca etkin dil `me/context.effectiveLanguage`'dan alınır, çerez/kalıcı depo güncellenir; dil değişince TanStack Query önbelleği geçersizlenir (sunucu tarafında çözülen adlar yenilensin). Seçici ve okul ayarı **özellik bayrağı** arkasında (`NEXT_PUBLIC_FEATURE_LANGUAGE_SWITCH`, `EXPO_PUBLIC_FEATURE_LANGUAGE_SWITCH`; emsal `NEXT_PUBLIC_FEATURE_CLUB_ADMINISTRATION_ADMISSION`), dev/test'te açık, üretimde kapalı — §9 soru 1. Seçenek etiketleri ("Türkçe", "English") kendi dilinde yazılır.
- **Kabul:** `en` seçen kullanıcıda sayfa yenilenince `<html lang="en">`, API istekleri `Accept-Language: en`; metinler `tr` fallback ile aynı görünür; okul varsayılanı `en` yapılınca tercihi boş kullanıcıda etkin dil `en`, açık tercihli kullanıcıda değişmez.
- **Test kanıtı:** api/api-mocks vitest (başlık ekleme, mock senaryoları), web Playwright akışı (seç → yenile → kalıcı), mobil cihazda aynı akış; bayrak kapalıyken seçicinin görünmediği ekran görüntüsü.

---

### Dilim B — Sunucu metinleri yerelleştirilebilir

**Amaç:** API'nin hata, doğrulama, bildirim, e-posta ve Excel metinleri için anahtar → kültür altyapısı. Hiçbir modülün metni taşınmaz; mevcut Türkçe çıktı birebir korunur.

- **Sorumlu:** `oksis-backend-developer`.
- **Bağımlılık:** A2 (`Program.cs` ve kültür sağlayıcı birleşmiş olmalı).
- **Dosyalar ve işler:**
  - JSON tabanlı `IStringLocalizer` fabrikası `Oksis.Infrastructure/Localization/` altında; kaynaklar gömülü JSON (`tr`, `en`; ad alanı başına), `en` eksikse `tr`, `tr` eksikse anahtar değil nötr cümle + uyarı günlüğü. Soyutlama Application'da `IStringLocalizer` (Microsoft.Extensions.Localization.Abstractions).
  - `Oksis.Api/Errors/ErrorMessageCatalog.cs` içeriği `errors` ad alanı `tr` dosyasına birebir taşınır; `ResultExtensions.ToApiError` ve `ExceptionHandlingMiddleware` yerelleştiriciyi kullanır; anahtar kalıbı (`TranslationKeyPattern`) ve "gerçek cümle geçer" davranışı korunur. `ErrorMessageCatalogTests` yeni kaynağın kapsam bekçisi olur.
  - Bildirim kancası: `NotificationMessage`'a anahtar + parametre ile kurulabilen biçim eklenir (mevcut hazır-metin biçimi aynen çalışır); alıcı dil çözümleyici (hesap kimlikleri → etkin dil, toplu sorgu, `Account.PreferredLanguage ?? SchoolSettings.DefaultLanguage`); `NotificationDispatcher` ve toplu kanallar (`IBatchNotificationChannel`, push) alıcıları dile göre gruplayıp her grupta metni o kültürde üretir. Hangfire yükü serileştirilebilir kalır (anahtar + parametre).
  - E-posta ve Excel için yalnız kültür kapsamı yardımcısı (açık kültürle üretim); `Infrastructure/Email/EmailTemplates.cs`, `Notifications/NotificationEmail.cs` ve ClosedXML çıktılarının metni bu dilimde **taşınmaz**.
- **Kabul:** `ErrorMessageCatalog`'daki her anahtar için eski `Humanize` çıktısı ile yeni yerelleştirici çıktısı `tr` kültüründe birebir eşit (altın test; eski sözlük ancak bu test yeşilken silinir); `en` kültüründe `tr`'ye düşer; anahtar+parametre biçimli test bildirimi iki dilli alıcı kümesinde her alıcıya kendi dilinde gider; hazır-metin biçimli mevcut bütün üreticiler değişmeden geçer.
- **Test kanıtı:** `test-changed.sh` + `--integration` (bildirim fan-out, push toplu kanal, Hangfire yolu ve süreç içi yedek yol); yeni bekçi: kodda kullanılan her `errors` anahtarının `tr` kaynakta karşılığı var.
- **Web/mobil payı:** istemci hata yüzeyi (`packages/api/src/client/mutation-error.ts`, mobil `change-password-form.tsx` anahtar sözlüğü) değişmez; yalnız `api-mocks`'taki hata yanıtları sözleşmeyle aynı kalır — doğrulama `oksis-qa-engineer`'de.

---

### Dilim C — Referans verisi anahtarı

**Amaç:** §2'deki tablolara anahtar kolonu, seed değerleri, kopyalama/boşaltma kuralları ve çözümleyici. Gösterim `tr`'de değişmez.

- **Sorumlu:** `oksis-database-specialist` (göç + geriye dönük doldurma), `oksis-backend-developer` (domain kuralları, çözümleyici).
- **Bağımlılık:** A1 göçü birleşmiş olmalı (model anlık görüntüsü ortak); çözümleyici B'nin yerelleştiricisine dayanır.
- **Dosyalar:**
  - Entity + yapılandırma: `GradeLevel`, `MasterExamType`, `ExamType`, `OfficialHoliday`, `DutyLocationTemplate`, `DutyLocation`, `ForeignLanguage`, `SystemRole`, `Permission`, `NotificationEventType`, `AcademicTermType`, `GradeScale` (+ §9 soru 4 sonucuna göre `ConsentBundle`, `Country`, `EducationProgram`).
  - Seed: ilgili `*SeedData.cs` `Row(...)` yardımcılarına anahtar alanı; `HasData` değiştiği için göç seed güncellemesi üretir.
  - Kopyalama/boşaltma: `ExamType` (`CreateFromMaster`, `Update`), `ExamTypeCatalogImporter`, `DutyLocation` (`Create`, `Update`), `CreateDutyLocationCommandHandler`, `OfficialHolidayResolver` + `OfficialHolidayRow` ve §2.4'teki beş tüketici.
  - Geriye dönük doldurma: göç içinde veri adımı (`school.exam_types`, `school.duty_locations`).
  - Çözümleyici: "anahtar doluysa etkin dilde çeviri, boşsa `Name`" kuralını uygulayan tek servis (yeri §9 soru 2'ye bağlı). `reference` ad alanı `tr` dosyası bugünkü adlarla doldurulur.
- **Kapsam sınırı:** DTO'ların tek tek çözümleyiciye bağlanması modül göçünün işidir; zemin yalnız kolonu, kuralı ve çözümleyiciyi kurar. İstisna: §9 soru 2 "sunucu" çıkarsa, yanıtlarda `name` alanı zaten etkin dilde döner ve sözleşme değişmez; "istemci" çıkarsa DTO'lara `nameKey` eklenmesi her modülün göçünde yapılır.
- **Kabul:** göç ileri/geri temiz; seed anahtarları her satırda dolu; okul açılışında içe aktarılan sınav türleri anahtar taşıyor; okulun eklediği tür/yer anahtarsız; nöbet yerinin adı değişince anahtar boş; mevcut okulların satırları doğru dolduruldu; `tr` kültüründe bütün ad çıktıları önceki ile aynı.
- **Test kanıtı:** domain birim testleri (kopyalama/boşaltma kuralları), göç entegrasyon testi (doldurma senaryoları: eşleşen/eşleşmeyen ad, özel tür), bekçi: her seed anahtarının `reference` `tr` dosyasında karşılığı var ve değeri `Name` ile aynı.

---

### Dilim D — Çeviri yönetimi hattı

**Amaç:** Tolgee'nin kurulumu ve iki depoda CI akışı. Tolgee çalışma anı bağımlılığı değildir.

- **Sorumlu:** `oksis-devops`.
- **Bağımlılık:** sunucu kurulumu Dilim 0.3'ten hemen sonra başlar (paralel); CI bağlantısı A3 (UI mesaj dosyaları) ve B (API kaynak dosyaları) yerleşince.
- **İşler:**
  - Sunucu: test VM'inde (veya §9 soru 7 kararına göre başka yerde) Docker + PostgreSQL, `cloudflared` ile alan adı, `TOLGEE_AUTHENTICATION_ENABLED` açık, yönetici hesabı ve API anahtarları gizli değişkenlerde (değer belgeye yazılmaz), PostgreSQL yedeği. Ücretsiz sürüm: en çok 10 kullanıcı, `ee/` özelliği yok.
  - Proje: kaynak dil `tr`, hedef `en`; ad alanları UI modülleri + API (`errors`, `validation`, `notifications`, `email`, `excel`) + `reference`.
  - CI (`oksis-ui/.github/workflows/ci.yml`, `oksis-api/.github/workflows/ci.yml`):
    - Her PR: `tr` dosyalarının JSON geçerliliği ve ad alanı yapısı; kodda kullanılan anahtarın `tr`'de bulunması **zorunlu** (kırmızı); `en` eksik anahtar raporu **uyarı** (iş özeti + yorum), durdurmaz.
    - `dev`'e birleşmede: yeni/değişen `tr` anahtarları Tolgee'ye yüklenir.
    - Elle tetiklenen + zamanlanmış iş: Tolgee'den `en` dışa aktarılır, yalnız `en` dosyalarını değiştiren PR açılır (`gh` CLI ile; üçüncü taraf eylem yok). `tr` dosyalarına Tolgee yazmaz — `tr`'nin sahibi koddur.
    - Dağıtım işinde eksik `en` sayısı uyarı olarak basılır.
  - `reference` ad alanı tek kaynaktan iki depoya gider (§9 soru 2 "sunucu" çıkarsa yalnız API'ye).
- **Kabul:** Tolgee'ye internetten yalnız kimlik doğrulamayla erişiliyor; bir deneme anahtarı `tr` ile koda girip Tolgee'de görünüyor, `en` çevirisi PR olarak geri geliyor; Tolgee kapatıldığında iki deponun derlemesi ve testleri etkilenmiyor.
- **Test kanıtı:** CI koşu bağlantıları (yükleme, PR açma, eksik anahtar raporu), Tolgee kapalıyken yeşil CI koşusu.

---

### Dilim E — Bekçiler, regresyon kanıtı ve kapanış

#### E1 — ESLint kuralı iskeleti
- **Sorumlu:** `oksis-web-developer`.
- **Dosyalar:** `packages/eslint-config` (yeni paylaşılan yapılandırma: JSX'te sabit metin yasağı, `react/jsx-no-literals`), `apps/web/eslint.config.js`, `apps/mobile/eslint.config.mjs`.
- **Kural:** kural **dizin listesiyle** açılır; zemin sonunda liste yalnız `packages/i18n` ve dil seçici bileşenlerini kapsar. Göç biten modül listeye eklenir (ADR).
- **Kabul:** `lint` yeşil; listedeki bir dosyaya sabit metin eklenince kırmızı (kanıt: deneme commit'i ya da kural testi).

#### E2 — Türkçe görünüm regresyonu ve `en` duman testi
- **Sorumlu:** `oksis-qa-engineer`.
- **Bağımlılık:** A, B, C birleşmeye hazır.
- **İşler:** dört rolün ana ekranları + giriş + okul ayarları + yoklama girişi (web Playwright, mobil cihaz) önce/sonra ekran görüntüsü karşılaştırması; API hata yanıtları altın test sonucu; bildirim içerikleri (push/uygulama içi/e-posta) `tr`'de aynı; `en` seçilmiş kullanıcıda bütün akışların `tr` fallback ile kırılmadan çalışması; Türkçe metin karşılaştıran mevcut testlerin kültürü sabitlenmiş olduğu (ADR "Sonuçları").
- **Kabul:** fark listesi boş ya da her fark gerekçeli; `en` duman testinde ham anahtar görünmüyor.
- **Ek:** `oksis-documents/docs/postman/` koleksiyonlarına `Accept-Language` örnekleri (ADR "Sonuçları").

#### E3 — Kapanış belgeleri
- **Sorumlu:** `oksis-tech-lead`.
- **Dosyalar (belge deposu):** `teknik/tech-stack.md` i18n satırı, `teknik/kurallar/frontend/` ve `teknik/kurallar/backend/` altında "sistem metni anahtarla yazılır" kuralı ("zemin bittiği gün kural olur"), ADR 0029'a §1.1 ad düzeltmeleri önerisi (Faruk onayıyla), bu planın silinmesi.
- **Kabul:** kural belgeleri kodla uyumlu; SESSION-HANDOFF arşive taşındı.

---

### Dilim S — Terim sözlüğü (paralel, kod dışı)

- **Sorumlu:** `oksis-education-expert` taslak + denetim, Faruk onay.
- **Amaç:** MEB terimlerinin İngilizce karşılıkları (devamsızlık, şube, nöbet, karne, rehberlik, yazılı/sınav, sözlü, performans görevi, müdür yardımcısı, sınıf öğretmeni, branş…).
- **Durum (2026-10-23):** taslak yazıldı — `teknik/kurallar/ortak/i18n-terim-sozlugu.md` (status: draft; 21 belirsiz terim S-1…S-21, 4 belge–kod çelişkisi). Kalan iş Faruk onayı ve Tolgee'ye aktarım. Tolgee sözlüğü ücretsiz sürümde yoksa (Dilim 0.3) tek kaynak bu belge olur, Tolgee'de anahtar açıklamalarıyla desteklenir.
- **Not:** Zemini engellemez; modül göçünün (madde 2) ön şartıdır. OYSS işindeki "Yazılı"/"Sınav" sözlük kararıyla bağlantılı.

---

## 5. Çalışma grafiği

**Sıralı zincirler**
- Kritik yol: **A1 → A2 → B → C → E2 → E3**. (A2 `Program.cs` ve kültür sağlayıcıyı kurar; B yerelleştiriciyi; C hem A1'in model anlık görüntüsüne hem B'nin yerelleştiricisine dayanır.)
- İstemci zinciri: **A3 (web) → A3 (mobil) → A4** ; A4 ayrıca A2'nin OpenAPI sözleşmesini bekler.
- Hat zinciri: **0.3 → D (sunucu)** ; **A3 + B → D (CI)**.

**Paralel çalışabilenler**
- Dilim 0'ın üç alt işi birbirinden bağımsız ve ilk gün başlar.
- A1/A2 (backend) ile A3 (web, mobil) eşzamanlı yürür; A3 için API gerekmez (MSW).
- D sunucu kurulumu ve S sözlüğü her şeyle paralel.
- E1, A3 birleşince başlar; kritik yolda değildir.

**Masa yükü (tahmini sıra)**
- Veri: A1 → C (göç).
- Backend: A2 → B → C (domain + çözümleyici).
- Web: 0.2 → A3 (paket + web) → A4 (web) → E1.
- Mobil: 0.1 → A3 (mobil) → A4 (mobil).
- DevOps: 0.3 → D.
- QA: her dilimin birleşme öncesi kontrolü + E2.
- İnceleme (`oksis-code-reviewer`): her dalda birleşme öncesi; A2'de `oksis-security-auditor` da.

## 6. Ortak dosyaya dokunan işlerin sırası

| Dosya | Dokunan alt işler | Sıra |
|---|---|---|
| `oksis-api/src/Oksis.Api/Program.cs` | A2, B | A2 → B (OYSS birleşmesinden sonra) |
| `Oksis.Infrastructure/Persistence/Migrations/OksisDbContextModelSnapshot.cs` + yeni göçler | A1, C | A1 → C; aynı gün iki göç açılmaz; OYSS göçleri önce |
| `Oksis.Application/DependencyInjection.cs`, `Oksis.Infrastructure/DependencyInjection` | A2, B, C | A2 → B → C |
| `Oksis.Api/Extensions/HangfireSetup.cs` | A2 | yalnız A2 |
| `Oksis.Api/Middleware/ExceptionHandlingMiddleware.cs` | A2, B | A2 → B |
| `Oksis.Api/Errors/ErrorMessageCatalog.cs` + testi | B | yalnız B |
| OpenAPI çıktısı / `oksis-ui/packages/api/src/generated/*` | A2 üretir, A4 tüketir | A2 birleşir → A4 codegen; başka açık dalın codegen'iyle aynı gün yapılmaz |
| `oksis-ui/package-lock.json` | A3 web, A3 mobil, D (CLI) | A3 web → A3 mobil → D |
| `apps/web/app/layout.tsx`, `apps/web/app/providers.tsx` | A3, A4 | A3 → A4 |
| `apps/mobile/src/app/_layout.tsx` | A3, A4 | A3 → A4 |
| `packages/api/src/client/config.ts`, `request.ts` | A3 | yalnız A3 |
| `packages/api-mocks` (me/context, school-settings) | A4 | web → mobil |
| `packages/eslint-config` | E1 | yalnız E1 |
| `.github/workflows/ci.yml` (iki depo) | D | B ve A3 birleştikten sonra |

## 7. Yeni kütüphane ve araç listesi

CLAUDE.md gereği açıkça listelenir; ilk beşi ADR 0029 ile Faruk onaylı. Listede olmayan hiçbir paket eklenmez; ihtiyaç doğarsa önce sorulur.

| Paket / araç | Yer | Gerekçe | Onay durumu |
|---|---|---|---|
| `next-intl` | `apps/web` | Web i18n (App Router, önek yok) | ADR 0029 |
| `i18next` | `apps/mobile` (gerekirse `packages/i18n`) | Mobil i18n çekirdeği | ADR 0029 |
| `react-i18next` | `apps/mobile` | React bağlayıcısı | ADR 0029 |
| `expo-localization` | `apps/mobile` (Expo SDK 57 ile uyumlu sürüm) | Cihaz dili; **yerel modül, yeni derleme ister** | ADR 0029 |
| Tolgee sunucu imajı (`tolgee/tolgee`) + PostgreSQL imajı | test VM'i (Docker) | Çeviri yönetimi, ücretsiz sürüm | ADR 0029 |
| `@tolgee/cli` | kök geliştirme bağımlılığı ya da yalnız CI'da | Push/pull | ADR akışının parçası; **paket olarak açık onay istenir** |
| `Microsoft.Extensions.Localization` (+ `.Abstractions`) | `Oksis.Infrastructure`, `Oksis.Application` | `IStringLocalizer` (ADR adıyla anıyor); Microsoft resmî paketi, ASP.NET Core ile gelir, Application için paket referansı gerekir | ADR 0029 (dolaylı) |
| Koşullu: `Intl.PluralRules` çoklu dolgusu (örn. `@formatjs/intl-pluralrules`) | `apps/mobile` | Yalnız Dilim 0.1 Hermes'te eksik bulursa | **Onay yok — gerekirse ayrıca sorulur** |

Bilinçli olarak eklenmeyenler: üçüncü taraf .NET JSON yerelleştirici paketi (kendi küçük fabrikamız yazılır), ESLint i18n eklentisi (`eslint-plugin-react` içindeki kural yeter), PR açan üçüncü taraf GitHub eylemi (`gh` CLI yeter), anahtar kod üreticisi (TS modül genişletmesi yeter).

## 8. Modül göçü için sıra önerisi (madde 2 — kapsam dışı, yalnız öneri)

ADR önceliği: veli/öğrenci → öğretmen → yönetim. Modüller birden çok yüzeye yayıldığı için birim **"modül × yüzey"** olur; bir modülün API hata/doğrulama/bildirim metinleri, o modülün ilk yüzeyiyle birlikte taşınır (`tr` fallback sayesinde güvenli).

1. **Ortak kabuk ve kimlik** (her rolün ilk gördüğü): giriş, parola değiştirme/sıfırlama, davet kabulü, hata yüzeyi, gezinme, bildirim zili ve bildirim e-posta çerçevesi (`NotificationEmail`), profil, `EmailTemplates` (davet, şifre sıfırlama). Önkoşul: Dilim S sözlüğü onaylı.
2. **Aile yüzeyleri (mobil önce):** duyurular (okuma), yemek menüsü, ders programı, devamsızlık (öğrenci/veli), notlar ve sınav takvimi (aile), ödev (öğrenci), kulüpler (öğrenci/veli), veli toplantısı (aile), etkinlikler. Her biriyle o modülün aileye giden bildirimleri.
3. **Öğretmen yüzeyleri (mobil + web):** yoklama girişi, not girişi, ödev yönetimi, kulüp danışmanlığı, nöbet, veli toplantısı (öğretmen).
4. **Yönetim yüzeyleri (web):** okul ayarları, kullanıcılar/roller/izinler, müfredat, ders atama, sezon sihirbazı, raporlar ve Excel çıktıları.
5. **Platform (merkez) yüzeyi:** §9 soru 5'e bağlı; varsayılan önerim en sona ya da kapsam dışı.

ADR madde 3 (kapanış: ESLint bütün depoda, kural belgeleri, ürün tanımı) bu planın dışında.

## 9. Faruk'a açık sorular

> **Hepsi yanıtlandı (2026-10-23).** 1 evet (bayrak arkasında, ilk aile yüzeyi göçünden sonra açılır) · 2 sunucuda · 3 ad alanı başına dosya · 4 (a) evet (b) evet (c) KVKK paket **başlığı** çevrilir, içerik Türkçe; ülke adları çevrilir; eğitim programları çevrilmez (d) **yabancı dil adları çevrilmez** · 5 hayır · 6 evet · 7 Oracle test sunucusu, `ceviri.oksis.net`, Cloudflare Access · 8 evet · 9 evet (API alt işleri OYSS 30 Ekim birleşmesinden sonra). Kütüphaneler §7'deki listenin tamamı onaylı. Terim sözlüğü onaylı. Aşağıdaki metin kayıt içindir.

1. **Seçicinin üretimde görünürlüğü:** Zemin bitince dil seçici ve okul varsayılan dili ayarı üretimde bayrak arkasında kapalı kalsın, ilk aile yüzeyi göçü bitince mi açılsın? (Öneri: evet. Aksi hâlde `en` seçen kullanıcı neredeyse tamamen Türkçe arayüz ve karışık doğrulama cümleleri görür.)
2. **`NameKey` nerede çözülsün?** Öneri: **sunucuda** — API `name` alanını etkin dilde döndürür, sözleşme değişmez, Excel ve bildirimler aynı çeviriyi kullanır, `reference` ad alanı tek depoda yaşar. ADR "arayüz anahtarlarıyla aynı dosyalarda çevrilir" diyor; istemci çözümü seçilirse her DTO'ya `nameKey` eklenir ve `reference` ad alanı iki depoya kopyalanır.
3. **Mesaj dosyası düzeni:** ADR `tr.json` / `en.json` diyor. Öneri: ad alanı (modül) başına ayrı dosya. Gerekçe: ~17 bin satırlık tek dosya, paralel modül göçlerinde her PR'da çakışma üretir. Ad alanı = modül ilkesi aynen korunur.
4. **Referans listesi farkları:** (a) `AcademicTermType` ve `GradeScale` eklensin mi (öneri: evet)? (b) `SystemSetting` açıklaması çıkarılsın mı (öneri: evet, ekranda yok)? (c) KVKK paket başlığı (`ConsentBundle`), ülke adları (`Country`), eğitim programı adları (`EducationProgram`) çevrilsin mi? Öneri: KVKK hukuki görüşe kadar hayır; ülke adları evet; eğitim programları hayır. (d) Yabancı dil adı çevrilirken aynı adlı branş çevrilmeyecek ("English" dil seçeneği, "İngilizce" branş). Kabul mü?
5. **Platform (merkez) yüzeyi** (`PlatformAccount`) v1'de dil desteği alacak mı? Öneri: hayır; yalnız `Accept-Language`/çerez, metinler `tr`.
6. **Sunucu kültürü:** İstekte yalnız UI kültürü değişsin, `CurrentCulture` `tr-TR` kalsın (Türkçe arama/sıralama korunur, TB-271). Onaylıyor musun?
7. **Tolgee'nin yeri ve adresi:** Oracle A1 test VM'inde mi (bellek test ortamıyla paylaşılır), ayrı makinede mi? Alan adı (örn. `ceviri.oksis.net`)? Tolgee'nin kendi girişine ek olarak Cloudflare Access kapısı istenir mi?
8. **Biçimleyici yolu:** Dilim 0.1 Hermes'te `Intl`'in güvenilmez olduğunu gösterirse, ADR'deki "ortak tarih/sayı biçimleyicileri (`Intl`)" ifadesi "saf TS tablolar (`tr` + `en`)" olarak düzeltilsin mi?
9. **Zamanlama:** API alt işleri (A1, A2, B, C) OYSS'nin 30 Ekim dev birleşmesinden sonra mı başlasın? Dilim 0, A3, D-sunucu ve S hemen başlayabilir.

## 10. Riskler

| Risk | Etki | Önlem |
|---|---|---|
| İstek kültürünün Türkçe arama/sıralamayı bozması | İngilizce kullanıcıda "VELİ" bulunmaz, sıralama değişir | Yalnız UI kültürü değişir (§9 soru 6); A2'ye arama entegrasyon testi |
| Hangfire kültür yakalama | Arka plan bildirimleri yanlış dilde | Varsayılan kültür sabitlenir; işte metin alıcı diliyle açıkça üretilir (B) |
| Dış ara katmanda kültür görünmemesi | İstisna mesajları yanlış dilde | `ExceptionHandlingMiddleware` kültürü kendisi çözer (A2) |
| Hermes `Intl` güvenilmezliği | Mobilde tarih/sayı ve çoğullar yanlış | Dilim 0.1 ölçümü; gerekirse saf TS tablo ya da onaylı çoklu dolgu |
| Worker boyut sınırı | Web dağıtımı kırılır | Dilim 0.2 ölçümü; yalnız etkin dilin mesajları yüklenir |
| Yerel modül (`expo-localization`) | OTA ile gitmez | Mobil sürüm planına yeni derleme eklenir |
| Tek mesaj dosyasında çakışma | Göç dalları sürekli çatışır | Ad alanı başına dosya (§9 soru 3) |
| OYSS ile aynı dosyalar | Göç ve `Program.cs` çatışması | Ortak dosya sırası (§6), API işleri OYSS'den sonra |
| Tolgee arm64 / bellek / ücretsiz sürüm sınırları | Kurulum gecikir, sözlük yok | Dilim 0.3; Tolgee kritik yolda değil, JSON dosyaları her zaman depoda |
| Tolgee'den gelen PR'ın `tr`'yi ezmesi | Kaynak metin bozulur | Çekme işi yalnız `en` dosyalarına yazar; `tr` değişikliği CI'da reddedilir |
| Kalıcı bildirim metni | Dil değişince eski bildirimler eski dilde | Kabul; kullanıcı belgesinde not |
| Sistem metninin kullanıcı verisi olarak yazılması (`"Müdür"` vb.) | Göçten sonra da Türkçe kalır | Modül göçünde envanter maddesi |
| Erken `en` seçimi | Karışık dil deneyimi | Seçici bayrak arkasında (§9 soru 1) |

---

## Onay sonrası ilk adımlar (onay gelmeden başlamaz)

1. Dilim 0'ın üç alt işi ve Dilim S aynı gün başlar.
2. §9 yanıtlarıyla A1/A2 brifingi backend + veri masalarına, A3 brifingi web masasına verilir.
3. SESSION-HANDOFF "Aktif dilim" bölümü bu plana bağlanır.
