---
tags: [decision, domain/attendance, domain/exams, teknik/kvkk]
date: 2026-10-26
status: accepted
---

# 0030 — Mazeret gerekçesi saklama ve imha: yoklama mazereti metni ve belgesi okul ayarıyla temizlenir, mazeret sınavında kategori ve belge işareti nötrlenir

**Durum:** Kabul edildi (Faruk, 2026-10-26; kararlar aşağıda). Hukuk teyidi bekliyor. Hukuk görüşü beklenmeden hazırlandı (Faruk, 2026-10-26). Süreler ayarla değiştirilebilir.

**İlgili:** [[0028-merkezi-sinav-takvimi-oyss]] M1, M12 ve açık sorular. Ön görüş taslağı: `teknik-analizler/oyss/mazeret-gerekce-saklama-on-gorus.md` §5, §7 açıklar 1, 2, 4, 5.

## Bağlam

- **Mazeret sınavı.** Terminal ya da bayat kayıtta serbest metin 180 gün sonra temizleniyor (`ExamDailySweepJob.cs:184-243`, `MakeupExam.cs:376-421`). Ama `ReasonCategory` (Sağlık dahil) ve `DocumentSeenOnPaper` süresiz kalıyor (`MakeupExam.cs:68`, `:74`). Süre sunucu genelinde tek ayar (`ExamDailySweepJob.cs:186`). Temizlik denetim izine yazılmıyor; `MakeupAuditAction` içinde bu iş için bir değer yok (`MakeupAuditAction.cs:4-17`).
- **Yoklama mazereti (`AttendanceExcuse`).** Şu alanlar hiç temizlenmiyor:
  - `Reason`: zorunlu, en çok 500 karakter (`AttendanceExcuse.cs:28`, `AttendanceExcuseConfiguration.cs:30`).
  - `DecisionNote` (`:38`).
  - `StoredFileId`, `DocumentName`, `DocumentMeta` (`:43-48`).
  - Bu metin, M1 otomatik kabulünde mazeret sınavının da gerekçesidir (`MakeupExam.cs:79-80`, `:194-205`).
- **Belge dosyası.** `AttendanceExcuseDocument` kategorisinin kendi süresiyle siliniyor: sezon sonu + 730 gün sonra silinmeye işaretlenir (`FileCategoryPolicyRegistry.cs:69-72`, `RetentionEnforcementJob.cs:155-210`), 30 gün sonra fiziksel olarak silinir (`SoftDeletePurgeJob.cs:56`, `:95-111`). Özgün dosya adı yalnız `StoredFile.OriginalFileName`'de duruyor (`CreateExcuseCommandHandler.cs:149-154`).
- **Geri alınan mazeret satırı.** Belge iliştirme başarısız olursa mazeret satırı yumuşak silinir (`CreateExcuseCommandHandler.cs:144`). Kiracı süzgeci bu satırı gizlediği için bir temizlik işi onu görmez; satır ise metni taşır.
- **Okul bazlı emsal.** Veli toplantısı notunun saklama süresi okul ayarı (`SchoolSettings.cs:408`, alt sınır `ParentMeetingPolicy.cs:45`, iş `ParentMeetingNoteRetentionJob.cs:64-69`). O iş her imhada sistem aktörlü denetim satırı ve okul başına `data.delete.kvkk` olayı yazıyor (`:105-121`).

## Seçenekler: yoklama mazereti temizliği nerede koşar

