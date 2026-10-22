
| | |
|---|---|
| **Belge türü** | Teknik tasarım: veri modeli, API sözleşmesi ve dilim planı. Backend, web ve mobil bu belgeden paralel çalışır. |
| **Modül** | Yemekhane (katalog anahtarı `cafeteria`). MVP kapsamı: Yemek Menüsü (`mealMenu`) |
| **Tarih** | 15 Ekim 2026 |
| **Durum** | **Öneri.** Faruk'un onayını bekliyor. Kararlar §1'de, Faruk'a giden tek açık soru §10'da. |
| **Girdi** | `ihtiyac-analizleri/yemek-menusu/OKSİS — Yemek Menüsü Modülü · İhtiyaç Analizi (Taslak).md`. §9'daki Y-1…Y-18 önerilerinin tamamı kabul edildi; §7.1 M-1…M-11 ve §8 BR-YM-01…23'te köşeli parantezli yerlerde öneri olan seçenek geçerli. |
| **Kod tabanı** | `oksis-api-yemek-menusu` (`feature/yemek-menusu`), `oksis-ui-yemek-menusu`. Taramanın tarihi 2026-10-15. Modül için kod yok. |
| **Terminoloji** | Analiz §11 geçerli: `cafeteria` (modül anahtarı), `mealMenu`, `mealPeriod` (öğün), `dish` (yemek), `dishCatalog`, `mealMenuGroup`, `allergen`. Bu adlar kodlanmadan önce `oksis-ui/CLAUDE.md` sözlüğüne eklenmelidir (W0 dilimi). |

> **Okuma sırası:** Backend §2–§6 ve §9'u, web ve mobil §3, §4, §7 ve §9'u okusun. Tel sözleşmesinin tek doğru kaynağı §3'tür. Codegen gelene kadar istemci tipleri `packages/api/src/meal-menu/contract.ts` içine buradan kopyalanır.

---

## 0. Koddan doğrulanan mevcut durum (tasarımı etkileyen bulgular)

| # | Bulgu | Kanıt | Tasarıma etkisi |
|---|---|---|---|
| D-1 | **`cafeteria` plan kataloğunda yok.** `master.plan_modules` yalnız attendance, marks, announcements, homework, messaging ve reports satırlarını taşıyor. `PlanModuleResolver` envanteri bu katalogdan kurduğu için `cafeteria` Ayarlar › Modüller listesine hiç gelmiyor. `UpdateModuleConfigCommandHandler` de açma isteğini 409 `plan-required` ile reddediyor. | `Migrations/20261007182449_20261007_baseline.cs` satır 8027–8042, `PlanModuleResolver.cs`, `UpdateModuleConfigCommandHandler.cs` satır 86–96 | Analiz §5'teki "API'de varsayılan açık" bilgisi **fiilen geçersiz**: tohum satırı `true` yazıyor ama modül ne listeleniyor ne açılabiliyor. Göç gerekli (§6.3). |
| D-2 | **Sunucuda modül kapısı yok.** Hiçbir handler "modül kapalı mı" diye sormuyor. UI'da da gezinme modül durumuna bakmıyor; `useModuleConfigs` yalnız Ayarlar ekranında kullanılıyor ve `school-settings.view` izni istediği için veli, öğrenci ve öğretmen onu çağıramıyor. | `GetModuleConfigsQuery.cs` (`[RequirePermission("school-settings.view")]`), `packages/core/src/nav/nav-config.ts` | BR-YM-21 için yeni bir okuma soyutlaması ve rol bağımsız bir sinyal (`me/context`) gerekiyor (§6.1). |
| D-3 | **Belge ile kod arasında çelişki.** `PlanModuleResolver` yorumu "`school_module_configs` hiçbir yerde doldurulmuyor" diyor. Oysa `SeedDefaultModuleConfigsHandler` yeni okulda bu tabloyu dolduruyor. | iki dosyanın yorumları | Göçte iki durum da ele alınıyor: satırı olan okul ve satırı olmayan okul (§6.3). |
| D-4 | **İzinler yalnız rol üzerinden geliyor.** `RolePermission` platform genelinde tek matris. Okula ya da kişiye izin tanıma mekanizması yok. Tohumlanmış okul rolleri yalnız SchoolAdmin, VicePrincipal, Teacher, Parent ve Student. Sekreter ve personel rolleri yok. | `RolePermission.cs` ("tenant-bazlı override ileri sürüm"), `AccountPermissionResolver.cs`, `MasterSeedIds.Roles` | Y-9 (b)'deki "izin verilen personel" bugünkü altyapıyla kurulamıyor. **Faruk'a soru §10.** |
| D-5 | Öğrenciden kademeye giden yol kesin: `StudentProfile.CurrentClassroomId` → `ClassRoom.GradeLevelId` → `GradeLevel.EducationLevel` (master). Hazırlık sınıfı `High` sayılıyor. | `GetMyExamScheduleQueryHandler.cs`, `ClassRoom.cs`, `GradeLevel.cs` | Grup çözümü kayıtta saklanmaz, her okumada türetilir (BR-YM-04). |
| D-6 | Tatil okuma soyutlaması hazır: `IHolidayCalendarReader.GetHolidaysInRangeAsync` okul tatillerini ve resmî tatilleri birlikte döner. Okul saat dilimine göre "bugün" bilgisini `ISchoolCalendarService.GetLocalNowAsync` veriyor. | `Common/Abstractions/Calendars/IHolidayCalendarReader.cs`, `Attendance/Abstractions/ISchoolCalendarService.cs` | Tatil için yeni okuyucu yazılmaz. |
| D-7 | Bildirim hattı şöyle işliyor: `INotificationEnqueuer.Enqueue(eventId, …)` çağrısı yapılır ve teslim kaydı deterministik `eventId` ile tekilleşir. Push kapsamını `PushEventKeyMap` ve katalog belirler. K-34'e göre kapsamdaki her anahtarın varsayılanı açık olmalı. Cooldown için Redis desenini `ICacheService.ExistsAsync/SetAsync` sağlıyor. | `INotificationEnqueuer.cs`, `PushEventKeyMap.cs`, `AbsenceThresholdReachedNotificationHandler.cs`, `notification-rules.md` §7 | Birleştirme mevcut cooldown deseniyle yapılır (§5.4). |

---

## 1. ADR: kritik kararlar

> Her karar şu sırayla anlatılır: bağlam, seçenekler, karar, sonuçlar, geri alma maliyeti. Her seçenek için tenant izolasyonu, modüller arası bağımlılık ve göç maliyeti ayrıca değerlendirildi. Faruk onaylarsa ADR'ler `domain/kararlar/` altına ayrı notlar olarak taşınabilir.

### ADR-YM-1: Menü verisi gün bazlı tutulur (`MealMenuDay` kökü)

- **Bağlam.** Yayın birimi serbest bir tarih aralığı (Y-3). Kopyalama gün ve hafta düzeyinde yapılıyor (M-5). Son dakika değişikliği tek günü etkiliyor ve "değişti" bildirimi bugün/yarın eşiğine bağlı (Y-10).
- **Seçenekler.**
  - (a) **Grup × tarih başına bir kök** (`MealMenuDay`), altında öğün kayıtları ve yemek satırları.
  - (b) Ay ya da hafta başına bir plan belgesi, günler JSON içinde.
  - (c) Gün × öğün başına bir kök.
- **Karar: (a).** (b) seçeneğinde yayın birimi belgeye kilitlenir, Y-3 bozulur. Tek bir günün düzeltilmesi de bütün ayın satırını kilitler. JSON içindeki yemeğe yabancı anahtar konamaz, bu yüzden BR-YM-06'daki "kullanımda mı" kontrolü OPENJSON gerektirir. (c) seçeneğinde gün notu ve gün durumu öğünler arasında dağılır. Yayın ve geri çekme gün düzeyindedir, öğün düzeyinde değildir.
- **Tenant.** Her tabloda `school_id` bulunur. Tekil indeks `school_id` ile başlar. Kök ve çocuklar aynı okulun satırlarıdır.
- **Modül bağımlılığı.** Yok.
- **Göç maliyeti.** Yeni tablolar eklenir (§2.3), veri taşınmaz.
- **Geri alma maliyeti.** Düşük. Model ay belgesine çevrilmek istenirse günler gruplanarak türetilebilir. Ters yön kayıpsızdır.

### ADR-YM-2: Yemek katalogda referansla bağlanır; geçmişin doğruluğu denetim anlık görüntüsüyle korunur

- **Bağlam.** BR-YM-06 kabul edildi: katalogda alerjen değişirse bu tüm günlere yansır. Analiz §10 bir risk de not ediyor: o gün veliye ne söylendiği kaybolabilir.
- **Seçenekler.**
  - (a) Günde yalnız `dish_id` tutulur, gösterim katalogdan okunur.
  - (b) Günde ad ve alerjen kopyası tutulur, katalog yalnız giriş kolaylığı sağlar.
  - (c) (a) uygulanır, ayrıca her yazma ve yayında gün denetim satırına önce/sonra anlık görüntüsü (JSON) yazılır.
- **Karar: (c).** (b) BR-YM-06 ile çelişir: alerjen düzeltmesi geçmiş ve gelecek günlere yansımaz, idarenin bir yemeğin alerjenini 30 günde 30 kez düzeltmesi gerekir. (c) görüntüyü katalogdan besler, denetim izi ise "o gün ne yayındaydı" sorusuna cevap verir. Ad değişikliği de aynı kurala tabidir: yazım hatası bir kez düzeltilir.
- **Tenant.** Yemek okul kapsamındadır. Yemek adı okul içinde tekildir (`ux (school_id, normalized_name)`).
- **Modül bağımlılığı.** Yok.
- **Göç maliyeti.** Yok.
- **Geri alma maliyeti.** Orta. (b)'ye geçmek için satırlara kolon eklenip katalogdan geri doldurmak gerekir. Bu tek bir göçtür ama BR-YM-06'yı değiştirir, yani bir ürün kararı gerektirir.

### ADR-YM-3: Menü grubu yalnız **kademeye** (EducationLevel) eşlenir; seviye eşlemesi ileride eklenebilir

- **Bağlam.** M-1 "kademelere (isteğe bağlı seviye)" diyor. Açık soru §10-2 de "kademe yeterli mi, seviye gerekir mi?" sorusunu soruyor.
- **Seçenekler.**
  - (a) Yalnız kademe: dört değer var ve "Tüm okul" grubu dört satır taşır.
  - (b) Kademe zorunlu, seviye isteğe bağlı. Bu seçenekte öncelik kuralı gerekir (seviye eşleşmesi kademe eşleşmesini ezer), iki ayrı tekillik kısıtı gerekir ve "eşlenmemiş" hesabı iki katmanlı olur.
  - (c) Yalnız seviye: 14 değer var, UI kademe kısayolu sunar.
- **Karar: MVP'de (a).** Gerekçeler:
  1. Sahadaki bütün örnekler kademe ayrımı yapıyor. Bilişim'de üç kademe var, Akdeniz Boğaziçi tek liste yayınlıyor, Arel'de anaokulu ayrı. Seviye ayrımının tek gerekçesi porsiyon (§10-2). Porsiyon MVP'de modellenmiyor (Y-6 öğün kalorisini seçti).
  2. (b) ve (c) her öğrenci için öncelik çözümü, iki kısıt ve iki katmanlı bir seçim arayüzü getirir. Bunun karşılığı olan bir ihtiyaç kanıtı yok.
  3. (c)'de okul yeni bir seviye açtığında (ör. Hazırlık) o seviye sessizce eşlenmemiş kalır. (a)'da Hazırlık zaten `High`'a düşer (`EducationLevelClassifier`).
- **Genişleme yolu.** (b)'ye ek göçle geçilir. `meal_menu_group_levels` tablosuna `grade_level_ordinal int NULL` kolonu eklenir. Tekil indeks `(school_id, education_level) WHERE grade_level_ordinal IS NULL` ve `(school_id, grade_level_ordinal) WHERE grade_level_ordinal IS NOT NULL` olarak ikiye bölünür. Çözücüye "önce seviye, sonra kademe" kuralı eklenir. Var olan satırlar kademe genelini temsil etmeye devam eder, veri dönüşümü gerekmez.
- **Tenant.** Eşleme satırı `school_id` taşır. Bir kademenin okul içinde tek gruba gitmesi DB'de zorlanır (BR-YM-03).
- **Modül bağımlılığı.** Eşleme alanı Academics'in `EducationLevel` enum'udur. Domain'deki bu paylaşılan tip zaten Announcements tarafından da kullanılıyor.
- **Göç maliyeti.** Yok.
- **Geri alma maliyeti.** Düşük, yukarıda anlatılan ek göç yeterli.

### ADR-YM-4: Durum gün düzeyinde tutulur: `Draft` ve `Published`. Ayrı bir yayın tablosu yoktur

- **Bağlam.** BR-YM-08 "gün durumu" diyor. BR-YM-09'a göre yayın aralıktaki taslakları birlikte yayınlar ve yayında olup boş kalan gün "menü yok" görünür. BR-YM-18'e göre yeniden yayın ikinci bir "yayınlandı" bildirimi üretmez. Y-15'e göre geri çekme vardır ve bildirimsizdir.
- **Seçenekler.**
  - (a) Gün düzeyinde `Status` alanı ve damgalar.
  - (b) Yayın aralıkları tablosu (`MealMenuPublication`): bir gün, kapsayan bir yayın varsa görünür.
  - (c) İkisi birden.
