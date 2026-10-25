---
tags: [decision, domain/academic, domain/exams]
date: 2026-10-22
status: accepted
---

# 0028 — Merkezi sınav takvimi (OYSS) platform kataloğundan okula içe alınır; merkezi satırın tarihi kilitlidir, saati gerekçeyle değişir

**Durum:** Kabul edildi (2026-10-22, Faruk). Doğrulama bekleyen iki madde 2026-10-25'te kapandı (bkz. "Doğrulama bekleyenler").

## Bağlam

MEB 2026-27 eğitim-öğretim yılında **Ortak Yazılı Sınav Sistemi**'ni (OYSS) açıkladı:

- Kapsam ortaokul 5–7 ve lise 9–11. sınıflardır. Sorular MEB havuzundan gelir, zümre 2 soru ekler; sınav Yeni EBA üzerinden yürür.
- **Günler merkezîdir.** Ders saati öntanımlı olarak 2. derstir (iki sınavın aynı güne düştüğü günde 2. ve 4. ders). Okul müdürlüğü ihtiyaç hâlinde farklı bir ders saati planlayabilir. Süre 40 dakikadır. Sınavı o saatte o şubede derse giren öğretmen uygular. (Doğrulandı 2026-10-25, OYSS uygulama takvimi s.1.)
- 1. sınavlar 3–13 Kasım 2026'dadır. Ülke geneli ortak sınavlar 6, 7, 9 ve 10. sınıflarda yapılır; 1. dönem 1. sınav penceresinde yalnız **6. sınıf Matematik (11 Kasım)** ve **10. sınıf Türk Dili ve Edebiyatı (12 Kasım)** ülke genelidir. Diğer ülke geneli tarihler 5–6 Ocak, 6–7 Nisan ve 8–9 Haziran'dır. Ülke geneli sınav o dersin **1. Sınav**'ı sayılır. Mazeret sınavları 23–27 Kasım'dadır ve okul planlar. Asıl sınava ve mazeret sınavına geçerli mazeretle giremeyen öğrenciyi dersin öğretmeni, **mazeretin bittiği günü izleyen 5 iş günü içinde** sınava alır. MEB bu sınava ayrı bir ad vermez; OKSİS arayüzde "Ek sınav" der.
- Soru bazlı puanlar e-Okul'a girilir. Soru ve cevap anahtarı OKSİS'te tutulmaz.
- Yönetmelik: md. 45/1-g (günde en çok 2 sınav), md. 48 (mazeret), md. 49/2 (sonuç 10 iş günü içinde), itiraz süresi 5 gün.

Bu sistem [[Sınav Takvimi]] modülünün varsayımını tersine çeviriyor. Modül bugün "öğretmen kendi saatini seçer, idare yayınlar" diye kurgulanmış. OYSS'de ise gün dışarıdan gelir, saat öntanımlıdır, içerik MEB'indir. Modül **içerikten lojistiğe** yeniden konumlanıyor: OKSİS "hangi gün, kaçıncı ders, hangi şube, kim uygular" sorusunu cevaplar; soruyu ve puanı tutmaz.

Kodda bugünkü durum (oksis-api, 2026-10-22):

- `ScheduledExam` (`academic.scheduled_exams`) pencere × şube × ders satırıdır. Tarih ve ders saati serbesttir; merkezî kaynağa bağ yoktur.
- `ExamRevisionKind` beş değer taşır (1 `ExamMoved` … 5 `SessionMoved`). Değerler kalıcıdır, sonuna eklenir.
- Kural denetleyicisinde `EX-H01…H13` ve `EX-S01…S09` var; `EX-S10` boştur ([[0018-sinav-kurallari-denetleyicide]]).
- Platform verisi `master` şemasında yaşar ve müfredat içe aktarması iki kişi kuralını zaten uygular ([[0022-mufredat-yayimi-iki-kisi-kurali]]). Merkezî takvim, mazeret sınavı ya da `CentralExam*` adlı bir yapı yoktur.