| | A. `ExamDailySweepJob`'a adım | B. Ayrı `AttendanceExcuseRetentionJob` (öneri) | C. Genel KVKK saklama orkestratörü |
|---|---|---|---|
| Tenant izolasyonu | Var olan okul döngüsü ve `SetForLoginFlow` | Aynı iskelet (`db.Schools` → `SetForLoginFlow` → açık `SchoolId`) | Aynı, ama tek işte çok modül |
| Modül bağımlılığı | Exams, Attendance ve Documents verisini değiştirir: sınır ihlali. Mazeret sınavı olmayan okulda da gerekir | Attendance sahiptir. Exams'e ve Documents'a yalnız soyutlamayla gider | Her modül bir adım arayüzü uygular. Temiz, ama bugün tek kullanıcısı var |
| Göç maliyeti | 1 kolon | 1 kolon ve 1 dizin | Aynı kolonlar, artı orkestratör altyapısı |
| Geri alma | Adım silinir | İş kaydı silinir | Altyapı kalır |

## Karar (öneri)

### 1. Yoklama mazereti

**Ne silinir.**

- `Reason` alanına `"—"` yazılır. Bu, mazeret sınavında iptal gerekçesi için kullanılan dilden bağımsız işaretle aynıdır (`MakeupExam.cs:38`, `:394`). Kolon NOT NULL kalır, DTO tipi değişmez.
- `DecisionNote`, `StoredFileId`, `DocumentName` ve `DocumentMeta` null olur.
- Yeni `ReasonClearedAt` alanına imha anı yazılır.
- Kayıt, durum, tarihler, `Type` ve `AffectedSessionCount` kalır. `Type = MedicalReport` resmî devamsızlık kaydının parçasıdır ve yoklama satırındaki "Raporlu" durumuyla tutarlı kalmalı; nötrleştirilmez. Bu nokta ön görüşteki hukuk sorularına eklenmeli.

**Süre ne zaman işler.** Üç koşulun hepsi aranır:

1. Karar verilmişse `DecidedAt`, bekleyen kayıtta `SubmittedAt` eşikten (şimdi eksi süre) eski olmalı.
2. Mazeretin bitiş günü (`EndDate`) eşik gününden eski olmalı. Uzun raporlar bu sayede korunur.
3. **Dönem kapanış koruması:** okulun kapanmış son döneminin bitişi L ise `EndDate <= L` olmalı (`AcademicTerms`, emsal `CreateMakeupFollowUpCommandHandler.cs:59-63`). Böylece süre ne kadar kısaltılırsa kısaltılsın kayıt dönem kapanmadan silinmez (ön görüş §3 asgari koşulu).

**Bekleyen kayıt.** Metni temizlenmiş bekleyen mazeret artık karara bağlanamaz: `Decide` yeni `Attendance.Excuse.ReasonCleared` hatasını verir.

**Bağlı mazeret sınavıyla ilişki.**

- Mazeret, ona işaret eden ve gerekçesi henüz temizlenmemiş (`ReasonClearedAt == null`) en az bir `MakeupExam` varken **bekletilir**.
- Ters yön gerekmez: mazeret sınavı metnini kendi saatiyle siler. Bağ kimliği iki tarafta da kalır ve zararsızdır.
- **Tavan:** bekletme, mazeretin kendi süresinin 2 katını geçince yok sayılır. Kilitlenmeyen bir pencerede askıda kalan sınav kaydı mazereti sonsuza dek tutmasın.

**Yüklenmiş belge.**

- Metinle aynı işlemde dosyanın mazeretle bağı (`FileAttachment`) silinir.
- Dosya başka bir varlığa bağlı değilse silinmeye işaretlenir (`MarkSoftDeleted`, `StoredFile.cs:132-144`) ve kayıttan sonra okulun kotası serbest bırakılır (`ReleaseAsync`).
- Fiziksel silmeyi var olan `SoftDeletePurgeJob` 30 gün sonra yapar ve özgün adı taşıyan satırı da siler. Önizleme dosyaları TB-40 yetim taraması ile üst dosyayı izler (`RetentionEnforcementJob.cs:100-153`).
- 730 günlük kategori süresi bir alt güvence olarak yerinde kalır.

**İş.** Yeni `AttendanceExcuseRetentionJob`:

- Günlük cron `55 4 * * *`; veli toplantısı işinden 5 dakika sonra.
- 5 deneme, okul başına try/catch.
- 100 kayıtlık partiler; her parti tek `SaveChanges`.
- Açma anahtarı: `Attendance:ExcuseRetention:Enabled`.

### 2. Mazeret sınavı: kategori ve belge işareti

`ClearSensitiveReason` ve `ClearStaleSensitiveReason` metodları bunlara ek olarak `ReasonCategory = null` ve `DocumentSeenOnPaper = false` yazar.

| Seçenek | DTO ve istemci etkisi | Karar |
|---|---|---|
| Enum'a `Expired` değeri | Yeni tel değeri `"expired"`. `MakeupWire.Category` ve `ParseCategory` (`MakeupWire.cs:54-68`), contract.ts birleşim tipi ve istatistikler değişir | Hayır |
| **null** | `reasonCategory` ve `documentSeenOnPaper` zaten boş olabilir (`MakeupDtoBuilder.cs:101-102`). Tel değişmez | **Evet** |

"Bağlıydı" ile "silindi" ayrımı için DTO'ya **eklemeli** `reasonCleared: bool` alanı gelir (`ReasonClearedAt != null`). Arayüz bu alan doluyken "Saklama süresi doldu" yazar (i18n anahtarı, tr ve en). Aynı alan `ExcuseDto`'ya da eklenir (`AttendanceDtos.cs:123-141`).

Domain'deki "bağ yoksa kategori zorunlu" kuralı yalnız oluşturmada doğrulanıyor (`MakeupExam.cs:462-504`); null'a çekmek bu kuralı bozmaz. Uygunluk listesindeki kategori (`ListMakeupEligibleQueryHandler.cs:83`, `:147`) eski terminal kayıtta zaten null döner.

### 3. Süre ayarı

**Yer.** Okul bazlı tek alan: `SchoolSettings.ExcuseReasonRetentionDays`. Hem yoklama mazereti hem mazeret sınavı bunu okur. Gerekçeler:

- Veri sorumlusu okuldur (ön görüş §6 s.5).
- Veli toplantısı emsaliyle tutarlıdır.
- Tek alan iki kaydın saatini uyumlu tutar ve bekletme kuralını basitleştirir.

**Sınırlar.**

- Sabitler `Oksis.Domain.Modules.Schools.Policies.ExcuseRetentionPolicy` içinde: öntanım 180 gün, alt sınır 30, üst sınır 730.
- Alt sınırın asıl güvencesi dönem kapanış korumasıdır. 30 gün, ön görüşteki "dönem sonu + yaklaşık 30 gün" ile örtüşür.
- Okuyucu, emsaldeki gibi sınır altındaki değeri sınıra çeker (`ParentMeetingNoteRetentionJob.cs:69`).

**Sunucu ayarı.** `Exams:Makeup:ReasonRetentionDays` kalkar. Göçten önce ortamlarda 180'den farklı bir değer verilmişse devops kontrol etmeli: göç yalnız 180 yazar.

**Değiştirme.** Yeni okul ayarı komutu, kısaltmada onay ister (emsal `SchoolSettings.cs:405-406`). Yetki var olan okul ayarı yetkisidir; yeni yetki gerekmiyor.

### 4. İmha kaydı

**Mazeret sınavı.**

- `MakeupAuditAction.ReasonCleared = 5`.
- `MakeupAuditEntry.ActorAccountId` boş olabilir hale gelir; sistem aktörü null'dır (emsal `ParentMeetingNoteRetentionJob.cs:105-107`). Diğer yol Guid.Empty sabitiydi; açık olduğu için null seçildi.
- Metin ve kategori kolonu yok (`MakeupAuditEntry.cs:11-13`).

**Yoklama mazereti.** Kayıt düzeyinde iz satırın kendisindeki `ReasonClearedAt`'tir. Ayrı bir iz tablosu açılmaz; satır silinmediği için yeterli.