- **Karar: (a), üç ekle birlikte.**
  1. `FirstPublishedAt`: BR-YM-18 kapısı. Bir yayın işlemi en az bir günü **ilk kez** yayınlıyorsa bildirim gider. Geri çekilip yeniden yayınlanan günler bildirim üretmez.
  2. `ChangedAfterPublishAt`: veliye "güncellendi" işaretini sağlar.
  3. **Boş günün somutlaştırılması.** Yayın, aralıktaki kaydı olmayan hafta içi ve tatil dışı günler için içi boş bir `Published` gün oluşturur. Böylece okuma tarafında kural tek cümleye iner: satır yoksa "henüz yayınlanmadı", yayında ve içi boşsa "Menü yok" (S-8). Yöneticinin isteğe bağlı gün notu ("Okul gezisi, kumanya verilecek") aynı satırda durur.

  (b) bir günün görünürlüğünü iki tablonun kesişimine bağlar. Yayın aralığının içinde sonradan girilen bir gün de kendiliğinden görünür olur. Bu, yayın sonrasında düzenleme yapanın taslak beklentisiyle çelişir.
- **Tenant.** Etkisi yok.
- **Modül bağımlılığı.** Yok.
- **Göç maliyeti.** Yok.
- **Geri alma maliyeti.** Düşük. Yayın tablosu gerekirse sonradan eklenir. Var olan `FirstPublishedAt` damgalarından türetilebilir.

### ADR-YM-5: Denetim izi modül içinde tutulur, yalnız eklenir, önce/sonra anlık görüntüsü taşır

- **Bağlam.** BR-YM-10 düzenlemenin kim, ne zaman, önce ve sonra bilgisiyle izlenmesini istiyor. ADR-YM-2 de geçmişin doğruluğunu bu ize bırakıyor.
- **Seçenekler.**
  - (a) `MealMenuDayAuditEntry`. `HomeworkAuditEntry` emsalidir: `Kind` alanı, sunucuda kurulan Türkçe `Action` metni, ayrıca `BeforeJson` ve `AfterJson`.
  - (b) Platform genelindeki bir audit log.
  - (c) Yalnız `UpdatedAt` ve `UpdatedBy`.
- **Karar: (a).** (c) önce/sonra bilgisini taşımaz. Ortak bir platform audit tablosu bu depoda modül düzeyinde bir emsale sahip değil. Duyuru, ödev ve not modüllerinin her biri kendi tablosunu tutuyor.
- **Tenant.** Satır `school_id` taşır. `PermanentTenantEntity` kullanılır: soft-delete alanı yoktur, satır güncellenmez.
- **Modül bağımlılığı.** Yok.
- **Göç maliyeti.** Bir tablo.
- **Geri alma maliyeti.** Düşük.

### ADR-YM-6: Modül kapısı `IModuleAvailability` soyutlamasıyla handler içinde uygulanır; istemciye `me/context.enabledModules` ile bildirilir

- **Bağlam.** Bulgu D-2: kapı yok. BR-YM-21'e göre modül kapalıyken menü yüzeyleri, rota ve bildirim görünmez, ama veri silinmez.
- **Seçenekler.**
  - (a) `Application/Common/Abstractions` altında `IModuleAvailability.IsEnabledAsync(schoolId, moduleKey)`. Uygulaması Schools modülündedir ve `PlanModuleResolver` ile aynı kuralı izler: tercih satırı varsa o geçerli, yoksa plan belirler. Her cafeteria handler'ı işe bu kapıyla başlar.
  - (b) Yeni bir MediatR pipeline davranışı, `[RequireModule]`.
  - (c) Yalnız istemcide gizleme.
- **Karar: (a).** (b) sabit pipeline sırasına dokunur (`mimari-kurallar.md`) ve bütün modülleri etkileyen kesişen bir değişikliktir. Diğer modüller de kapı istediğinde (b)'ye terfi edilebilir. (c) güvenlik değildir. İstemcinin gezinmeyi gizleyebilmesi için `ContextView`'a ekleme yoluyla `enabledModules: string[]` alanı konur. Bu bağlam zaten her rolde açılışta çağrılıyor, okul adı da aynı gerekçeyle oraya taşınmıştı (X-12).
- **Tenant.** Kapı okulun kendi tercih satırını okur. Global filtre bunu kapsar.
- **Modül bağımlılığı.** Cafeteria yalnız soyutlamayı tanır. Identity de `enabledModules` için aynı soyutlamayı kullanır.
- **Göç maliyeti.** Yok (göç §6.3'tedir ve ADR-YM-8'e aittir).
- **Geri alma maliyeti.** Düşük. Alan eklemedir, kaldırılırsa istemci modülü kapalı sayar.

### ADR-YM-7: Üç izin: `cafeteria.read`, `cafeteria.view-all`, `cafeteria.manage`

- **Bağlam.** Y-11'e göre öğretmen tüm grupları görür, veli ve öğrenci yalnız kendi grubunu görür. İstemciden gelen rol parametresine bakılmaz. Bu, ödev ve not modüllerinde kayda geçmiş bir ilkedir.
- **Karar.** Kapsam izinle çözülür (`timetable.view-all` emsali):
  - `read`: kendi menüm. Öğretmen, veli ve öğrenci alır, katalog üzerinden idare de alır.
  - `view-all`: bütün grupların yayındaki menüsü. Öğretmen alır, katalog üzerinden idare de alır.
  - `manage`: bütün yazma işlemleri ve taslağı görme. Yalnız SchoolAdmin alır, VicePrincipal aynalama ile alır. `AllPermissionIds` kataloğuna **girmez**, yani SuperAdmin almaz. Bu, `exams.manage` ve `clubs.manage` kalıbıdır: platform hesabı okul adına menü yayınlayamaz.
- **Geri alma maliyeti.** Düşük, seed ve göç.

### ADR-YM-8: Varsayılan durum KAPALI (Y-14); "Tüm okul" grubu ve varsayılan öğünler modül açılınca tohumlanır

- **Seçenekler.** Tohumlama için üç yol düşünüldü:
  - (a) Okul oluşturulunca (`SchoolCreated`).
  - (b) Modül açılınca (`ModuleConfigChangedEvent`), idempotent.
  - (c) İlk yönetim ekranı açılışında tembel olarak.
- **Karar: (b).** (a) modülü hiç kullanmayan okullarda veri üretir ve var olan okullar için ayrıca geri doldurma göçü ister. (c) bir GET'in yazma yapması demektir. (b) var olan ve yeni okulları aynı yoldan kapsar. Kapatıp yeniden açma veriye dokunmaz: grup ya da öğün varsa tohumlama atlanır.
- **Göç.** `plan_modules` tablosuna `cafeteria` eklenir ve var olan tüm okullar için tercih satırı `is_enabled = 0` yapılır. Ayrıntı §6.3'te.
- **Geri alma maliyeti.** Düşük, iki SQL bloğu.

### ADR-YM-9: "Menü değişti" bildirimleri Redis cooldown ile birleştirilir (pencerenin başı kazanır)

- **Seçenekler.**
  - (a) İlk değişiklikte hemen gönder, aynı grup ve tarih için 30 dakika boyunca sonraki değişiklikleri bastır. Mevcut `notification-rules.md` §7 desenidir.
  - (b) İlk değişiklikte Hangfire ile 30 dakika sonraya iş planla (pencerenin sonu kazanır).
  - (c) 30 dakikalık kovaya bağlı deterministik `eventId`.
- **Karar: (a).**
  - (b) bugünkü öğle yemeğinin değişikliğini 30 dakika geciktirir. "Bugün" eşiğinin var olma nedeni hızdır.
  - (c) kova sınırında bölünür: 10:29 ve 10:31'deki iki düzenleme iki bildirim üretir.
  - (a)'nın sınırı bilinçli olarak kabul edildi. Pencere içindeki ikinci düzenleme için ayrı bildirim gitmez. Bildirim gövdesi içerik taşımadığı ve derin bağlantı her zaman güncel veriyi açtığı için kayıp yalnız şundan ibarettir: bildirimi pencere içinde zaten açmış olan veli ikinci değişikliği görmez.

---

## 2. Aggregate sınırları, alanlar ve DDL

### 2.1 Aggregate haritası

| Kök (aggregate root) | Çocuklar (aynı işlemde yaşar, fiziksel silinir) | Taban sınıf | Not |
|---|---|---|---|
| `MealMenuGroup` | `MealMenuGroupLevel` (kademe eşlemesi), `MealMenuGroupPeriod` (grubun sunduğu öğünler) | Kök `TenantEntity` (soft delete), çocuklar `PermanentTenantEntity` | Kademenin okul içinde tekil olması kökler arası bir kısıttır. Son savunma DB'deki tekil indekstir, handler önceden 409 döner. |
| `MealPeriod` | — | `TenantEntity` | Kullanılmış öğün silinmez, pasifleştirilir (BR-YM-05). |
| `Dish` | — | `TenantEntity` | Alerjenler bit maskesi olarak tutulur (`int`). |
| `MealMenuDay` | `MealMenuEntry` (gün × öğün), `MealMenuEntryDish` (sıralı yemek satırı) | Kök `TenantEntity`, çocuklar `PermanentTenantEntity` | Yayın ve geri çekme durumu kökte tutulur. Çocuklar upsert sırasında bütünüyle değiştirilir. |
| `MealMenuDayAuditEntry` | — | `PermanentTenantEntity` | Yalnız eklenir. |

> **Çocuklar neden `PermanentTenantEntity`:** `database-rules.md` §3 soft delete'i kökler için zorunlu tutar, aggregate'in kendi çocuklarını istisna sayar. Soft-delete arayüzü (`ISoftDeletable`) olmayan çocuğu koleksiyondan çıkarmak gerçek bir DELETE üretir (`AnnouncementTemplate` emsali). Geçmiş içerik silinmiş satırlarda değil denetim anlık görüntüsünde yaşar (ADR-YM-5). Grup soft-delete edilirken handler önce çocuk koleksiyonlarını boşaltır, yoksa kademe tekil indeksi o kademeyi bırakmaz.

### 2.2 Enum'lar (Domain › `Modules/Cafeteria/Enums`)

| Enum | Değerler (tel değeri = ad, PascalCase) | Saklama |
|---|---|---|
| `MealMenuDayStatus` | `Draft = 1`, `Published = 2` | `tinyint` |
| `DishCategory` | `Soup = 1`, `MainCourse = 2`, `SideDish = 3`, `DessertOrFruit = 4`, `Beverage = 5` | `tinyint NULL` (kategorisiz yemek olabilir) |
| `Allergen` (`[Flags]`) | 14 değer, §4.1'de | `int` bit maskesi |
| `MealMenuAuditKind` | `Created = 1`, `Updated = 2`, `NoteChanged = 3`, `Published = 4`, `Withdrawn = 5`, `Copied = 6`, `Cleared = 7` | `tinyint` |
| `EducationLevel` | mevcut enum: `Preschool = 0`, `Primary = 1`, `Middle = 2`, `High = 3` | `tinyint` |

> **Tel kuralı:** `JsonStringEnumConverter` bir `[Flags]` değerini `"Gluten, Milk"` biçiminde tek string'e çevirir. DTO alerjenleri **`string[]`** olarak taşır, örneğin `["Gluten","Milk"]`. Eşleme handler projeksiyonunda yapılır (`AllergenSet.ToNames/FromNames`).

### 2.3 DDL taslağı (SQL Server, `school` şeması)

> `ToSchoolTable("…")` kullanılır. Ortak denetim kolonları (`created_at`, `created_by`, `updated_at`, `updated_by`, `row_version`) her tabloda vardır. Soft-delete kolonları (`is_deleted`, `deleted_at`, `deleted_by`) yalnız köklerde bulunur. Aşağıda kısalık için `/* audit */` ve `/* soft */` olarak yazıldılar.