## Seçenekler

- **A — Her okul MEB takvimini kendisi girer:** yeni tablo gerekmez. Ama her okul aynı 40 satırı elle girer. Yanlış girilen tarihin kaynağı yoktur. Kilit kavramı yoktur, çünkü kilidin dayanacağı bir "resmî değer" kaydı yoktur.
- **B — Platform kataloğu ve okula önizlemeli içe alma (önerilen ve seçilen):** MEB takvimi bir kez `master` şemasına girer ve iki kişi kuralıyla doğrulanır. Okul bu katalogdan kendi `ScheduledExam` satırlarını üretir. Üretilen satır katalog satırına kimlikle bağlıdır ve o anki MEB değerinin kopyasını taşır.
- **C — Katalogdan canlı okuma:** okulda satır üretilmez; takvim her okumada katalogla birleştirilir. Katalogdaki her düzeltme yayınlanmış takvimleri sessizce değiştirir; revizyon kaydı ve bildirim doğmaz. Ayrıca her okul okuması `master` ile `academic` şemalarını birleştirir.
- **D — Ayrı "merkezi sınav" türü ya da ayrı agregat:** `ScheduledExam`'ın yanına ikinci bir sınav kaydı açılır. Kural denetleyicisi, pano, ders programı etiketi ve not defteri beslemesi ikiye bölünür. Faruk'un K5 kararıyla da çelişir: müfredat "1. yazılı" diyorsa OKSİS de "1. yazılı" der.

## Karar

**B.**

1. **Katalog (göç M1).** `master.central_exam_calendars` ve `master.central_exam_calendar_entries` tabloları açılır. Yalnız ekleme yapılır: doğrulanmış takvim ve satırları değişmez. Düzeltme yeni sürümdür (`:new-version`).
2. **İki kişi kuralı.** Takvimi yükleyen kullanıcı aynı takvimi doğrulayamaz. Kural [[0022-mufredat-yayimi-iki-kisi-kurali]]'ndaki gibi durum geçişinin içinde yaşar, yönergeye bırakılmaz.
3. **Okul satırı (göç M2).** `ScheduledExam`'a üç alan eklenir, üçü de boş bırakılabilir: `CentralEntryId`, `CentralDate`, `CentralPeriod`. Son ikisi içe alma anındaki MEB değeridir. Böylece "saat MEB'den saptı mı" sorusu katalog okunmadan cevaplanır.
4. **Kilit.** `CentralEntryId != null` olan satıra **merkezi satır** denir.
   - Tarihi değişmez. Taşıma, öğretmenin gün seçmesi ve yerleşimin kaldırılması domain'de reddedilir.
   - Ders saati yalnız `:change-central-period` komutuyla ve gerekçeyle değişir. Yayınlanmış takvimde bu değişiklik **revizyon türü 6 `CentralPeriodChanged`** yazar. Bildirim, var olan "sınav taşındı" türüyle gider.
5. **`EX-S10` (yumuşak).** Merkezi satırda `EX-H01` (günlük sınır), `EX-H03` (sahibin kendi saati) ve `EX-H08` (duyuru payı) sert ihlal üretmez; `EX-S10` uyarısına yumuşar. `EX-H05`, `H06`, `H10`, `H11`, `H12` ve `H13` sert kalır, çünkü fiziksel çakışmayı MEB takvimi de çözemez.
   - S10 yalnız ders saati MEB'in saatinden saparsa gerekçe ister. Sapma yoksa yalnız görünür.
   - MEB saatinde o şubede ders yoksa, yani uygulayıcı bulunamazsa, uygulayan dersin öğretmeni olur ve S10 düşer.