**Okul özeti.**

- İş başına tek `IAuditLogger` olayı: `data.delete.kvkk`.
- Yük: `SchoolId`, `Excuses`, `Files`, `Makeups`, `RetentionDays`, tarih.
- Öğrenci, kişi ve metin yok. Log satırına yalnız sayılar yazılır.

### 5. Tenant izolasyonu

- İki iş de `db.Schools` üzerinden okul döngüsü kurar (School kiracı varlığı değil), sonra `SetForLoginFlow(schoolId)` çağırır. Her sorguya `SchoolId == schoolId` açıkça yazılır. Okul sonunda ve hatada `ChangeTracker.Clear()` çağrılır (`ExamDailySweepJob.cs:131`, `:141`).
- **`IgnoreQueryFilters` tek yerde gerekir:** yumuşak silinmiş mazeret satırları (`CreateExcuseCommandHandler.cs:144` geri alma yolu). Süzgeç bu satırları gizler; ama satır metni taşıyor.
  - Gerekçe: yalnız silinme bacağı aşılır; tenant izolasyonu açık `SchoolId` eşitliğiyle korunur (emsal `SoftDeletePurgeJob.cs:20-27`, `:80-88`).
  - Kodda `// SECURITY:` yorumu zorunlu.
- Bekletme ve dosya soyutlamaları `schoolId` parametresi alır, kendi sorgularına da eşitliği yazar.
- `TenantSaveChangesInterceptor` başka okula yazmayı ayrıca engeller.

**Modüller arası soyutlamalar** (tüketen modül arayüzün sahibidir; emsal `IExamAttendanceReader`, Exams'te tanımlı, `Attendance/Common/ExamAttendanceReader.cs:65`'te uygulanır):

| Arayüz | Tanım | Uygulama | Biçim |
|---|---|---|---|
| `IExcuseRetentionHoldReader` | `Application/Modules/Attendance/Abstractions/` | Exams `Internal/` | Okul ve mazeret kimlikleri alır, bekletilen kimlikleri döndürür |
| `IEntityFileRetirer` | `Application/Modules/Attendance/Abstractions/` (ya da Documents ortak arayüzü) | Documents | Okul, varlık tipi `"AttendanceExcuse"`, varlık kimliği ve an alır. Değişiklikleri izlemeye koyar, serbest bırakılacak kota boyutlarını döndürür. Kayıt işi yapar |

Not: `CreateExcuseCommandHandler.cs:110-143` bugün Documents handler'larını doğrudan kuruyor. Bu, soyutlama kuralına aykırı bir emsaldir; bu kararla düzeltilmez, takip listesine girer.

### 6. Göç taslağı

Belge deposu kod barındırmadığı için DDL tabloyla verilir.

| Tablo | Değişiklik | Tip, boşluk, öntanım | Geri alma |
|---|---|---|---|
| `academic.attendance_excuses` | `reason_cleared_at` ekle | datetimeoffset, NULL | Kolonu düşür |
| `academic.attendance_excuses` | Süzgeçli dizin `(school_id, decided_at, end_date)`, koşul `reason_cleared_at IS NULL` | — | Dizini düşür |
| okul ayarları tablosu (`SchoolSettingsConfiguration.cs:317` emsali) | `excuse_reason_retention_days` ekle | int, NOT NULL, öntanım 180 | Kolonu düşür |
| `academic.makeup_audit_entries` | `actor_account_id` boş olabilir yap | uniqueidentifier, NULL | Boşları Guid.Empty ile doldurup NOT NULL yap |
| `academic.makeup_exams` | DDL yok | `reason_category` zaten boş olabilir | — |

**Veri göçü yok** (Faruk kararı 3): önceden temizlenmiş mazeret sınavı kayıtlarında kategori ve işaret olduğu gibi kalır; nötrleştirme yalnız bu sürümden sonra temizlenen kayıtlara uygulanır.