```sql
-- ── Öğün (okul düzeyi) ──────────────────────────────────────────────
CREATE TABLE [school].[meal_periods] (
  id            uniqueidentifier NOT NULL CONSTRAINT pk_meal_periods PRIMARY KEY,
  school_id     uniqueidentifier NOT NULL,
  name          nvarchar(40)     NOT NULL,
  sort_order    int              NOT NULL,
  is_active     bit              NOT NULL CONSTRAINT df_meal_periods_is_active DEFAULT 1,
  /* audit */ /* soft */
);
CREATE UNIQUE INDEX ux_meal_periods_school_id_name ON [school].[meal_periods](school_id, name) WHERE is_deleted = 0;
CREATE INDEX ix_meal_periods_school_id_sort_order ON [school].[meal_periods](school_id, sort_order) WHERE is_deleted = 0;

-- ── Menü grubu ───────────────────────────────────────────────────────
CREATE TABLE [school].[meal_menu_groups] (
  id            uniqueidentifier NOT NULL CONSTRAINT pk_meal_menu_groups PRIMARY KEY,
  school_id     uniqueidentifier NOT NULL,
  name          nvarchar(60)     NOT NULL,
  sort_order    int              NOT NULL,
  /* audit */ /* soft */
);
CREATE UNIQUE INDEX ux_meal_menu_groups_school_id_name ON [school].[meal_menu_groups](school_id, name) WHERE is_deleted = 0;

CREATE TABLE [school].[meal_menu_group_levels] (           -- çocuk, fiziksel silinir
  id                 uniqueidentifier NOT NULL CONSTRAINT pk_meal_menu_group_levels PRIMARY KEY,
  school_id          uniqueidentifier NOT NULL,
  meal_menu_group_id uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_group_levels_group REFERENCES [school].[meal_menu_groups](id) ON DELETE CASCADE,
  education_level    tinyint          NOT NULL CONSTRAINT ck_meal_menu_group_levels_level CHECK (education_level BETWEEN 0 AND 3),
  /* audit */
);
-- BR-YM-03: bir kademe okul içinde EN ÇOK bir gruba
CREATE UNIQUE INDEX ux_meal_menu_group_levels_school_id_education_level
  ON [school].[meal_menu_group_levels](school_id, education_level);

CREATE TABLE [school].[meal_menu_group_periods] (          -- çocuk, fiziksel silinir
  id                 uniqueidentifier NOT NULL CONSTRAINT pk_meal_menu_group_periods PRIMARY KEY,
  school_id          uniqueidentifier NOT NULL,
  meal_menu_group_id uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_group_periods_group REFERENCES [school].[meal_menu_groups](id) ON DELETE CASCADE,
  meal_period_id     uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_group_periods_period REFERENCES [school].[meal_periods](id),
  /* audit */
);
CREATE UNIQUE INDEX ux_meal_menu_group_periods_group_period
  ON [school].[meal_menu_group_periods](meal_menu_group_id, meal_period_id);
CREATE INDEX ix_meal_menu_group_periods_school_id_period
  ON [school].[meal_menu_group_periods](school_id, meal_period_id);

-- ── Yemek kataloğu ──────────────────────────────────────────────────
CREATE TABLE [school].[dishes] (
  id              uniqueidentifier NOT NULL CONSTRAINT pk_dishes PRIMARY KEY,
  school_id       uniqueidentifier NOT NULL,
  name            nvarchar(120)    NOT NULL,   -- görünen ad (kırpılmış)
  normalized_name nvarchar(120)    NOT NULL,   -- tekillik + arama: Trim, iç boşluk tekleştirme, ToUpper(tr-TR)
  category        tinyint          NULL CONSTRAINT ck_dishes_category CHECK (category BETWEEN 1 AND 5),
  allergens       int              NOT NULL CONSTRAINT df_dishes_allergens DEFAULT 0
                  CONSTRAINT ck_dishes_allergens CHECK (allergens BETWEEN 0 AND 16383),  -- 14 bit
  is_active       bit              NOT NULL CONSTRAINT df_dishes_is_active DEFAULT 1,
  /* audit */ /* soft */
);
CREATE UNIQUE INDEX ux_dishes_school_id_normalized_name ON [school].[dishes](school_id, normalized_name) WHERE is_deleted = 0;
CREATE INDEX ix_dishes_school_id_is_active ON [school].[dishes](school_id, is_active) INCLUDE (normalized_name, name, category, allergens) WHERE is_deleted = 0;

-- ── Menü günü (kök) ─────────────────────────────────────────────────
CREATE TABLE [school].[meal_menu_days] (
  id                       uniqueidentifier NOT NULL CONSTRAINT pk_meal_menu_days PRIMARY KEY,
  school_id                uniqueidentifier NOT NULL,
  meal_menu_group_id       uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_days_group REFERENCES [school].[meal_menu_groups](id),
  menu_date                date             NOT NULL,
  status                   tinyint          NOT NULL CONSTRAINT ck_meal_menu_days_status CHECK (status IN (1,2)),
  note                     nvarchar(200)    NULL,
  first_published_at       datetimeoffset   NULL,  -- BR-YM-18 kapısı
  published_at             datetimeoffset   NULL,
  published_by             uniqueidentifier NULL,  -- Person.Id
  changed_after_publish_at datetimeoffset   NULL,  -- "güncellendi" işareti
  /* audit */ /* soft */
);
CREATE UNIQUE INDEX ux_meal_menu_days_school_id_group_date
  ON [school].[meal_menu_days](school_id, meal_menu_group_id, menu_date) WHERE is_deleted = 0;
CREATE INDEX ix_meal_menu_days_school_id_date_status
  ON [school].[meal_menu_days](school_id, menu_date, status) INCLUDE (meal_menu_group_id) WHERE is_deleted = 0;

CREATE TABLE [school].[meal_menu_entries] (                -- gün × öğün, fiziksel silinir
  id                uniqueidentifier NOT NULL CONSTRAINT pk_meal_menu_entries PRIMARY KEY,
  school_id         uniqueidentifier NOT NULL,
  meal_menu_day_id  uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_entries_day REFERENCES [school].[meal_menu_days](id) ON DELETE CASCADE,
  meal_period_id    uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_entries_period REFERENCES [school].[meal_periods](id),
  note              nvarchar(200)    NULL,
  calories          smallint         NULL CONSTRAINT ck_meal_menu_entries_calories CHECK (calories BETWEEN 0 AND 5000),
  /* audit */
);
CREATE UNIQUE INDEX ux_meal_menu_entries_day_period ON [school].[meal_menu_entries](meal_menu_day_id, meal_period_id);
CREATE INDEX ix_meal_menu_entries_school_id_period ON [school].[meal_menu_entries](school_id, meal_period_id);  -- "öğün kullanımda mı"

CREATE TABLE [school].[meal_menu_entry_dishes] (           -- sıralı yemek satırı, fiziksel silinir
  id                 uniqueidentifier NOT NULL CONSTRAINT pk_meal_menu_entry_dishes PRIMARY KEY,
  school_id          uniqueidentifier NOT NULL,
  meal_menu_entry_id uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_entry_dishes_entry REFERENCES [school].[meal_menu_entries](id) ON DELETE CASCADE,
  dish_id            uniqueidentifier NOT NULL
      CONSTRAINT fk_meal_menu_entry_dishes_dish REFERENCES [school].[dishes](id),
  sort_order         tinyint          NOT NULL,
  /* audit */
);
CREATE UNIQUE INDEX ux_meal_menu_entry_dishes_entry_sort ON [school].[meal_menu_entry_dishes](meal_menu_entry_id, sort_order);
CREATE UNIQUE INDEX ux_meal_menu_entry_dishes_entry_dish ON [school].[meal_menu_entry_dishes](meal_menu_entry_id, dish_id);
CREATE INDEX ix_meal_menu_entry_dishes_school_id_dish ON [school].[meal_menu_entry_dishes](school_id, dish_id);  -- "yemek kullanımda mı"

-- ── Denetim izi (yalnız eklenir) ────────────────────────────────────
CREATE TABLE [school].[meal_menu_day_audit_entries] (
  id                 uniqueidentifier NOT NULL CONSTRAINT pk_meal_menu_day_audit_entries PRIMARY KEY,
  school_id          uniqueidentifier NOT NULL,
  meal_menu_day_id   uniqueidentifier NOT NULL,   -- FK YOK: kök soft-delete edilse de iz kalır
  meal_menu_group_id uniqueidentifier NOT NULL,
  menu_date          date             NOT NULL,
  kind               tinyint          NOT NULL,
  actor_person_id    uniqueidentifier NOT NULL,
  action             nvarchar(300)    NOT NULL,   -- sunucuda kurulan Türkçe cümle
  before_json        nvarchar(max)    NULL,       -- MealMenuDaySnapshot (§2.5)
  after_json         nvarchar(max)    NULL,
  occurred_at        datetimeoffset   NOT NULL,
  /* audit */
);
CREATE INDEX ix_meal_menu_day_audit_entries_school_id_day_occurred
  ON [school].[meal_menu_day_audit_entries](school_id, meal_menu_day_id, occurred_at);
```

**Kısıt özetleri ve doğrulama sınırları.** Hepsi Domain'de sabittir ve EF kolon uzunluklarıyla aynıdır.

| Alan | Sınır |
|---|---|
| Grup adı | 2–60 karakter, okul içinde tekil |
| Öğün adı | 2–40 karakter, okul içinde tekil |
| Yemek adı | 2–120 karakter. Tekillik ölçütü `normalized_name`. |
| Gün notu, öğün notu | 0–200 karakter |
| Kalori | 0–5000 tam sayı, isteğe bağlı |
| Öğün başına yemek | 1–12. Aynı yemek aynı öğünde iki kez yer alamaz. |
| Grup | En az 1 öğün sunar |
| Okul | En az 1 grup (BR-YM-02) ve en az 1 aktif öğün |
| Tarih aralığı (sorgu, yayın, geri çekme) | En çok 62 gün |
| Kopyalama | Kaynak aralığı en çok 35 gün. Hedef tarih sayısı en çok 31. |

### 2.4 EF Core konfigürasyon taslağı (örnek, uygulama kodu değil)

```csharp
public sealed class MealMenuDayConfiguration : IEntityTypeConfiguration<MealMenuDay>
{
    public void Configure(EntityTypeBuilder<MealMenuDay> b)
    {
        b.ToSchoolTable("meal_menu_days");
        b.HasKey(x => x.Id);
        b.Property(x => x.Id).ValueGeneratedNever();
        b.Property(x => x.SchoolId).IsRequired();
        b.Property(x => x.MealMenuGroupId).IsRequired();
        b.Property(x => x.MenuDate).IsRequired();                       // DateOnly → date
        b.Property(x => x.Status).HasConversion<byte>().IsRequired();
        b.Property(x => x.Note).HasMaxLength(MealMenuDay.MaxNoteLength); // 200
        b.Property(x => x.RowVersion).IsRowVersion();

        b.HasMany(x => x.Entries).WithOne()
            .HasForeignKey(e => e.MealMenuDayId).OnDelete(DeleteBehavior.Cascade);
        b.Navigation(x => x.Entries).UsePropertyAccessMode(PropertyAccessMode.Field);

        b.HasIndex(x => new { x.SchoolId, x.MealMenuGroupId, x.MenuDate })
            .IsUnique().HasFilter("is_deleted = 0")
            .HasDatabaseName("ux_meal_menu_days_school_id_group_date");
        b.HasIndex(x => new { x.SchoolId, x.MenuDate, x.Status })
            .IncludeProperties(x => x.MealMenuGroupId).HasFilter("is_deleted = 0")
            .HasDatabaseName("ix_meal_menu_days_school_id_date_status");
    }
}

public sealed class DishConfiguration : IEntityTypeConfiguration<Dish>
{
    public void Configure(EntityTypeBuilder<Dish> b)
    {
        b.ToSchoolTable("dishes");
        b.HasKey(x => x.Id);
        b.Property(x => x.Name).IsRequired().HasMaxLength(Dish.MaxNameLength);
        b.Property(x => x.NormalizedName).IsRequired().HasMaxLength(Dish.MaxNameLength);
        b.Property(x => x.Category).HasConversion<byte?>();
        b.Property(x => x.Allergens).HasConversion<int>().IsRequired().HasDefaultValue(Allergen.None);
        b.ToTable(t => t.HasCheckConstraint("ck_dishes_allergens", "allergens BETWEEN 0 AND 16383"));
        b.Property(x => x.RowVersion).IsRowVersion();
        b.HasIndex(x => new { x.SchoolId, x.NormalizedName }).IsUnique().HasFilter("is_deleted = 0")
            .HasDatabaseName("ux_dishes_school_id_normalized_name");
    }
}
```

Domain yüzeyi (imza düzeyinde öneri):

```csharp
public sealed class MealMenuDay : TenantEntity
{
    public static MealMenuDay CreateDraft(Guid schoolId, Guid groupId, DateOnly date);
    public static MealMenuDay CreatePublishedEmpty(Guid schoolId, Guid groupId, DateOnly date, Guid by, DateTimeOffset now); // ADR-YM-4
    public ContentChange ReplaceContent(string? note, IReadOnlyList<EntryInput> entries, DateOnly schoolToday, DateTimeOffset now);
        // geçmiş tarih → CafeteriaDomainException("Cafeteria.Day.PastDateLocked")
        // Published ise ve içerik değiştiyse: ChangedAfterPublishAt = now, MealMenuDayChangedEvent
    public void SetNote(string? note, DateTimeOffset now);   // geçmiş tarihte DE serbest (BR-YM-11: "yalnız not")
    public bool Publish(Guid by, DateTimeOffset now);        // true = ilk yayın (FirstPublishedAt null idi)
    public void Withdraw(DateOnly schoolToday);              // Published → Draft, olay YAYMAZ (Y-15 bildirimsiz)
    public bool IsEmpty { get; }                             // Entries.Count == 0
}
```

### 2.5 Denetim anlık görüntüsü (`before_json` / `after_json`)

```json
{
  "status": "Published",
  "note": null,
  "entries": [
    {
      "mealPeriodId": "5b0c…",
      "mealPeriodName": "Öğle Yemeği",
      "calories": 650,
      "note": null,
      "dishes": [
        { "dishId": "a1…", "name": "Mercimek çorbası", "allergens": ["Gluten", "Celery"] },
        { "dishId": "a2…", "name": "Fırın tavuk", "allergens": [] }
      ]
    }
  ]
}
```

Anlık görüntü yazma anındaki ad ve alerjenleri **dondurur**. ADR-YM-2'nin "o gün ne söylendi" güvencesi budur.

---

## 3. REST API sözleşmesi

**Ortak kurallar** (`api-sozlesmesi.md` ile aynı):

- Temel yol `/api/v1/meal-menus`.
- Yanıtlar `ApiResponse<T>` zarfında döner, alan adları camelCase.
- Enum'lar PascalCase string olarak gider.
- Tarihler `yyyy-MM-dd` biçimindedir ve okul yerel günüdür. Anlar ISO 8601 `datetimeoffset` olarak gider.
- Yaşam döngüsü fiilleri iki nokta ile yazılır (`:publish`). Generic PATCH kullanılmaz.