6. **İçe alma.** Önizlemeli ve idempotenttir. Önizleme şunları gösterir: eşleşen satırlar, **eşleşmeyen dersler listesi** (katalogda olup okulun programında karşılığı olmayan dersler) ve pencere dışına düşen tarihler (`outOfWindowDates`). Aynı takvim sürümünü ikinci kez içe almak satır çoğaltmaz. Anahtar, var olan şube × ders × pencere tekilliği ile `CentralEntryId`'dir.
7. **Ülke geneli sınavlar.** Ayrı bir sınav türü açılmaz. Ülke geneli sınav, MEB'in adlandırdığı yazılı penceresine (ör. 1. yazılı) satır olarak bağlanır; katalog satırı kapsamı (okul geneli OYSS / ülke geneli) bilgi olarak taşır. Ülke geneli sınav o dersin 1. Sınav'ı sayılır (doğrulandı 2026-10-25).
8. **Bildirim.** Yeni bildirim türü açılmaz. Takvimin yayınlanması, yarın sınav var ve saat değişikliği için var olan türler kullanılır.
9. **Mazeret.** Mazeret sınavı `MakeupExam` adında ayrı bir agregattır ve Sürüm 2'de gelir (ayrıntı: "Sürüm 2 — mazeret sınavı"). Mazeret sınavının notu [[Notlar]] modülünde girilir. Bu modül notu tutmaz.

### Faruk kararları (2026-10-22)

| No | Karar |
|---|---|
| K1 | Hedef yalnız dev ortamıdır; OKSİS canlıda değil. |
| K2 | Öğrenci ve veli sınav ekranı yalnız mobildedir (2026-09-08 kararı korunur). |
| K3 | Çevrimdışı çalışma kapsam dışıdır; canlıya geçiş öncesi listesine not düşülür. |
| K4 | Test okulu Altınay'dır; dağıtım kanalı önemsizdir. |
| K5 | Adlandırma kuralı: müfredat "1. yazılı" diyorsa OKSİS de "1. yazılı" der; ayrı sınav türü yok. |
| K6 | Entegrasyon raporunun açık konuları 1, 2, 3, 5 ve 6 önerildiği gibi kabul edildi: tarih ve saat kilitli, platform kataloğu, `MakeupExam`, ders saati öntanımlı, soru bazlı puan yok. |

### Uygulamada netleşenler (2026-10-23)

Kod incelemesi ve uçtan uca testlerde verilen kararlar. Yukarıdaki maddeleri daraltır ya da açar; çelişkide bu bölüm geçerlidir.