**Geri alınabilirlik.**

- Şema eklemeli ve ucuz.
- Temizlik, nötrleştirme ve dosya imhası tasarım gereği geri alınamaz; dönüş yalnız yedekten olur. Yedekten dönülürse iş idempotent olduğu için aynı kayıtları yeniden temizler (emsal `ParentMeetingNoteRetentionJob.cs:20-21`).
- Göçler iki PR'da, sırayla eklenir.

## Agregat sınırı

| Agregat | Değişiklik | Sınır kuralı |
|---|---|---|
| `AttendanceExcuse` (Attendance) | `ReasonClearedAt`, `ClearSensitiveContent(at)` (idempotent), `Decide` koruması | Dosyayı kendisi silmez; yalnız referansı boşaltır |
| `MakeupExam` (Exams) | Temizlik metodları kategori ve işareti de nötrler | Mazereti okumaz, yalnız bekletme sorusuna yanıt verir |
| `MakeupAuditEntry` | `ReasonCleared` işlemi, aktör boş olabilir | Yalnız ekleme |
| `StoredFile` ve `FileAttachment` (Documents) | Değişiklik yok, var olan `MarkSoftDeleted` kullanılır | Fiziksel silme yalnız `SoftDeletePurgeJob`'da |
| `SchoolSettings` (Schools) | `ExcuseReasonRetentionDays` ve güncelleme metodu (sınır doğrulamalı) | Okulun tek politika satırı |

## Sonuçlar

- M12 yoklama mazeretini de kapsar; [[0028-merkezi-sinav-takvimi-oyss]] içindeki "yoklama mazeretlerinin serbest metinleri" açık sorusu kapanır.
- Belge metinle birlikte silinir: metnin temizlendiği günden en geç 30 gün sonra fiziksel imha olur.
- Yedeklerde kalma süresi (ön görüş §6 s.6), denetim izinin ve durum alanlarının saklanması (açık 6), istatistik ayrıştırması ve `kvkk.md` eksikliği (açık 7) bu kararın dışında.
- Sağlık dışı kategoride sağlık bilgisi yazılması (açık 3) ürün kararı olarak ayrı kalır.

## Geri alma maliyeti

**Düşük-orta.** Kod ve şema kolay döner. Pahalı olan, imha edilmiş veridir; geri gelmez. Bu yüzden prod'da açma anahtarı Faruk kararıyla açılmalı.

## Dilimleme