**Modül kapısı.** Her uç önce `IModuleAvailability` ile kontrol eder. Modül kapalıysa **403 `Cafeteria.ModuleDisabled`** döner, mesajı "Yemekhane modülü bu okulda kapalı." olur.

**Tenant.** Bütün sorgular global filtreye tabidir. Ek olarak her sorgu `SchoolId == tenant.CurrentSchoolId` yüklemini açıkça taşır: süper yönetici okulu üstlendiğinde global süzgeç düşer, bu yüzden yüklem şarttır (R45 emsali). Başka okulun ya da hiç olmayan kaydın kimliği **404 `Error.NotFound`** döner.

### 3.1 Uç listesi (22 yeni uç ve 1 değişen uç)

| # | Yöntem ve yol | İzin | Amaç |
|---|---|---|---|
| 1 | `GET /meal-menus/groups` | `cafeteria.view-all` | Gruplar, kademe eşlemesi, eşlenmemiş kademeler |
| 2 | `POST /meal-menus/groups` | `cafeteria.manage` | Grup oluştur |
| 3 | `PUT /meal-menus/groups/{groupId}` | `cafeteria.manage` | Ad, sıra, kademeler, öğünler |
| 4 | `DELETE /meal-menus/groups/{groupId}` | `cafeteria.manage` | Soft delete |
| 5 | `GET /meal-menus/meal-periods` | `cafeteria.view-all` | Öğünler (`includeInactive` yalnız manage) |
| 6 | `POST /meal-menus/meal-periods` | `cafeteria.manage` | Öğün ekle |
| 7 | `PUT /meal-menus/meal-periods/{mealPeriodId}` | `cafeteria.manage` | Ad, sıra, aktiflik |
| 8 | `DELETE /meal-menus/meal-periods/{mealPeriodId}` | `cafeteria.manage` | Yalnız hiç kullanılmamışsa |
| 9 | `GET /meal-menus/dishes` | `cafeteria.manage` | Katalog ve otomatik tamamlama (sayfalı) |
| 10 | `POST /meal-menus/dishes` | `cafeteria.manage` | Yemek ekle |
| 11 | `PUT /meal-menus/dishes/{dishId}` | `cafeteria.manage` | Ad, kategori, alerjen, aktiflik |
| 12 | `DELETE /meal-menus/dishes/{dishId}` | `cafeteria.manage` | Yalnız hiç kullanılmamışsa |
| 13 | `GET /meal-menus/groups/{groupId}/days?from&to&includeDrafts` | `cafeteria.view-all` (taslak için ayrıca `manage`) | Aralık sorgusu: öğretmen görünümü, yönetim ızgarası, **yazdırma verisi** |
| 14 | `PUT /meal-menus/groups/{groupId}/days/{date}` | `cafeteria.manage` | Gün upsert |
| 15 | `PUT /meal-menus/groups/{groupId}/days/{date}/note` | `cafeteria.manage` | Gün notu (geçmiş tarihte de açık) |
| 16 | `GET /meal-menus/groups/{groupId}/days/{date}/audit-trail` | `cafeteria.manage` | Denetim izi |
| 17 | `POST /meal-menus/groups/{groupId}/days:copy-range` | `cafeteria.manage` | Hafta veya ay kopyala |
| 18 | `POST /meal-menus/groups/{groupId}/days/{date}:copy-to` | `cafeteria.manage` | Günü şu günlere kopyala |
| 19 | `POST /meal-menus/groups/{groupId}:publish` | `cafeteria.manage` | Aralığı yayınla |
| 20 | `POST /meal-menus/groups/{groupId}:withdraw` | `cafeteria.manage` | Aralığı geri çek (bildirimsiz) |
| 21 | `GET /meal-menus/me?from&to&studentPersonId` | `cafeteria.read` | Benim menüm: öğrenci kendisi, veli çocuk başına |
| 22 | `GET /meal-menus/me/today?studentPersonId` | `cafeteria.read` | Anasayfa "Bugünün menüsü" kartı |
| D | `GET /auth/me/context` (**değişir, ekleme**) | — | `enabledModules: string[]` alanı (ADR-YM-6) |

> **Tatil bilgisi nasıl döner:** Ayrı bir uç yoktur. Aralık dönen her uç (13, 21) `holidays[]` taşır. Bugün kartı (22) tatili atlar. Kaynak `IHolidayCalendarReader.GetHolidaysInRangeAsync`'tir: okul tatili `source: "TENANT"`, resmî tatil `source: "OFFICIAL"`. Gün upsert yanıtı da o tarihe düşen tatili taşır. Bu bilgi engellemez (BR-YM-13), istemci yalnız uyarı gösterir.

### 3.2 Ortak DTO'lar

```ts
// Grup
type MealMenuGroupDto = {
  id: string; name: string; sortOrder: number;
  educationLevels: EducationLevel[];          // "Preschool" | "Primary" | "Middle" | "High"
  mealPeriodIds: string[];                     // grubun sunduğu öğünler, sortOrder sırasıyla
};

// Öğün
type MealPeriodDto = { id: string; name: string; sortOrder: number; isActive: boolean; isInUse: boolean };

// Yemek
type DishDto = {
  id: string; name: string;
  category: DishCategory | null;               // "Soup" | "MainCourse" | "SideDish" | "DessertOrFruit" | "Beverage"
  allergens: Allergen[];                        // §4.1 kodları, sabit sırada
  isActive: boolean; isInUse: boolean;
};

// Gün (görüntüleme + yönetim)
type MealMenuDayDto = {
  date: string;                                 // "2026-11-02"
  status: "Draft" | "Published";               // aile ve öğretmen yalnız "Published" görür
  note: string | null;                          // gün notu ("Okul gezisi — kumanya")
  isEmpty: boolean;                             // yayında ama öğün yok → "Menü yok"
  isUpdated: boolean;                           // yayından sonra değişti
  updatedAt: string | null;                     // ChangedAfterPublishAt
  rowVersion: string | null;                    // yalnız cafeteria.manage taşıyana dolu (base64)
  entries: MealMenuEntryDto[];                  // öğün sortOrder sırasıyla
};
type MealMenuEntryDto = {
  mealPeriodId: string; mealPeriodName: string; sortOrder: number;
  note: string | null; calories: number | null;
  dishes: MealMenuDishDto[];                    // giriş sırasıyla
};
type MealMenuDishDto = { dishId: string; name: string; category: DishCategory | null; allergens: Allergen[] };

type HolidayDto = { name: string; startDate: string; endDate: string; source: "TENANT" | "OFFICIAL" };

// Aralık
type MealMenuRangeDto = {
  group: { id: string; name: string };
  from: string; to: string;
  mealPeriods: { id: string; name: string; sortOrder: number }[]; // grubun sunduğu AKTİF öğünler
  days: MealMenuDayDto[];                       // yalnız VAR OLAN ve çağırana görünen günler; tarih sıralı
  holidays: HolidayDto[];
};
```

**Görünürlük kuralı (sunucuda uygulanır).** `days[]` içinde bir tarih yoksa "henüz yayınlanmadı" anlamına gelir. `isEmpty: true` ise "Menü yok" demektir. Tarih bir tatile düşüyorsa istemci "Tatil" gösterir. Tatil önceliklidir.

### 3.3 Yönetim uçları: istek, yanıt ve hata

**1 · `GET /meal-menus/groups`** → `200`

```ts
{ groups: MealMenuGroupDto[];
  schoolEducationLevels: EducationLevel[];        // okulun aktif seviyelerinin kademeleri (SchoolGradeLevels → GradeLevel)
  unassignedEducationLevels: EducationLevel[] }   // schoolEducationLevels − atanmış (BR-YM-03 uyarısı)
```

**2 · `POST /meal-menus/groups`** → `201 MealMenuGroupDto`

```ts
{ name: string; educationLevels: EducationLevel[]; mealPeriodIds: string[]; moveLevels?: boolean }
```

- `moveLevels: true` ise başka grupta olan kademe bu gruba **taşınır**. Örnek: "Anaokulu"na ayrı menü açılırken Preschool "Tüm okul" grubundan alınır. `false` ise 409 döner.
- Hatalar:
  - 400 `Validation`.
  - 400 `Cafeteria.Group.NoMealPeriods`.
  - 409 `Cafeteria.Group.NameTaken`.
  - 409 `Cafeteria.Group.LevelAssignedElsewhere` ("Ortaokul başka bir grupta. Taşımak için onaylayın.").
  - 409 `Cafeteria.MealPeriod.Inactive`.

**3 · `PUT /meal-menus/groups/{groupId}`** → `200 MealMenuGroupDto`

- Gövde 2'ninkiyle aynıdır, ek olarak `sortOrder: number` alır. Hatalar da aynıdır, ek olarak 404 döner.
- Gruptan bir öğün çıkarılırsa geçmiş ve gelecek günlerdeki o öğün kayıtları **silinmez**, yalnız görüntüden düşer. `mealPeriods` grubun **güncel** öğünleridir. Bunu istemci "Bu öğündeki kayıtlı günler görünmez olur" diye uyarır.

**4 · `DELETE /meal-menus/groups/{groupId}`** → `204`

- 409 `Cafeteria.Group.LastGroup` (BR-YM-02).
- 409 `Cafeteria.Group.HasUpcomingPublishedDays` (bugün ve sonrasında yayında gün var, önce geri çekilmeli).
- Silme kademe eşlemelerini boşaltır.

**5 · `GET /meal-menus/meal-periods?includeInactive=false`** → `200 MealPeriodDto[]`

**6 · `POST /meal-menus/meal-periods`** `{ name, sortOrder }` → `201 MealPeriodDto`

- 409 `Cafeteria.MealPeriod.NameTaken`.

**7 · `PUT /meal-menus/meal-periods/{id}`** `{ name, sortOrder, isActive }` → `200`

- 409 `Cafeteria.MealPeriod.NameTaken`.
- 409 `Cafeteria.MealPeriod.LastActive`.
- Pasif öğün yeni girişte ve aile görünümünde yer almaz. Geçmiş kayıtlar denetim izinde kalır.

**8 · `DELETE /meal-menus/meal-periods/{id}`** → `204`

- 409 `Cafeteria.MealPeriod.InUse` ("Bu öğün geçmiş menülerde kullanıldı. Silmek yerine pasifleştirin."), BR-YM-05.
- 409 `Cafeteria.MealPeriod.LastActive`.

**9 · `GET /meal-menus/dishes?search&category&includeInactive&page&pageSize`** → `200 PagedResult<DishDto>`

- `search` en az 2 karakterdir ve `normalized_name LIKE '%…%'` olarak uygulanır. Sıralama: önce adın başında eşleşenler, sonra ad.
- Otomatik tamamlama `pageSize=10&includeInactive=false` ile çağrılır.

**10 · `POST /meal-menus/dishes`** `{ name, category?: DishCategory|null, allergens: Allergen[] }` → `201 DishDto`

- 409 `Cafeteria.Dish.NameTaken`. Tekillik `normalized_name` üzerindendir, yani "mercimek  çorbası" ile "Mercimek Çorbası" aynı yemektir.

**11 · `PUT /meal-menus/dishes/{id}`** `{ name, category, allergens, isActive }` → `200 DishDto`

- Değişiklik **tüm günlere** yansır, geçmiş günler dâhil (BR-YM-06). İstemci kaydetmeden önce şunu göstersin: "Bu yemek N günde kullanılıyor; değişiklik hepsine yansır."
- Sayıyı DTO'daki `isInUse` alanı vermez. Bilinçli olarak yalın tutuldu, mesaj sayısız gösterilebilir.

**12 · `DELETE /meal-menus/dishes/{id}`** → `204`

- 409 `Cafeteria.Dish.InUse` ("Kullanılmış yemek silinemez; pasifleştirin.").

**13 · `GET /meal-menus/groups/{groupId}/days?from=2026-11-01&to=2026-11-30&includeDrafts=true`** → `200 MealMenuRangeDto`

- `includeDrafts=true` yalnız `cafeteria.manage` ile kullanılabilir. Diğerleri 403 `Cafeteria.Drafts.Forbidden` alır.
- `rowVersion` yalnız manage taşıyana doludur.
- Yazdırma görünümü (M-10) bu ucu `from`/`to` ayın ilk ve son günü olacak şekilde kullanır. Yayındaki veri basılır, ayrı bir uç yoktur.
- Hatalar: 400 `Cafeteria.Range.TooLong` (62 günü aşarsa), 400 `Validation` (`from > to`).

**14 · `PUT /meal-menus/groups/{groupId}/days/{date}`** → `200 MealMenuDayUpsertResultDto`

```ts
// istek
{
  rowVersion: string | null;           // var olan günü düzenlerken zorunlu; yeni gün için null
  note: string | null;
  entries: {
    mealPeriodId: string;
    note: string | null;
    calories: number | null;
    dishes: ({ dishId: string } | { name: string })[];   // name → katalogda (normalized) varsa bağlanır, yoksa OLUŞTURULUR (alerjensiz)
  }[];                                  // boş liste = gün boş
}
// yanıt
{ day: MealMenuDayDto | null;            // null = taslak gün tamamen boşaltıldı ve silindi ("henüz girilmedi"ye döndü)
  createdDishes: DishDto[];              // isimden yeni açılan yemekler; UI "alerjen ekle" önerir
  holiday: HolidayDto | null }           // bu tarihe düşen tatil (uyarı amaçlı)
```