| No | Konu | Karar |
|---|---|---|
| U1 | İki kişi kuralı (madde 2) | Doğrulayamayanlar yükleyen ve taslağı **düzelten herkestir**, yalnız son düzelten değil (`editor_ids`). Kural `CentralExamCalendar.CanBeVerifiedBy` içinde yaşar. Platform detayı aynı kuraldan `canVerify` döner. |
| U2 | EX-S10 kapsamı (madde 5) | `EX-H01`, ancak günün sınavlarının **tamamı merkeziyse** `EX-S10`'a yumuşar. Gün yerel sınav da içeriyorsa sert kalır, çünkü okul yerel sınavı kaydırabilir. Yerleştirme ve yayın aynı kuralı kullanır. Sert ret `409 exams.rule-violation` ve `ruleCodes` ile döner. |
| U3 | İzinli saatler (madde 4) | Katalog satırı `AllowedPeriods` taşır (ör. 2 ve 4). İzinli küme içindeki değişiklik gerekçe istemez. Küme dışına çıkmak 15–500 karakter gerekçe ister ve `isPeriodOverridden` olarak işaretlenir. |
| U4 | Ders saati numarası | Merkezi satırdaki saat **ders ordinalidir**, teneffüsler sayılmaz. Etiket ve saat aralığı zil şablonunun yalnız ders slotlarından üretilir. |
| U5 | Bildirim (madde 8) | Yeni tür açılmaz, gövde ayrımı yapılır. Uygulayıcı değişimi yalnız eski ve yeni uygulayıcıya gider. Yayındaki pencereye yeni eklenen merkezi satırlar şube başına tek özet olarak gider. Saat değişimi var olan "sınav taşındı" türünü kullanır. |
| U6 | Yayındaki pencerede yeniden içe alma | MEB yeni sürümle tarih ya da saat değiştirirse her satır için revizyon (`CentralRealigned`) yazılır. İşlem başına tek sürüm artışı olur ve bildirim gider. |
| U7 | Uygulayıcının yeniden çözülmesi | Gece taraması, ders programı değişince merkezi satırın uygulayıcısını yeniden çözer. Satırı "taşındı" yapmaz. Kilitli penceredeki ve tarihi geçmiş satırlara dokunmaz, böylece fiilî uygulayıcı korunur. Sınavın yapıldığı gün bu korumanın dışında kalır: aynı gün yapılan bir program değişikliği uygulayıcıyı hâlâ değiştirebilir. Bu bilinçli olarak kabul edildi. |
| U8 | CSV yükleme | Dönem dışına düşen tarih reddedilmez, uyarıyla yüklenir. Hatalı satırlar takvime girmez. Yükleme sorunlarının ilk 200'ü toplam sayıyla takvimde saklanır ve doğrulayana gösterilir. |
| U9 | Takvimi atma | `Draft` ve `Verified` takvim atılabilir. `Published` ve `Superseded` atılamaz. |
| U10 | Müfredat aktivasyonu | Aynı dersin ortak ve seçmeli MEB satırları snapshot'ta tek satırdır. Seçmeli payı `elective_weekly_hours` ile ayrıca tutulur. Kota ekranı dersi parçalarına ayırarak gösterir, haftalık saat ekranları toplamı gösterir. |

### Sürüm 2 — mazeret sınavı (2026-10-25)

Plan: https://claude.ai/artifact/WYs7fopue7kdNvZEXQPcUk. Faruk 2026-10-25'te on iki kararın hepsini önerildiği gibi onayladı. Kod: oksis-api ve oksis-ui `feature/oyss-surum-2` dalları (dev'e PR ile gelir).

**Kapsam sınırı.** EBA'da sınavın içeriği (soru, kâğıt, asıl/mazeret sınav tipi) zümre başkanınca hazırlanır. Puan ve "G" (girmedi) e-Okul'a girilir. OKSİS mazeretin lojistiğini tutar: kim hak sahibi, kabul mü, hangi gün, kaçıncı ders, hangi derslik, kim gözetmen, girdi mi.