| PR | Dal | İçerik | Ajan |
|---|---|---|---|
| 1 | `feature/mazeret-saklama-ayar-ve-sinav` | Okul ayarı (kolon, politika, komut ve doğrulayıcı, okuma DTO'su); `ExamDailySweepJob` okul değerini okur ve dönem koruması ekler; kategori ve işaret nötrleştirme; `ReasonCleared` iz kaydı, aktör boş olabilir; veri göçü; `reasonCleared` DTO alanı; `data.delete.kvkk` olayı | `oksis-backend-developer` (üst model), ardından `oksis-security-auditor` ve `oksis-code-reviewer` |
| 2 | `feature/yoklama-mazereti-saklama` (PR1'den sonra) | `AttendanceExcuse` alanları ve metodları; iki soyutlama ve uygulamaları; `AttendanceExcuseRetentionJob` ve Hangfire kaydı; açma anahtarı; `ExcuseDto.reasonCleared` | Aynı zincir. Tenant ve `IgnoreQueryFilters` güvenlik denetiminde zorunlu madde |
| 3 | `feature/mazeret-saklama-arayuz` (oksis-ui) | Okul ayarı ekranında süre alanı ve kısaltma onayı; mazeret ve mazeret sınavı ekranlarında "Saklama süresi doldu" (tr ve en anahtarları); mobil Onay Merkezi'nde belge açma düğmesinin gizlenmesi; codegen | `oksis-web-developer`, `oksis-mobile-developer`, `oksis-browser-verifier` |

## Test listesi

**Domain**

- Mazeret sınavı temizliği kategoriyi ve işareti boşaltır; ikinci çağrı etkisizdir; bayat kayıt yolu aynı davranır; bayat temizlikten sonra iptal edilen kayıt yeniden temizlenir.
- Mazeret temizliği: işaret yazılır, beş alan boşalır, idempotenttir, temizlenmiş kayda karar verilemez.
- Ayar sınırları: 29 reddedilir, 30 ve 730 kabul edilir, 731 reddedilir.

**Application**

- Ayar komutunun doğrulaması ve yetkisi.
- Bekletme okuyucusu yalnız aynı okulda ve gerekçesi temizlenmemiş kayıtları döndürür.
- Dosya soyutlaması: bağı siler; dosya başka varlığa bağlıysa dosyayı silinmeye işaretlemez; kota boyutunu döndürür.

**Entegrasyon (Docker)**

- İki okul, iki farklı süre: her okul kendi süresini kullanır.
- Kapanmamış dönemdeki kayıt silinmez.
- Açık bağlı mazeret sınavı mazereti bekletir; tavan süresi dolunca bekletme kalkar.
- Yumuşak silinmiş mazeret satırı temizlenir, başka okulun satırına dokunulmaz.
- A okulunun turu B okuluna yazmaz.
- İkinci koşu 0 kayıt işler.
- Parti devam eder.
- İz satırı sayısı temizlenen kayıt sayısına eşittir.
- Olay yükünde yalnız sayılar var; yakalanan loglarda metin geçmez.
- 30 gün sonra `SoftDeletePurgeJob` depodan siler; temizlenmiş mazeretin indirme adresi 404 döner.
- Önceden temizlenmiş satırların kategori ve işareti değişmez (veri göçü yok).
- Mimari bekçi: Attendance, Exams ve Documents somut tiplerine başvurmaz.

## Faruk'a sorulan kararlar

**Cevaplar (2026-10-26):** 1 öneri gibi (okul bazlı, 180 / 30–730, dönem koruması); 2 evet, belge metinle birlikte; 3 **geçmişe dokunulmaz**: mazeret sınavı için veri göçü yapılmaz. Açılış anahtarı önerildiği gibi (dev/test açık, prod'da Faruk).

1. **Süre ayarının yeri ve sınırları.** Öneri: okul bazlı tek alan, öntanım 180 gün, aralık 30–730, dönem kapanış koruması. Sunucu ayarı kaldırılır.
2. **Yüklenmiş belge metinle aynı anda mı silinsin?** Öneri: evet. Dosya silinmeye işaretlenir, 30 gün sonra depodan fiziksel olarak silinir. 730 günlük kategori süresi alt güvence olarak kalır.
3. **Geriye dönük uygulama ve açılış.** Öneri: daha önce temizlenmiş mazeret sınavı kayıtlarında kategori ve işaret göçle nötrlensin. Yoklama mazereti işi dev ve test ortamında açık gelsin; prod'da `Attendance:ExcuseRetention:Enabled`'ı Faruk açsın. Açılan ilk gece eski kayıtlar partiler halinde temizlenir.

## Belge ile kod çelişkileri (bu kararla bulunan)

- `teknik/mimari/background-job-rules.md` §6 işin `SchoolId` argümanı ve `OverrideForSuperAdmin` ile çalışmasını söylüyor; kod okul döngüsü ve `SetForLoginFlow` kullanıyor (`ExamDailySweepJob.cs:42-46`, `:87`). Belge düzeltilmeli.
- `SchoolSettings.cs:405` yorumu "Aralık 0-10; varsayılan 2" diyor; kodda alt sınır 3, öntanım 3 (`ParentMeetingPolicy.cs:44-46`, `SchoolSettings.cs:974-979`). Yorum bayat.