Kurallar:

- Yeni gün `Draft` olarak doğar.
- `Published` gün düzenlenince yayında kalır (BR-YM-10). İçerik değişmişse `isUpdated` işaretlenir ve olay yayılır (§5).
- `Published` gün boşaltılırsa yayında "Menü yok" olarak kalır.
- `Draft` gün tamamen boşaltılırsa (ne öğün ne not) soft delete edilir.
- Her yazma bir denetim satırı üretir (`Created`, `Updated` ya da `Cleared`).

Hatalar:

| HTTP | Kod | Anlamı |
|---|---|---|
| 400 | `Validation` | Gövde biçim hatası |
| 400 | `Cafeteria.Day.MealPeriodNotOffered` | Öğün grupta yok |
| 400 | `Cafeteria.Day.TooManyDishes` | Öğünde yemek sınırı aşıldı |
| 400 | `Cafeteria.Day.DuplicateDish` | Aynı yemek aynı öğünde iki kez |
| 409 | `Cafeteria.Day.PastDateLocked` | "Geçmiş günün menüsü değiştirilemez; yalnız not eklenebilir." (BR-YM-11) |
| 409 | `Cafeteria.Day.Stale` | `rowVersion` bayat |
| 409 | `Cafeteria.Dish.Inactive` | Pasif `dishId` gönderildi |
| 404 | — | Grup yok |

**15 · `PUT …/days/{date}/note`** `{ note: string|null, rowVersion: string|null }` → `200 MealMenuDayDto`

- Geçmiş tarihte de çalışır. BR-YM-11'deki "yalnız not eklenebilir" kuralı bu uçtur.
- Gün yoksa `Draft` olarak açılır.
- `Published` günde not değişikliği "değişti" olayı sayılır (§5.2).

**16 · `GET …/days/{date}/audit-trail`** → `200`

```ts
{ id: string; kind: MealMenuAuditKind; action: string; actorName: string; occurredAt: string;
  before: MealMenuDaySnapshot | null; after: MealMenuDaySnapshot | null }[]
```

Sıralama yeniden eskiyedir.

**17 · `POST …/groups/{groupId}/days:copy-range`** → `200 CopyResultDto`

```ts
{ sourceFrom: string; sourceTo: string; targetFrom: string;
  targetGroupId?: string;          // yoksa aynı grup
  overwrite: boolean;              // false → dolu hedef gün ATLANIR
  includeHolidays: boolean }       // false (varsayılan) → tatile düşen hedef ATLANIR
```

- Kayma `targetFrom − sourceFrom` kadardır ve **7'nin katı olmalıdır**. Böylece hafta günleri hizalı kalır: "Geçen haftayı kopyala" 7 gün kaydırır, 4 haftalık döngü 28 gün kaydırır. Uymazsa 400 `Cafeteria.Copy.WeekdayMismatch`.
- Kaynak aralığı en çok 35 gündür, aksi 400 `Cafeteria.Range.TooLong`.

**18 · `POST …/days/{date}:copy-to`** `{ targetDates: string[]; targetGroupId?: string; overwrite: boolean; includeHolidays: boolean }` → `200 CopyResultDto`

```ts
type CopyResultDto = {
  copiedDates: string[];
  skippedNonEmptyDates: string[];     // overwrite=false iken dolu olanlar → UI "N gün dolu, üzerine yazılsın mı?" sorup overwrite=true ile tekrar yollar
  skippedPastDates: string[];         // hedef geçmişse her zaman atlanır (BR-YM-11)
  skippedHolidayDates: string[];
  skippedEntries: number;             // hedef grup o öğünü sunmuyorsa atlanan öğün kayıtları
};
```

Kopyalama kuralları:

- Kopyalanan şeyler: öğün kayıtları, yemekler, öğün notu, kalori ve gün notu.
- Hedefte yeni gün `Draft` olarak doğar.
- `overwrite` ile üzerine yazılan `Published` gün **yayında kalır**. Bu bir düzenlemedir: `isUpdated` işaretlenir, gün yakın tarihteyse §5.2 bildirimi gider.
- Denetim kaydı `Copied` türüyle yazılır.

**19 · `POST …/groups/{groupId}:publish`** `{ from: string; to: string }` → `200`

```ts
{ publishedDates: string[];          // Draft → Published
  emptyDatesMarked: string[];        // kaydı olmayan hafta içi, tatil dışı günler → "Menü yok" (ADR-YM-4)
  skippedPastDates: string[];
  notificationSent: boolean }        // en az bir gün İLK kez yayınlandıysa true (BR-YM-18)
```

- Geçmiş tarihler kırpılır ve değişmez.
- 409 `Cafeteria.Publish.NothingToPublish`: aralıkta hiç dolu taslak yoksa (BR-YM-09). Yalnız boş gün somutlaştırmak da yayın sayılmaz.
- 400 `Cafeteria.Range.TooLong`.

**20 · `POST …/groups/{groupId}:withdraw`** `{ from: string; to: string }` → `200`

```ts
{ withdrawnDates: string[]; skippedPastDates: string[] }
```

- Bugün ve sonrasındaki `Published` günleri `Draft`'a çevirir. Bildirim üretmez (Y-15). Denetim kaydı `Withdrawn` türüyle yazılır.
- 409 `Cafeteria.Withdraw.NothingToWithdraw`.

### 3.4 Görüntüleme uçları

**21 · `GET /meal-menus/me?from&to&studentPersonId`** → `200`

```ts
{
  status: "Ok" | "NoClassroom" | "NoGroup";
  student: { personId: string; fullName: string } | null;
  menu: MealMenuRangeDto | null;      // yalnız status=Ok; days[] YALNIZ Published
}
```

- **Öğrenci** `studentPersonId` göndermez ve kendi menüsünü alır.
- **Veli** `studentPersonId` göndermek zorundadır. Mobilde aktif çocuk, webde seçili çocuk kullanılır. Velinin o öğrenciyle iptal edilmemiş ve `CanViewInfo` taşıyan bir ilişkisi yoksa **403 `Cafeteria.Student.Forbidden`** döner. Bu kapı `GetMyExamScheduleQueryHandler` ile aynıdır.
- `NoClassroom` durumu: öğrenci aktif değil, şubesi yok ya da çağıran öğrenci değil (öğretmen bu ucu çağırmamalı).
- `NoGroup` durumu: öğrencinin kademesi hiçbir gruba eşli değil. UI "Bu sınıf için menü tanımlanmamış." gösterir (BR-YM-03).
- Grup çözümü: `StudentProfile.CurrentClassroomId → ClassRoom.GradeLevelId → GradeLevel.EducationLevel → meal_menu_group_levels`. Sonuç saklanmaz (BR-YM-04).

**22 · `GET /meal-menus/me/today?studentPersonId`** → `200`

```ts
{
  cards: {
    group: { id: string; name: string };
    students: { personId: string; fullName: string }[];   // veli: bu gruptaki çocukları; öğrenci/öğretmen/idare: []
    date: string;
    isToday: boolean;                                     // false → "Pazartesi'nin menüsü"
    day: MealMenuDayDto;                                  // Published
  }[]                                                      // boş dizi → kart GİZLENİR
}
```

Kartın tarih seçimi (BR-YM-14): okul yerel bugününden başlanır ve 7 gün ileriye bakılır. Seçilen ilk tarih d için şu koşullar aranır:

- d tatil değildir,
- d için `Published` bir gün vardır,
- d bugünse gün boş olabilir ("Bugün menü yok" ve not gösterilir), d bugün değilse gün boş olmamalıdır.

Hangi gruplara bakıldığı role göre değişir:

| Çağıran | Bakılan gruplar |
|---|---|
| Veli, `studentPersonId` olmadan | Bütün çocukları. Aynı gruptaki çocuklar tek kartta birleşir (BR-YM-15). |
| Veli, `studentPersonId` ile | Yalnız o çocuk |
| Öğrenci | Kendi grubu |
| `view-all` taşıyan (öğretmen, idare) | Bütün gruplar, `sortOrder` sırasıyla |

### 3.5 Hata kodları: tam liste ve HTTP eşlemesi

`ResultExtensions.MapStatusCode` içine bir **`Cafeteria.*` modül kovası** eklenmeli. `ErrorMessageCatalog`'a her kod için Türkçe cümle girilir. UI tarafında `DOMAIN_FORBIDDEN_CODE_PREFIXES` listesine `"Cafeteria."` eklenir.

| Kod | HTTP |
|---|---|
| `Cafeteria.ModuleDisabled` | 403 (açık eşleme) |
| `Cafeteria.Drafts.Forbidden`, `Cafeteria.Student.Forbidden` | 403 (X-17 aile kuralı: `…Forbidden`) |
| `Cafeteria.Group.NoMealPeriods`, `Cafeteria.Day.MealPeriodNotOffered`, `Cafeteria.Day.TooManyDishes`, `Cafeteria.Day.DuplicateDish`, `Cafeteria.Range.TooLong`, `Cafeteria.Copy.WeekdayMismatch` | 400 |
| `Cafeteria.Group.NameTaken`, `Cafeteria.Group.LevelAssignedElsewhere`, `Cafeteria.Group.LastGroup`, `Cafeteria.Group.HasUpcomingPublishedDays`, `Cafeteria.MealPeriod.NameTaken`, `Cafeteria.MealPeriod.InUse`, `Cafeteria.MealPeriod.LastActive`, `Cafeteria.MealPeriod.Inactive`, `Cafeteria.Dish.NameTaken`, `Cafeteria.Dish.InUse`, `Cafeteria.Dish.Inactive`, `Cafeteria.Day.PastDateLocked`, `Cafeteria.Day.Stale`, `Cafeteria.Publish.NothingToPublish`, `Cafeteria.Withdraw.NothingToWithdraw` | 409 |
| `Validation` | 400 (FluentValidation) |
| `Error.NotFound` | 404 |

---

## 4. Sabitler ve tohum verisi

### 4.1 Alerjenler (14, AB 1169/2011 Ek II ve TGK Etiketleme Yönetmeliği)

| Bit | Kod (`Allergen`) | TR ad (tam) | TR kısa etiket (çip) |
|---|---|---|---|
| 1 | `Gluten` | Gluten içeren tahıllar (buğday, arpa, çavdar, yulaf) | Gluten |
| 2 | `Crustaceans` | Kabuklular (karides, yengeç, ıstakoz) | Kabuklu deniz ürünü |
| 4 | `Eggs` | Yumurta ve yumurta ürünleri | Yumurta |
| 8 | `Fish` | Balık ve balık ürünleri | Balık |
| 16 | `Peanuts` | Yer fıstığı ve yer fıstığı ürünleri | Yer fıstığı |
| 32 | `Soybeans` | Soya fasulyesi ve soya ürünleri | Soya |
| 64 | `Milk` | Süt ve süt ürünleri (laktoz dâhil) | Süt |
| 128 | `TreeNuts` | Sert kabuklu meyveler (fındık, ceviz, badem, kaju, antep fıstığı vb.) | Sert kabuklu meyve |
| 256 | `Celery` | Kereviz ve kereviz ürünleri | Kereviz |
| 512 | `Mustard` | Hardal ve hardal ürünleri | Hardal |
| 1024 | `Sesame` | Susam tohumu ve susam ürünleri | Susam |
| 2048 | `Sulphites` | Kükürt dioksit ve sülfitler | Sülfit |
| 4096 | `Lupin` | Acı bakla (lupin) ve ürünleri | Acı bakla |
| 8192 | `Molluscs` | Yumuşakçalar (midye, kalamar, ahtapot) | Yumuşakça |

- Serbest alerjen girilemez (BR-YM-07).
- "Etiket yok" ifadesi "alerjen içermez" anlamına gelmez. İstemci etiketsiz yemekte hiçbir şey göstermez. Menü görünümünün altında sabit uyarı (BR-YM-17) durur. Metin core sabitidir, hukuki teyidi §10'daki notta.

### 4.2 Yemek kategorileri

| Kod (`DishCategory`) | TR |
|---|---|
| `Soup` | Çorba |
| `MainCourse` | Ana yemek |
| `SideDish` | Yardımcı yemek |
| `DessertOrFruit` | Tatlı / Meyve |
| `Beverage` | İçecek |
| `null` | Kategorisiz (kahvaltılıklar çoğu zaman böyle kalır) |

### 4.3 Tohum: ne zaman, ne

Tohumlama `cafeteria` modülünün okulda **açıldığı an** yapılır (ADR-YM-8). Handler `ModuleConfigChangedEvent` olayını dinler, `IModuleAvailability` modülün açık olduğunu söylerse çalışır.

Okulda hiç `MealPeriod` yoksa üç öğün tohumlanır:

| sortOrder | Ad |
|---|---|
| 1 | Kahvaltı |
| 2 | Öğle Yemeği |
| 3 | İkindi Kahvaltısı |

Okulda hiç `MealMenuGroup` yoksa "Tüm okul" grubu tohumlanır:

- `sortOrder = 1`,
- kademeler: `Preschool`, `Primary`, `Middle`, `High` (dördü birden). Okul sonradan yeni bir kademe açarsa o kademe zaten eşli olur.
- öğünler: tohumlanan üçü. Gerekçe: tohum kimseyi kısıtlamamalı. Boş öğün veliye görünmez. İdare gereksiz öğünü gruptan kaldırır.

Davranış özellikleri:

- **İdempotent:** grup ya da öğün varsa hiçbiri yeniden yazılmaz. Kapatıp açmak veriye dokunmaz.
- **Tenant:** handler `IgnoreQueryFilters` kullanmaz ve istek bağlamında çalışır. Yine de varlık kontrolü `SchoolId == evt.SchoolId` yüklemini açıkça taşır.

---

## 5. Bildirim

### 5.1 Yeni türler (`NotificationKind`, enum'un sonuna)

| Değer | Kind | Grup | Severity | Bekleyen iş | Push anahtarı | Kitle (`PushAudience`) |
|---|---|---|---|---|---|---|
| 60 | `MealMenuPublished` | **`Cafeteria = 7` (yeni)** | Info | hayır | `MEAL_MENU_PUBLISHED` | Student, Parent |
| 61 | `MealMenuChanged` | `Cafeteria` | Warning | hayır | `MEAL_MENU_CHANGED` | Student, Parent |

Yeni grubun gerekçesi: Homework ve Club emsali. Zil sekmesinde "Duyurular"a karışmasın. Değer sona eklenir.

Değişecek dosyalar:

- `NotificationKind.cs`, `NotificationEventGroup.cs`, `NotificationKindMetadata.cs`, `PushEventKeyMap.cs` (Map ve Audiences).
- `NotificationEventTypeSeedData` için iki satır: `push: true`, `delivered: true`, K-34 gereği varsayılan açık.
- `PushDeepLinks.MealMenu(DateOnly date)` → `/meal-menu?date=2026-11-02`.
- `NotificationSourceTypes` için ekleme gerekmez.

Push kapsamının gerekçesi: yayın grup başına ayda bir ya da iki kez olur. Değişiklik bildirimi zamana duyarlıdır. Kitle tek grup ailesidir. Fan-out en çok okulun tamamı kadardır, bu da `ANNOUNCEMENT` ile aynı ölçüdür.

### 5.2 Tetikler

| Olay (Domain) | Ne zaman yayılır | Bildirim |
|---|---|---|
| `MealMenuRangePublishedEvent(SchoolId, GroupId, From, To, OperationId, FirstPublishedCount)` | `:publish` sonunda, **en az bir gün ilk kez yayınlandıysa** | `MealMenuPublished`, bir kez (BR-YM-18) |
| `MealMenuDayChangedEvent(SchoolId, GroupId, Date)` | `Published` bir günün içeriği değişince: öğün, yemek, kalori, öğün notu, gün notu ya da boşaltma. Kaynak upsert, not ucu ya da üzerine yazan kopyalama olabilir. | Yalnız tarih **bugün ya da yarın** ise `MealMenuChanged`. Daha uzak tarihte bildirim gitmez, yalnız `isUpdated` işaretlenir (BR-YM-19). |
| Geri çekme | — | **Yok** (Y-15) |
| Katalogda yemek değişikliği | — | **Yok.** Değişiklik günün değil yemeğin olayıdır. Bildirimle yayılırsa gürültü olur. |

"Yarın"ın tanımı şöyle yorumlandı: bugünden sonraki **ilk okul günü**. Hafta sonu ve tatil atlanır, kontrol `ISchoolCalendarService.IsSchoolDayAsync` ile yapılır. Gerekçe: Cuma 15:00'te Pazartesi menüsü değişirse velinin haberdar olması Y-10'un amacıdır. Kural tek bir fonksiyonda yaşar (`MealMenuNotificationRules.IsNearDate`). Faruk takvim yarını isterse tek satır değişir.

### 5.3 Alıcı çözümü (handler'da)

1. **Modül kapısı.** `IModuleAvailability` kapalı derse bildirim yazılmaz (BR-YM-21).
2. **Grubun kademeleri** okunur.
3. **Hedef öğrenciler** bulunur: `StudentProfile.IsActiveStudent` taşıyan ve `CurrentClassroomId → ClassRoom.GradeLevelId → GradeLevel.EducationLevel` zincirinde kademesi gruptaki kademelerden biri olan öğrenciler. Bu sorgu tek bir iç okuyucuda yaşar (`MealMenuAudienceReader`). Uç 21 ve 22 de grup çözümü için aynı okuyucuyu kullanır.
4. **Veliler** `INotificationRecipientResolver.ResolveGuardianPersonIdsAsync(…, includeInfoRestricted: false)` ile bulunur. Varsayılan güvenli taraftır, `TB-78`. Ardından `ResolveGuardianAccountsMapAsync` çağrılır.
5. **Öğrenci hesapları** yalnız `Middle` ve `High` kademeleri için eklenir, `ResolvePersonAccountsMapAsync` ile. Kural KR-03 ile aynıdır: anaokulu ve ilkokul öğrencisi bağımsız bildirim muhatabı sayılmaz. Bu kural Cafeteria'da yerel bir statik kural olarak yazılır, Announcements domain'ine bağımlılık kurulmaz.
6. **Tekilleştirme.** Hesap kümesi birleşir. Aynı gruptaki iki çocuğun velisi tek bildirim alır. Farklı gruplardaki çocukların velisi grup başına bir bildirim alır.
7. **Öğretmen ve idare bildirim almaz.** Menüyü görürler ama bildirim almazlar (Y-11 görünürlükle sınırlıdır).

### 5.4 Birleştirme ve tekilleştirme

| Tür | `eventId` (deterministik) | Birleştirme |
|---|---|---|
| `MealMenuPublished` | `DeterministicGuid.Combine(schoolId, groupId, from, to, "MEAL_MENU_PUBLISHED", operationId)` | Yok. İşlem başına tektir, yeniden deneme teslim kaydında tekilleşir. |
| `MealMenuChanged` | `DeterministicGuid.Combine(schoolId, groupId, date, "MEAL_MENU_CHANGED", occurredAt.UtcTicks)` | **Redis cooldown**: anahtar `tenant:{schoolId}:cooldown:meal-menu-changed:{groupId}:{yyyy-MM-dd}`, TTL 30 dakika. Anahtar varsa atlanır, yoksa yazılır ve gönderilir (ADR-YM-9). |

Sessiz saat mevcut hatta uygulanır: push 22:00–07:00 arasında sabaha kayar. Uygulama içi bildirim kapatılamaz (BR-YM-20). Günlük "bugünün menüsü" push'u **yoktur**.

### 5.5 İçerik ve derin bağlantı

| Tür | Başlık | Gövde | Derin bağlantı |
|---|---|---|---|
| Published (aralık tek ayda ve en az 20 gün) | "Kasım menüsü yayınlandı" | "2–30 Kasım yemek menüsü yayında." | `/meal-menu?date={from}` |
| Published (diğer) | "Yemek menüsü yayınlandı" | "2–6 Kasım haftasının menüsü yayında." | aynı |
| Changed (bugün) | "Bugünkü menü değişti" | "Güncel menüyü görmek için dokunun." | `/meal-menu?date={date}` |
| Changed (bir sonraki okul günü) | "Yarınki menü değişti" ya da "Pazartesi menüsü değişti" | aynı | aynı |

- Gövde içerik taşımaz. Bildirim bir haberdir (`GradePublished` ilkesi).
- Bağlantı rolden bağımsızdır. Veli tarafında istemci aktif çocuktan çözer (2026-08-29 kararı).
- İstemci `resolveNotificationTarget` fonksiyonuna `/meal-menu` dalını ekler. Biçimi tanımayan eski istemci satırı tıklanamaz bırakır.

---

## 6. Modül, izin ve göç

### 6.1 Modül kapalıyken davranış (BR-YM-21)

| Yüzey | Davranış |
|---|---|
| Bütün `/meal-menus/*` uçları | 403 `Cafeteria.ModuleDisabled`. Veri silinmez. |
| `me/context.enabledModules` | `cafeteria` listede yer almaz |
| Web ve mobil gezinme | `NavItem.requiresModule: "cafeteria"` (yeni alan) taşıyan öğe çizilmez. Derin bağlantıyla gelinirse "Bu modül okulunuzda kapalı" boş durumu gösterilir. |
| Anasayfa kartı | Çizilmez, istek de atılmaz |
| Bildirim | Handler kapıda durur, yeni bildirim üretilmez. Daha önce gönderilmiş bildirimler kalır, dokunulunca boş durum açılır. |
| Ayarlar › Modüller | Kart görünür (D-1 göçünden sonra) ve açılıp kapatılabilir |

BR-YM-22 seçimi şöyle yapıldı. Y-14 (b) kabul edildiği için modülün açık olması bilinçli bir idare eylemidir. Bu yüzden modül açık ama yayında gün yoksa gezinme öğesi **görünür**, sayfada "Henüz yayınlanmış menü yok." boş durumu gösterilir. Anasayfa kartı ise 7 gün içinde yayında gün yoksa gizlenir (BR-YM-14). Gezinmeyi veriye göre gizlemek her açılışta ek bir istek demektir. Karşılığında kazanılan tek şey, yeni açılmış modülün ilk birkaç gününde menü öğesinin görünmemesidir.

### 6.2 İzinler (`PermissionSeedData`, `RolePermissionSeedData`, `MasterSeedIds`)

```csharp
// PermissionSeedData — CAFETERIA
Row(MasterSeedIds.Permissions.CafeteriaRead,    "CAFETERIA", "READ",     "cafeteria.read",     "Yemek menüsünü görüntüle (kendi menü grubu)"),
Row(MasterSeedIds.Permissions.CafeteriaViewAll, "CAFETERIA", "VIEW_ALL", "cafeteria.view-all", "Tüm menü gruplarının yayındaki menüsünü görüntüle"),
Row(MasterSeedIds.Permissions.CafeteriaManage,  "CAFETERIA", "MANAGE",   "cafeteria.manage",   "Menü grubu, öğün, yemek kataloğu ve günlük menü girişi; yayınla / geri çek"),
```

| İzin | SuperAdmin | SchoolAdmin | VicePrincipal | Teacher | Parent | Student |
|---|---|---|---|---|---|---|
| `cafeteria.read` | ✓ (katalog) | ✓ (katalog) | ✓ (ayna) | ✓ | ✓ | ✓ |
| `cafeteria.view-all` | ✓ (katalog) | ✓ (katalog) | ✓ (ayna) | ✓ | — | — |
| `cafeteria.manage` | — | ✓ (ayrı satır) | ✓ (ayna) | — | — | — |

- `read` ve `view-all` `AllPermissionIds()` kataloğuna girer.
- `manage` kataloğa girmez. `exams.manage` ve `clubs.manage` kalıbı izlenir: platform hesabı okul adına menü yayınlayamaz.
- Göç: `HasData` değişikliği olduğu için EF göçü üretilir. Mevcut oturumların izin önbelleği 30 dakikalık TTL ile ya da `perms_ver` artışıyla yenilenir. Bu adım sınav modülü izin göçünün emsaliyle aynı yoldan yapılmalıdır.
- **Y-9 (b) "izin verilen personel" bugünkü altyapıyla kurulamıyor** (D-4). MVP'de `manage` yalnız idarededir, yani müdür ve müdür yardımcısı. Karar sorusu §10'da.

### 6.3 `SeedDefaultModuleConfigsHandler` değişikliği ve mevcut okullar

1. **Handler:** `new("cafeteria", IsEnabled: false, PlanRestricted: false, Tier: ModuleTier.Standard)` (Y-14). Yorum satırına "Yemekhane: Y-14 varsayılan kapalı; açılınca menü tohumu (ADR-YM-8)" notu düşülür.
2. **Göç `YYYYMMDD_cafeteria_meal_menu`.** Tek göçtür ve **sıra önemlidir**:
   1. Yedi tabloyu ve indeksleri oluştur (§2.3).
   2. Var olan **bütün okullar** için `school.school_module_configs` satırında `module_name = 'cafeteria'` değerini `is_enabled = 0` yap. Satırı olmayan okula `is_enabled = 0` satırı ekle. **Gerekçe:** bugün `true` olan satırlar kimsenin bilinçli kararı değil. Modül plan kataloğunda olmadığı için listelenmiyordu ve açma isteği 409 alıyordu (D-1). Bu adım atlanırsa 3. adım bütün okullarda modülü sessizce açar, çünkü çözücü satırı olmayan okulda planı esas alır.
   3. `master.plan_modules` tablosuna `('Free','cafeteria')`, `('Standard','cafeteria')` ve `('Premium','cafeteria')` satırlarını idempotent `NOT EXISTS` ile ekle. UI kataloğundaki `tag: "free"` her planda bulunmak anlamına gelir. Plan ve fiyat kararı Faruk'tadır, öneri tüm planlardır.
   4. İzin satırları (EF `HasData` farkı).
3. **Pilot okul:** idare modülü Ayarlar › Modüller'den açar. "Tüm okul" grubu ve üç öğün bu anda tohumlanır.
4. **UI kataloğu:** `MODULE_CATALOG.cafeteria.description` değeri `"Yemek menüsü ve alerjen bilgisi."` olur (Y-1: yoklama gelince eklenir). Analiz §5'teki "UI mock'unda kapalı" bilgisi yeni varsayılanla tutarlı hâle gelir.
5. **Belge:** `genel/mvp-kapsami.md` §3'teki "Yemekhane, Sprint 5+" satırı güncellenmeli (analiz §10'daki risk). Bu adım Faruk onayından sonra yapılır.