| No | Konu | Karar |
|---|---|---|
| M1 | Yoklama bağı | İsteğe bağlıdır (`Exams:Makeup:RequireAttendanceLink`, öntanım kapalı). Bağ yoksa gerekçe kategorisi zorunludur. Onaylı bir devamsızlık mazereti sınav gününü kapsıyorsa talep kendiliğinden kabul edilir; gerekçe Yoklama'da kalır, `MakeupExam` yalnız kimliği tutar. |
| M2 | Gerekçe | Kategori (Sağlık / Aile / Resmî görev / Diğer) ve "belge kâğıtta görüldü" işareti (isteğe bağlı). Belge yüklenmez. Sağlık kategorisinde serbest not yazılamaz; diğerlerinde not 15–300 karakterdir. |
| M3 | Öğretmenin gördüğü | Ders öğretmeni gerekçe, kategori ve not görmez. Gerekçe yalnız `exams.makeup.manage` ile `attendance.manage` birlikteyken görünür; SuperAdmin hiçbir durumda görmez. Yetki yokken hak sahibi listesinde "izinli" ile "raporlu" tek bir nötr koda iner. |
| M4 | Hak sahibi | Sınav saatinde özürsüz devamsız, izinli ya da raporlu olan öğrenci aday olarak listelenir; kararı idare verir. Aday kalıcı durum değildir, okuma anında yoklamadan türetilir. |
| M5 | Mazeret notu | Öğrencinin mazeret sınavı "girdi" işaretliyse, Notlar'daki 48 saatlik düzeltme penceresi o öğrenci × o sınav için dönem sonuna kadar açılır (`IMakeupMarkGate`). Denetim izinde "mazeret sınavı notunu girdi" yazar. |
| M6 | Mazeret haftası | Okul aralığı pencereye girer (en çok 31 gün, dönem sonunu aşmaz, pencere bitişinden sonra başlar). Kilitli pencerede de girilebilir. |
| M7 | Adlandırma | Tek terim "Mazeret Sınavı". İki tür: okulun mazeret haftası (`SchoolWeek`) ve öğretmenin ek sınavı (`TeacherCatchUp`, arayüzde "Ek sınav"). "Telafi" kelimesi kullanılmaz. Ek sınavın son tarihi, mazeretin bittiği günü izleyen 5. okul günüdür ve dönem sonunu aşmaz. Okul günü: yürürlükteki sezonun dönemi içinde, hafta içi, tatil değil, zil çizelgesinde kapalı değil. |
| M8 | Yeniden planlama | Dönem sonuna kadar yeniden planlanabilir. Girmeyen öğrenci için yeni geçerli mazeret yeni bir ek sınav kaydı açar (önceki kayda bağlı). |
| M9 | Ülke geneli sınav | Ülke geneli sınav için de mazeret açılır, okul planlar. Katalog şeması değişmez. |
| M10 | Öğretmen mobili | Mazeret oturumları yalnız öğrenci sayısıyla görünür, öğrenci adı yoktur. |
| M11 | Push sızıntısı | Yoklama push'u artık öğrenci adı, şube ve durum taşımaz. Serbest gerekçe ya da not taşıyan bütün türlerde push gövdesi geneldir; ayrıntı uygulama içinde kalır. |
| M12 | Saklama | Serbest metin gerekçe (not, karar notu, iptal gerekçesi) terminal durumdan 180 gün sonra temizlenir (`Exams:Makeup:ReasonRetentionDays`). Kilitli pencerede bayat kalan açık kayıtlar da aynı süreyle temizlenir. Süre taslaktır, hukuk teyidi bekler. |

**Kural kodları** ([[0018-sinav-kurallari-denetleyicide]] eki, mimar onayı bekler):

- `EX-H14` (sert): Geçmiş gün, okul günü olmayan gün, mazeret haftası dışı tarih, ek sınavda mazeret bitişinden önce ya da son tarihten sonra, dönem sonundan sonra.
- `EX-H15` (sert): Öğrencinin kendi şubesinin o saatte planlı sınavı var.
- `EX-S11` (uyarı): Günlük sınav yükü okul sınırını aşıyor.
- `EX-S12` (uyarı): Gözetmenin o saatte kendi dersi var.
- `EX-S13` (uyarı): Derslik kapasitesi aşılıyor ya da duyuru payı bir günden az.

Mevcut `EX-H05`, `H06` ve `H11` mazeret satırlarına da uygulanır. Yumuşak kural geçişi gerekçe istemez ve saklanmaz.

**Model ve uçlar.**

- `MakeupExam` öğrenci başına satırdır; oturum agregatı yoktur. Aynı gün, saat ve derslikteki satırları istemci okuma anında gruplar.
- Durumlar: `Applied → Accepted | Rejected → Scheduled → Held | NoShow`; her açık durumdan `Cancelled`. Bir öğrenci × bir asıl sınav için aynı anda tek açık kayıt olur.
- Yeni izin `exams.makeup.manage` yalnız Okul Müdürü ve Müdür Yardımcısı rollerindedir.
- Gerekçe okuma, karar, iptal ve mazeret bitiş günü değişikliği metin içermeyen, yalnız eklenebilir bir denetim izine yazılır (`MakeupAuditEntry`).
- Bildirimde yeni tür açılmadı: planlama, taşıma, iptal ve "yarın sınav" var olan türlerle öğrenciye ve veliye gider. Gözetmene hücre başına tek bildirim gider.

**MEB yeni sürüm sinyali.** Platformda daha yüksek sürümlü bir takvim yayımlanınca, okulun içe aldığı en yüksek sürümle karşılaştırılır. Fark varsa pencere listesinde `centralUpdates` ile görünür (`GET exams/windows/{id}/central-updates` ayrıntıyı verir). Gece taraması yöneticiye "takvim yayımlandı" türüyle tek bildirim gönderir. Yeniden aktarım, yeni sürümden bir satır alındığında sinyali söndürür.

**KVKK sertleştirmeleri (Sürüm 2 ile gelen).**

- Log maskeleme bileşeni ilk kez devreye bağlandı. Kimlik bilgisi anahtarları alt dizeyle, serbest metin alanları tam adla maskelenir.
- Mazeret belgesinin görünen adı ve indirme adı sunucuda üretilir.
- Gerekçe dönen uçlar `Cache-Control: no-store` döner.

**Açık sorular (Faruk).**

- Saklama süresi ve "Sağlık" kategorisinin süre sonunda kalması hukuk teyidi bekler.
- Yoklama mazeretlerinin serbest metinleri için anonimleştirme işi.
- Bilgi kısıtlı velinin mazeret bildirimi alması.
- E-posta kanalında "Raporlu" metni.
- Eski mazeret satırlarındaki özgün belge adlarının geri doldurulması.
- MEB yeni sürümde kalkan dersin yeniden aktarımdaki davranışı.
- İptalden sonra öğrencinin yeniden aday olması (bugünkü davranış).

## Agregat sınırları

| Agregat | Kök | Kiracı | İçerik | Bağ |
|---|---|---|---|---|
| Merkezi takvim | `CentralExamCalendar` | Yok (platform, `master`) | Akademik yıl, kaynak belge, sürüm, durum (`Draft → Verified`, ardından `Superseded`), yükleyen, doğrulayan; satırlar çocuktur | — |
| Merkezi takvim satırı | `CentralExamCalendarEntry` (takvimin çocuğu) | Yok | Sınıf seviyesi, ders, dönem, yazılı sırası (MEB adıyla), tarih, öntanımlı ders saati, kapsam (OYSS / ülke geneli) | Ders ve seviye master kimlikleri |
| Planlı sınav (var olan) | `ScheduledExam` | `SchoolId` | + `CentralEntryId`, `CentralDate`, `CentralPeriod` | Katalog satırına yalnız kimlikle; navigasyon yok |
| Mazeret sınavı (Sürüm 2) | `MakeupExam` | `SchoolId` | Öğrenci, asıl sınav, plan tarihi ve saati, tür (okul mazeret haftası / öğretmenin ek sınavı), durum | `ScheduledExam`'a ve yoklama kaydına kimlikle; not Grades'te |

Kilit kuralı `ScheduledExam`'ın içinde yaşar, çünkü tek satıra bakar. S10 ve yumuşama kural denetleyicisinde yaşar, çünkü birden çok satıra bakar ([[0018-sinav-kurallari-denetleyicide]]).

## Şema taslağı (örnektir, uygulama kodu değildir)

**M1 — `master.central_exam_calendars`**

| Alan | Tür | Not |
|---|---|---|
| `id` | uniqueidentifier | PK |
| `academic_year_code` | nvarchar(9) | ör. `2026-2027` |
| `version` | int | aynı yıl içinde 1'den artar |
| `source_reference` | nvarchar(300) | MEB yazısı / duyuru bağlantısı |
| `payload_sha256` | char(64) | CSV parmak izi; idempotentlik |
| `status` | nvarchar(30) | `Draft`, `Verified`, `Superseded` |
| `uploaded_by`, `uploaded_at` | | yükleyen hesap |
| `verified_by`, `verified_at` | | boş olabilir; `verified_by <> uploaded_by` denetimi |
| `row_version` | rowversion | |
| Tekillik | | (`academic_year_code`, `version`); (`academic_year_code`, `payload_sha256`) |