**Dağıtım notu (D-7).** Göç sonrası Redis'teki `module-configs` ve `plan-status` (tenant önekli) önbellek anahtarları TEMİZLENMELİ (örn. `RemoveByPrefix` ya da Redis'te ilgili anahtarların silinmesi): göç tercih satırlarını doğrudan SQL ile değiştirir, uygulama bu önbelleği geçersiz kılmaz; temizlenmezse Ayarlar › Modüller eski durumu (cafeteria açık) gösterebilir. `IModuleAvailability` önbellek kullanmaz, doğrudan veritabanını okur. Ayrıca: tercih satırı yoksa ya da silinmişse `cafeteria` KAPALI sayılır (diğer modüller `PlanModuleResolver` varsayılanını korur); göç silinmiş cafeteria satırlarını kapalı olarak geri getirir.

**Geri alma.** Göçün `Down` adımı tabloları düşürür, `plan_modules` satırlarını siler ve tercih satırlarını olduğu gibi bırakır. Veri kaybı yalnız menü verisidir. Pilot sonrasında geri alma ancak yedekle yapılmalıdır.

---

## 7. UI tarafı

### 7.1 Rotalar

**Web (`apps/web/app/(dashboard)/meal-menu/…`).** `/exams` deseni izlenir: aynı rota role göre farklı yüz gösterir.

| Rota | Kim | İçerik |
|---|---|---|
| `/meal-menu` | `cafeteria.manage` | **Menü Planlama.** Grup seçici, ay ızgarası (satır öğün, sütun gün, hafta sonu gizli), hücre içi yemek girişi (katalog otomatik tamamlama ve yeni yemek), öğün kalorisi ve notu, gün notu. Tatil hücreleri gri ve uyarılıdır. Taslak ve yayın rozeti, "güncellendi" işareti. Araç çubuğu: "Bu ayı yayınla", "Bu haftayı yayınla", "Geri çek", "Geçen haftayı kopyala", "Günü kopyala…", "Yazdır". |
| `/meal-menu` | `view-all` taşıyan ama `manage` taşımayan (öğretmen) | Grup seçicili salt okunur hafta ve ay görünümü, "Yazdır" |
| `/meal-menu` | veli, öğrenci | Çocuğun (ya da kendi) menüsü: hafta listesi, ay görünümü, alerjen çipleri, alerjen açıklaması, sabit uyarı. Veli çok çocukluysa çocuk seçici gösterilir. `NoGroup` ve `NoClassroom` boş durumları vardır. |
| `/meal-menu/print?groupId=…&month=2026-11` | manage, view-all | Yazdırma görünümü (M-10, Y-16 (a)): yatay A4, print CSS, okul adı `me/context.schoolName`'den, alt bilgide alerjen lejantı ve sabit uyarı |
| `/meal-menu/dishes` | manage | Yemek Kataloğu: liste, arama, kategori ve alerjen düzenleme, pasifleştirme |
| `/meal-menu/settings` | manage | Menü Grupları (kademe atama, taşıma onayı, eşlenmemiş kademe uyarısı) ve Öğünler (sıra, pasifleştirme) |
| `/` (Panel) | herkes | "Bugünün menüsü" kartı (uç 22) |

**Mobil (`apps/mobile/src/app/(tabs)/(home,more)/meal-menu.tsx`).** Y-17 (a) uyarınca mobilde giriş yoktur.

| Ekran | İçerik |
|---|---|
| `meal-menu` | Haftalık gün kartları (öğün başlığı, yemekler, alerjen çipleri, kalori), hafta kaydırma, ay ızgarası geçişi. Veli aktif çocuğu kullanır (`ChildContextSheet`). İdare ve öğretmen grup seçiciyle salt okunur görür. `?date=` derin bağlantısı ilgili haftayı açar ve günü vurgular. |
| Anasayfa kartı | "Bugünün menüsü" ya da "Pazartesi'nin menüsü". Satır "Mercimek çorbası · Fırın tavuk · Pirinç pilavı · Ayran" biçiminde gösterilir, altında alerjen çipleri durur. Kart `meal-menu` ekranına açılır. |

**Gezinme (`packages/core/src/nav/nav-config.ts`):**

- `NavItem.requiresModule?: string` alanı yeni eklenir. Değerlendirme fonksiyonu `isModuleLocked(item, enabledModules)` öğeyi **gizler**.
- Öğe: `{ id: "meal-menu", label: "Yemek Menüsü", href: "/meal-menu", requiresModule: "cafeteria" }`. Yönetici, öğretmen, veli ve öğrenci rollerinin hepsinde bulunur.
- `mobileModuleEntryTab` içine `"/meal-menu": ["home", "more"]` eklenir.
- `requiresLinkedChild` listesine `/meal-menu` eklenir (veli için).

### 7.2 `packages/core` içine girecekler

`packages/core/src/meal-menu/`:

```ts
// types.ts
export type Allergen =
  | "Gluten" | "Crustaceans" | "Eggs" | "Fish" | "Peanuts" | "Soybeans" | "Milk"
  | "TreeNuts" | "Celery" | "Mustard" | "Sesame" | "Sulphites" | "Lupin" | "Molluscs"
export type DishCategory = "Soup" | "MainCourse" | "SideDish" | "DessertOrFruit" | "Beverage"
export type MealMenuDayStatus = "Draft" | "Published"
export type MealMenuEducationLevel = "Preschool" | "Primary" | "Middle" | "High"   // backend EducationLevel; sözlükte `schoolStage`
export type MyMealMenuStatus = "Ok" | "NoClassroom" | "NoGroup"
export type MealMenuAuditKind = "Created" | "Updated" | "NoteChanged" | "Published" | "Withdrawn" | "Copied" | "Cleared"

// constants.ts
export const ALLERGENS: readonly { code: Allergen; label: string; shortLabel: string }[] = [ /* §4.1 sırası */ ]
export const DISH_CATEGORY_LABEL: Record<DishCategory, string> = { Soup: "Çorba", MainCourse: "Ana yemek", SideDish: "Yardımcı yemek", DessertOrFruit: "Tatlı / Meyve", Beverage: "İçecek" }
export const SCHOOL_STAGE_LABEL: Record<MealMenuEducationLevel, string> = { Preschool: "Anaokulu", Primary: "İlkokul", Middle: "Ortaokul", High: "Lise" }
export const MEAL_MENU_DAY_STATUS_LABEL = { Draft: "Taslak", Published: "Yayında" } as const
export const MEAL_MENU_ALLERGEN_NOTICE =
  "Alerjen bilgisi okulun beyanıdır; çocuğunuzun özel durumu için okulla görüşün."   // BR-YM-17, hukuki teyit bekliyor
export const MEAL_MENU_NOTE_HINT =
  "Not alanına öğrenci adı veya sağlık bilgisi yazmayın."                             // KVKK riski (analiz §10)
export const MEAL_MENU_LIMITS = { maxRangeDays: 62, maxCopySourceDays: 35, maxCopyTargets: 31,
  maxDishesPerEntry: 12, maxCalories: 5000, noteMax: 200, dishNameMax: 120, groupNameMax: 60, mealPeriodNameMax: 40 } as const

// logic.ts (saf fonksiyonlar, vitest'li)
export function dishLine(entry): string                       // "Mercimek çorbası · Fırın tavuk · …"
export function entryAllergens(entry): Allergen[]             // öğünün birleşik alerjenleri, ALLERGENS sırasıyla
export function dayDisplayState(date, day | undefined, holidays): "holiday" | "noMenu" | "notPublished" | "menu"
export function publishShortcutRange(kind: "month" | "week", anchor: Date): { from: string; to: string }
export function previousWeekCopy(anchorMonday: string): { sourceFrom; sourceTo; targetFrom }   // kayma 7
export function todayCardTitle(card): string                  // "Bugünün menüsü" | "Pazartesi'nin menüsü"
```

Bildirimler (`packages/core/src/notifications/constants.ts` ve `logic.ts`):

- `NOTIFICATION_KIND_CONFIG.MealMenuPublished = { label: "Menü yayınlandı", icon: "calendar" }`
- `MealMenuChanged = { label: "Menü değişti", icon: "alert" }`
- `NOTIFICATION_GROUP_ORDER` sonuna `"Cafeteria"` eklenir, `NOTIFICATION_GROUP_LABEL.Cafeteria = "Yemekhane"` olur.
- `resolveNotificationTarget` fonksiyonuna `/meal-menu?date=` dalı eklenir.

Modüller (`packages/core/src/modules/constants.ts`):

- `cafeteria.description` güncellenir (§6.3).

Oturum bağlamı:

- `enabledModules: string[]` alanı okunur. Alan yoksa (eski API) `cafeteria` kapalı sayılır, güvenli taraf budur.

`packages/api/src/meal-menu/`:

- `contract.ts`: mock-first dönemde `declare module "../generated/schema"` ile §3 tiplerini taşır. Codegen gelince silinir.
- `endpoints.ts` ve `queries.ts`. Sorgu anahtarları: `qk.mealMenu.groups()`, `periods()`, `dishes(params)`, `range(groupId, from, to, includeDrafts)`, `my(studentId, from, to)`, `today(studentId)`.
- Mutasyonlardan sonra geçersiz kılma: gün yazımı `range` ve `today` anahtarlarını, yayın ve geri çekme `range`, `my` ve `today` anahtarlarını tazeler.

### 7.3 MSW mock'ları için örnek yükler

`packages/api-mocks/src/meal-menu/{meal-menu-data.ts, meal-menu-handlers.ts}`. Senaryolar: `mealMenuEmpty` (modül açık, yayın yok), `mealMenuPublished`, `mealMenuModuleDisabled` (403), `mealMenuNoGroup`.

**`GET /api/v1/meal-menus/groups`**

```json
{
  "data": {
    "groups": [
      { "id": "11111111-0000-0000-0000-000000000001", "name": "Anaokulu", "sortOrder": 1,
        "educationLevels": ["Preschool"],
        "mealPeriodIds": ["22222222-0000-0000-0000-000000000001", "22222222-0000-0000-0000-000000000002", "22222222-0000-0000-0000-000000000003"] },
      { "id": "11111111-0000-0000-0000-000000000002", "name": "İlkokul–Lise", "sortOrder": 2,
        "educationLevels": ["Primary", "Middle"],
        "mealPeriodIds": ["22222222-0000-0000-0000-000000000002"] }
    ],
    "schoolEducationLevels": ["Preschool", "Primary", "Middle", "High"],
    "unassignedEducationLevels": ["High"]
  },
  "meta": null, "errors": null, "correlationId": "mock"
}
```

**`GET /api/v1/meal-menus/dishes?search=mer&pageSize=10`**

```json
{
  "data": {
    "items": [
      { "id": "33333333-0000-0000-0000-000000000001", "name": "Mercimek çorbası", "category": "Soup",
        "allergens": ["Gluten", "Celery"], "isActive": true, "isInUse": true }
    ],
    "page": 1, "pageSize": 10, "totalCount": 1, "totalPages": 1, "hasPreviousPage": false, "hasNextPage": false
  },
  "meta": null, "errors": null, "correlationId": "mock"
}
```

**`GET /api/v1/meal-menus/groups/{id}/days?from=2026-10-26&to=2026-10-30&includeDrafts=true`.** Örnekte yayında bir gün, güncellenmiş ve not taşıyan bir gün, tatil (28 Ekim yarım gün, 29 Ekim resmî) ve bir taslak gün var.

```json
{
  "data": {
    "group": { "id": "11111111-0000-0000-0000-000000000002", "name": "İlkokul–Lise" },
    "from": "2026-10-26", "to": "2026-10-30",
    "mealPeriods": [ { "id": "22222222-0000-0000-0000-000000000002", "name": "Öğle Yemeği", "sortOrder": 2 } ],
    "days": [
      { "date": "2026-10-26", "status": "Published", "note": null, "isEmpty": false,
        "isUpdated": false, "updatedAt": null, "rowVersion": "AAAAAAAAB9E=",
        "entries": [
          { "mealPeriodId": "22222222-0000-0000-0000-000000000002", "mealPeriodName": "Öğle Yemeği", "sortOrder": 2,
            "note": null, "calories": 650,
            "dishes": [
              { "dishId": "33333333-0000-0000-0000-000000000001", "name": "Mercimek çorbası", "category": "Soup", "allergens": ["Gluten", "Celery"] },
              { "dishId": "33333333-0000-0000-0000-000000000002", "name": "Fırın tavuk", "category": "MainCourse", "allergens": [] },
              { "dishId": "33333333-0000-0000-0000-000000000003", "name": "Pirinç pilavı", "category": "SideDish", "allergens": [] },
              { "dishId": "33333333-0000-0000-0000-000000000004", "name": "Ayran", "category": "Beverage", "allergens": ["Milk"] }
            ] }
        ] },
      { "date": "2026-10-27", "status": "Published", "note": "Balık tedarik edilemedi; menü değiştirildi.", "isEmpty": false,
        "isUpdated": true, "updatedAt": "2026-10-26T14:05:00+03:00", "rowVersion": "AAAAAAAAB+A=",
        "entries": [
          { "mealPeriodId": "22222222-0000-0000-0000-000000000002", "mealPeriodName": "Öğle Yemeği", "sortOrder": 2,
            "note": null, "calories": 700,
            "dishes": [
              { "dishId": "33333333-0000-0000-0000-000000000005", "name": "Ezogelin çorbası", "category": "Soup", "allergens": ["Gluten"] },
              { "dishId": "33333333-0000-0000-0000-000000000006", "name": "Etli kuru fasulye", "category": "MainCourse", "allergens": [] },
              { "dishId": "33333333-0000-0000-0000-000000000007", "name": "Sütlaç", "category": "DessertOrFruit", "allergens": ["Milk"] }
            ] }
        ] },
      { "date": "2026-10-30", "status": "Draft", "note": null, "isEmpty": false,
        "isUpdated": false, "updatedAt": null, "rowVersion": "AAAAAAAAB+k=",
        "entries": [
          { "mealPeriodId": "22222222-0000-0000-0000-000000000002", "mealPeriodName": "Öğle Yemeği", "sortOrder": 2,
            "note": null, "calories": null,
            "dishes": [ { "dishId": "33333333-0000-0000-0000-000000000008", "name": "Karnıyarık", "category": "MainCourse", "allergens": [] } ] }
        ] }
    ],
    "holidays": [
      { "name": "Cumhuriyet Bayramı Arifesi (yarım gün)", "startDate": "2026-10-28", "endDate": "2026-10-28", "source": "OFFICIAL" },
      { "name": "Cumhuriyet Bayramı", "startDate": "2026-10-29", "endDate": "2026-10-29", "source": "OFFICIAL" }
    ]
  },
  "meta": null, "errors": null, "correlationId": "mock"
}
```