**M1 — `master.central_exam_calendar_entries`**

| Alan | Tür | Not |
|---|---|---|
| `id` | uniqueidentifier | PK |
| `calendar_id` | uniqueidentifier | FK, `Restrict` |
| `grade_level_id`, `subject_id` | uniqueidentifier | master kimlikleri |
| `term_ordinal` | tinyint | 1 / 2 |
| `written_ordinal` | tinyint | MEB'in adlandırdığı yazılı (1. yazılı …) |
| `scope` | nvarchar(20) | `Oyss`, `National` |
| `exam_date` | date | |
| `default_period` | tinyint | öntanımlı 2 (çift sınav gününde 2 ve 4) |
| Tekillik | | (`calendar_id`, `grade_level_id`, `subject_id`, `term_ordinal`, `written_ordinal`) |

**M2 — `academic.scheduled_exams` ek alanları**

| Alan | Tür | Not |
|---|---|---|
| `central_entry_id` | uniqueidentifier, boş olabilir | şemalar arası FK yok; kimlik bağı |
| `central_date` | date, boş olabilir | içe alma anındaki MEB tarihi |
| `central_period` | tinyint, boş olabilir | içe alma anındaki MEB saati |
| Dizin | | (`school_id`, `central_entry_id`), `central_entry_id IS NOT NULL` süzgeçli |

EF Core yapılandırması var olan kalıbı izler: katalog `ToMasterTable` ile, kiracısız ve `CurriculumImportRun` gibi parmak izi tekilliğiyle kurulur. Ek alanlar `ScheduledExamConfiguration`'a eklenir. `ExamRevisionKind.CentralPeriodChanged = 6` sona eklenir. Değer int olarak yazıldığı için göç gerekmez.

## Değerlendirme