**`GET /api/v1/meal-menus/me/today`.** Çağıran iki çocuklu bir veli. Çocuklar farklı gruplarda, bugün 2026-10-15 (Perşembe).

```json
{
  "data": {
    "cards": [
      { "group": { "id": "11111111-0000-0000-0000-000000000001", "name": "Anaokulu" },
        "students": [ { "personId": "44444444-0000-0000-0000-000000000001", "fullName": "Ela Kaya" } ],
        "date": "2026-10-15", "isToday": true,
        "day": { "date": "2026-10-15", "status": "Published", "note": null, "isEmpty": false, "isUpdated": false,
                 "updatedAt": null, "rowVersion": null,
                 "entries": [
                   { "mealPeriodId": "22222222-0000-0000-0000-000000000001", "mealPeriodName": "Kahvaltı", "sortOrder": 1, "note": null, "calories": 350,
                     "dishes": [ { "dishId": "33333333-0000-0000-0000-000000000009", "name": "Haşlanmış yumurta", "category": null, "allergens": ["Eggs"] },
                                 { "dishId": "33333333-0000-0000-0000-000000000010", "name": "Beyaz peynir", "category": null, "allergens": ["Milk"] } ] },
                   { "mealPeriodId": "22222222-0000-0000-0000-000000000002", "mealPeriodName": "Öğle Yemeği", "sortOrder": 2, "note": null, "calories": 500,
                     "dishes": [ { "dishId": "33333333-0000-0000-0000-000000000001", "name": "Mercimek çorbası", "category": "Soup", "allergens": ["Gluten", "Celery"] } ] }
                 ] } },
      { "group": { "id": "11111111-0000-0000-0000-000000000002", "name": "İlkokul–Lise" },
        "students": [ { "personId": "44444444-0000-0000-0000-000000000002", "fullName": "Can Kaya" } ],
        "date": "2026-10-15", "isToday": true,
        "day": { "date": "2026-10-15", "status": "Published", "note": "Okul gezisi: kumanya verilecek.", "isEmpty": true,
                 "isUpdated": false, "updatedAt": null, "rowVersion": null, "entries": [] } }
    ]
  },
  "meta": null, "errors": null, "correlationId": "mock"
}
```

**`GET /api/v1/meal-menus/me?from=2026-11-02&to=2026-11-06&studentPersonId=…`** (`NoGroup`)

```json
{ "data": { "status": "NoGroup", "student": { "personId": "44444444-0000-0000-0000-000000000003", "fullName": "Deniz Kaya" }, "menu": null },
  "meta": null, "errors": null, "correlationId": "mock" }
```

**`POST /api/v1/meal-menus/groups/{id}:publish`** `{ "from": "2026-11-01", "to": "2026-11-30" }`

```json
{ "data": { "publishedDates": ["2026-11-02", "2026-11-03"], "emptyDatesMarked": ["2026-11-27"],
            "skippedPastDates": [], "notificationSent": true },
  "meta": null, "errors": null, "correlationId": "mock" }
```

**Hata: modül kapalı (403)**

```json
{ "data": null, "meta": null,
  "errors": [ { "code": "Cafeteria.ModuleDisabled", "message": "Yemekhane modülü bu okulda kapalı.", "field": null } ],
  "correlationId": "mock" }
```

**Hata: geçmiş gün (409)**

```json
{ "data": null, "meta": null,
  "errors": [ { "code": "Cafeteria.Day.PastDateLocked", "message": "Geçmiş günün menüsü değiştirilemez; yalnız not eklenebilir.", "field": null } ],
  "correlationId": "mock" }
```

---

## 8. Dilimleme ve bağımlılık sırası

> **Ortak dosyalar sıralı ilerler.** `NotificationKind`, `PushEventKeyMap`, izin seed'leri, `ResultExtensions`, `ContextView` ve `nav-config.ts` başka ekiplerin de dokunduğu dosyalardır. Bu yüzden B0 ve W0 ilk ve tek seferde yapılır. Panoda (coordinator) önceden duyurulmalıdır.

```
              ┌──────── W0 (core + contract + MSW) ───────┬── W1..W5 (web, mock'a karşı)
bu belge ─────┤                                           └── M1..M2 (mobil, mock'a karşı)
              └── B0 → B1 → B2 → B3 → B4 → B5 → B6 ── codegen ── W*/M* gerçek uca bağlanır
```

| Dilim | Kapsam | Önkoşul | Çıktı / kabul |
|---|---|---|---|
| **B0** Altyapı | `IModuleAvailability` (Schools) ve `ContextView.enabledModules`. Üç izin (seed ve göç). `ResultExtensions` içine `Cafeteria.*` kovası ve `ErrorMessageCatalog`. `NotificationKind` 60/61, `NotificationEventGroup.Cafeteria`, `NotificationKindMetadata`, `PushEventKeyMap`, katalog seed'i, `PushDeepLinks.MealMenu`. | — | `test-changed` yeşil. Bekçi testleri geçer (`NotificationKindMetadataTests`, `PushEventKeyMapTests`, `ErrorMessageCatalogTests`, `MasterRoleSeedTests`). |
| **B1** Model | Domain (5 kök ve çocuklar, enum'lar, kurallar), EF konfigürasyonu, göç (tablolar ve §6.3 adım 2–3), `SeedDefaultModuleConfigsHandler` değişikliği, açılışta tohum handler'ı | B0 | Domain birim testleri: geçmiş tarih kilidi, ilk yayın kapısı, boş gün, normalizasyon. Göç boş veritabanında ve dolu kopyada denenir. |
| **B2** Ayar uçları | Uç 1–12 | B1 | Entegrasyon: kademe tekilliği, taşıma, son grup, kullanımdaki öğün ve yemek |
| **B3** Gün | Uç 13–16, `MealMenuAudienceReader` | B2 | Upsert, `rowVersion`, denetim anlık görüntüsü, tatil bilgisi |
| **B4** Yayın ve kopya | Uç 17–20 | B3 | Hafta günü hizası, atlama listeleri, yayın sonucu |
| **B5** Görüntüleme | Uç 21–22 | B3 (B4 ile paralel olabilir) | Veli ilişki kapısı (403), `NoGroup`, kart tarih seçimi, çok çocuk birleştirme |
| **B6** Bildirim | İki handler, cooldown, alıcı çözümü | B4, B5 | Çözücü testi, cooldown testi (aynı olay iki kez gelirse ikincisi atlanır), idempotency testi, sessiz saat (notification-rules §13) |
| **W0** Ortak katman | Sözlüğe eklemeler (`oksis-ui/CLAUDE.md`), `packages/core/src/meal-menu/*` (vitest), bildirim sabitleri ve `resolveNotificationTarget`, `NavItem.requiresModule` ve `enabledModules`, `packages/api/src/meal-menu/{contract,endpoints,queries}.ts`, MSW verisi ve handler'ları (§7.3) | yalnız bu belge | typecheck, lint ve vitest yeşil |
| **W1** Ayarlar | `/meal-menu/settings` (gruplar ve öğünler) | W0 | MSW ile taşıma onayı ve eşlenmemiş kademe uyarısı |
| **W2** Katalog | `/meal-menu/dishes` | W0 | Alerjen çok seçimi, pasifleştirme, "N günde kullanılıyor" uyarısı |
| **W3** Planlama ızgarası | `/meal-menu` (manage): ızgara, hücre editörü, otomatik tamamlama, kopyalama, yayınlama ve geri çekme, denetim paneli | W0, W2 bileşenleri | 409 akışları (bayat kayıt, geçmiş gün, yayınlanacak bir şey yok) |
| **W4** Görüntüleme ve yazdırma | `/meal-menu` (aile ve öğretmen), `/meal-menu/print` | W0 | Print CSS ve A4 yatay kontrolü |
| **W5** Panel kartı ve bildirim | Panel kartı, zil eşlemesi | W0 | — |
| **M1** Mobil görüntüleme | `meal-menu.tsx`, gezinme kapısı, derin bağlantı | W0 | Expo web ve MSW ile veli (iki çocuk), öğrenci, öğretmen |
| **M2** Mobil kart | Anasayfa "Bugünün menüsü" | W0 | `NoGroup`, boş kart ve "Pazartesi'nin menüsü" durumları |

Kritik yol: **B0 → B1 → B3 → B5**. Görüntüleme uçlarının gerçek veriyle bağlanabilmesi buna bağlıdır. Web ve mobil W0'dan sonra backend'i beklemeden MSW'ye karşı ilerler. Codegen geldiğinde `contract.ts` silinir ve typecheck uyuşmazlığı ortaya çıkarır.

---

## 9. Riskler ve notlar

| Risk ya da not | Azaltma |
|---|---|
| Göç adım 2 atlanırsa modül bütün okullarda sessizce açılır (D-1, D-3). | Adımlar aynı göçte, yazılı sırayla yapılır. Göç testi "satırı olmayan okul kapalı kalır" durumunu doğrular. |
| Katalogda yemek adı ya da alerjen değişince geçmiş görünüm de değişir (BR-YM-06). | Denetim anlık görüntüsü tutulur (ADR-YM-2/5). Katalog düzenleme ekranı uyarı gösterir. |
| Gruptan çıkarılan öğünün kayıtları görünmez olur ama silinmez. | PUT uyarısı gösterilir. Öğün gruba geri eklenirse kayıtlar yeniden görünür. |
| Cooldown penceresi içindeki ikinci değişiklik için bildirim gitmez (ADR-YM-9). | Bilinçli kabul edildi. Gövde içerik taşımaz, bağlantı güncel veriyi açar. |
| Geri çekilip yeniden yayınlanan aralık "yayınlandı" bildirimi üretmez (`FirstPublishedAt`). | Yanlış ay yayını senaryosunda doğru davranış budur. Düzeltilen içerik bugün ya da yarındaysa "değişti" bildirimi gelir. |
| Cafeteria, öğrenciden kademeye gitmek için Users, AcademicSessions ve Academics tablolarını `IApplicationDbContext` üzerinden okur. Bu, sınav modülündeki mevcut uygulamayla aynı. Kural yalnız soyutlama üzerinden bağımlılık ister. | Bütün okumalar tek iç sınıfta yaşar (`MealMenuAudienceReader`). Bir `IStudentPlacementReader` soyutlaması doğarsa değişiklik tek dosyada kalır. Tatil, takvim, alıcı ve modül durumu zaten soyutlamayla okunuyor. |
| Not alanlarına sağlık bilgisi yazılması riski (KVKK). | Alanın yanında `MEAL_MENU_NOTE_HINT` sabiti gösterilir. |

---

## 10. Faruk'a soru (önerilerle çözülemeyen)

**Y-9 (b) "idare ve `cafeteria.manage` izni verilen personel" bugünkü yetki altyapısıyla uygulanamıyor.** İzinler yalnız platform genelindeki rol matrisinden geliyor. Okula ya da kişiye izin tanıma yok, sekreter veya personel rolü de tohumlanmamış (D-4). Seçenekler:

- **(a) MVP'de `cafeteria.manage` yalnız müdür ve müdür yardımcısında olsun.** Personel yetkilendirmesi, genel rol ve izin özelleştirmesi geldiğinde ek yapılmadan devreye girer, çünkü izin anahtarı hazırdır. **Öneri budur.**
- (b) İzni öğretmen rolüne de vermek. Bu, bütün öğretmenlere menü yayınlama yetkisi verir ve kapsamı aşar.
- (c) Modül içinde bir "menü sorumlusu" listesi (`cafeteria_editors`). Bu, izin sisteminin yanında ikinci bir yetki kapısı açar ve denetim ve yetki ekranlarında görünmez. Önerilmez.

Pilotta menüyü fiilen sekreter ya da idari işler giriyorsa (a) seçeneği bunu müdür yardımcısı hesabına yükler. Bu kabul edilebilir mi?

> **Önerilerle çözüldü, yalnız bilgi:**
> - Plan erişimi: `cafeteria` tüm planlarda (§6.3).
> - "Yarın" tanımı: bir sonraki okul günü (§5.2).
> - BR-YM-22: modül açıkken gezinme görünür, sayfada boş durum (§6.1).
> - Grup eşlemesi: yalnız kademe (ADR-YM-3).
> - BR-YM-17 uyarı metninin hukuki teyidi ürünü bloklamıyor. Metin core sabiti olduğu için tek yerden değişir.