| Ölçüt | A — okul elle | **B — katalog + içe alma** | C — canlı okuma | D — ayrı tür/agregat |
|---|---|---|---|---|
| Tenant izolasyonu | Doğal (her şey `SchoolId`'li) | Katalog kiracısızdır ve yalnız okunur; okul satırı `SchoolId` + global süzgeçle kalır. İçe alma yalnız çağıranın okuluna yazar. Katalog okuyan sorgu `IgnoreQueryFilters` gerektirmez, çünkü `master` tabloları kiracı süzgecine girmez | Her okul okuması kiracısız tabloyla birleşir; süzgeç birleştirmenin bir tarafında kalır, gözden kaçma riski yüksek | B ile aynı, ama iki ayrı okul tablosu |
| Modüller arası bağımlılık | Yok | Exams, kataloğu yalnız bir okuma soyutlamasıyla okur (ör. `ICentralExamCalendarReader`). Ders programına mevcut sayaç soyutlaması üzerinden, Grades'e hiç bağımlı değildir | Exams'in her okuma yolu kataloğa bağımlı olur | Rule inspector, pano ve not beslemesi iki kaynağı bilmek zorunda kalır |
| Kaynağa bağlılık ve kilit | Yok | Var (kimlik + MEB değerinin kopyası) | Var, ama revizyon izi yok | Var |
| Göç maliyeti | Yok | M1: 2 yeni tablo; M2: 3 boş olabilir kolon + 1 süzgeçli dizin; veri göçü yok | 1 tablo, ama okuma yollarının tamamı yeniden yazılır | 2 tablo + yeni okul tablosu + kural/pano çatallanması |
| Yayınlanmış takvime etki | — | Katalog düzeltmesi okulu kendiliğinden değiştirmez; okul yeni sürümü önizleyip alır | Sessiz değişiklik | — |

## Sonuçları

- Merkezi satırda öğretmen gün seçemez. Öğretmen ekranı OYSS satırını salt okunur gösterir; yalnız saat değişikliğini idare yapar.
- Kural listesi büyür: `EX-S10` merkezi satıra özgüdür, merkezi olmayan satırda hiç üretilmez. Okul sınırı (`EX-H01`) MEB takviminden bir günü 2'den fazla sınava götürürse sert değil uyarıdır; çünkü günü okul seçmedi.
- Uygulayan öğretmen, MEB saatinde o şubede derse giren öğretmendir. Hücre boşsa dersin öğretmenidir. Bu kural `ScheduledExam`'ın var olan sahip/uygulayan ayrımına oturur (K-8); yeni alan gerekmez.
- Sınav Takvimi modülünün "okul dışı merkezi sınavlar kapsam dışı" maddesi bu kararla daralır: OYSS lojistiği kapsamdadır, içerik ve puan dışarıdadır.
- Mobilde OYSS satırı rozetle görünür; "puanlar e-Okul'da" şeridi gösterilir. OKSİS e-Okul'a bağlanmaz; yalnız bilgi verir.
- Tek platform hesabıyla takvim doğrulanamaz; dev ortamında ikinci bir platform hesabı gerekir ([[0022-mufredat-yayimi-iki-kisi-kurali]] ile aynı bilinçli maliyet).
- Mazeret sınavı Sürüm 2'ye kadar OKSİS'te izlenmez; 23–27 Kasım mazeret haftası Sürüm 2 takvimiyle yetişmelidir (2–18 Kasım).
- Çevrimdışı çalışma yoktur (K3). Sınav günü bağlantı yoksa mobil "bağlantı yok" ekranı gösterir; canlıya geçiş öncesi listesine yazılır.

## Geri alma maliyeti

**Orta-düşük.** M1 ve M2 eklemelidir; geri almak iki tablonun ve üç boş kolonun düşürülmesidir, mevcut veri etkilenmez. Pahalı olan davranıştır. İçe alınmış ve yayınlanmış merkezi satırlar geri alındığında sıradan satıra döner, kilit kalkar ve S10 kaybolur. Yayınlanmış revizyon kayıtlarındaki `CentralPeriodChanged` (6) değeri kalır; enum'dan silinemez, yalnız kullanılmaz olur. A'ya dönmek okullara elle giriş yükü getirir. D'ye geçmek kural denetleyicisini çatallamak demektir ve en pahalı yoldur.

## Doğrulama bekleyenler

İki madde 2026-10-25'te birincil kaynaktan kapandı:

- **MEB ders saati kuralı — doğrulandı.** Günde tek sınavda 2. ders, iki sınavda 2. ve 4. ders, süre 40 dakika; okul müdürlüğü ihtiyaç hâlinde farklı ders saati planlayabilir. Kodun `default_period` eşlemesi değişmedi.
- **Ülke geneli sınavın adlandırması — doğrulandı.** MEB metni ülke geneli sınavı "1. dönem 1. sınavı" diye anar; katalog satırının `written_ordinal` değeri 1'dir.

## Kaynaklar

- OYSS — OKSİS Entegrasyon Raporu: https://claude.ai/artifact/LgBJ2e1L648YdSNcJAk9cX
- OYSS Geçiş Planı: https://claude.ai/artifact/AatFzUknJCw1qUB5pNmrPM — vault karşılığı [[oyss-gecis-plani]]

## İlgili

- [[Sınav Takvimi]]
- [[Planlanmış Sınav]]
- [[Sınav Penceresi]]
- [[0018-sinav-kurallari-denetleyicide]]
- [[0022-mufredat-yayimi-iki-kisi-kurali]]
- [[0021-aktif-sezon-mufredati-snapshottan-okur]] — değişmez sürüm ilkesi
- [[Notlar]] — mazeret sınavı notu burada girilir
