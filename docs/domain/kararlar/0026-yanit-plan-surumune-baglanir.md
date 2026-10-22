---
tags: [decision, domain/parent-meetings, domain/messaging]
date: 2026-10-15
status: accepted
---

# 0026 — Veli yanıtı şube toplantısının plan sürümüne bağlanır; sıfırlama fiziksel değil sürüm artışıdır, yanıt geçmişi yalnız eklenir

## Bağlam

K-9'a göre şube toplantısının tarihi ya da saati değişince veli yanıtları sıfırlanır, eski yanıtlar geçmişte kalır ve velilere yeniden yanıt vermeleri söylenir. K-8 ise idarenin veli adına yanıt girebilmesini ve "kim, ne zaman, kimin adına" bilgisinin kaybolmamasını ister.

Bildirim tarafında da aynı soru vardı. Ertelemeden önce kuyruğa girmiş bir hatırlatma ya da "saat değişti" bildirimi gönderildiği anda eskimiş olabilir (BR-PM-35). Yeni tarih için hatırlatmanın yeniden gidebilmesi de gerekir; ödevdeki `TB-111` dersi, damganın "geçerli tarih için haber gitti" anlamına gelmesi gerektiğini gösterdi.

## Seçenekler

- **A — Yerinde güncelleme + toplu sıfırlama:** tek yanıt satırı tutulur; tarih değişince bütün satırlar "yanıt yok"a çekilir; değişiklikler ayrı bir geçmiş tablosuna yazılır.
- **B — Plan sürümü (önerilen):** şube toplantısı bir `ScheduleRevision` sayacı taşır; yanıt satırı `(şube, veli, sürüm)` tekliğiyle tutulur. Güncel yanıt, güncel sürümdeki satırdır; satır yoksa "yanıt yok"tur. Ayrıca yalnız eklenen bir yanıt geçmişi tutulur.
- **C — Yalnız olay kaydı:** güncel yanıt geçmiş tablosundan türetilir, ayrı bir yanıt satırı tutulmaz.

## Karar

**B önerilir.** Yalnız tarih ya da saat değişikliği sürümü artırır. Yer değişikliği ayrı bir `LocationRevision` artırır ve yanıtları sıfırlamaz. Sürüm artınca güncel yanıtı olan her veli için geçmişe bir "plana bağlı sıfırlama" satırı yazılır. Böylece geçmiş kendi başına okunabilir kalır ve sıfırlama sessiz olmaz. Bildirim olay kimlikleri sürümü içerir. Hatırlatma damgaları da "gönderildiği sürüm" olarak tutulur.

## Değerlendirme

| Ölçüt | A | B | C |
|---|---|---|---|
| Sıfırlamanın maliyeti | Toplu `UPDATE`; yarışta yarım sıfırlanmış şube kalabilir | Tek sayaç artışı + sıfırlanan veli başına bir geçmiş satırı | Tek sayaç |
| Etki önizlemesi ile uygulamanın uyuşması (BR-PM-17) | İki ayrı sorgu | Aynı hesap çekirdeği | Aynı |
| Rapor ve liste sorgusu | Basit | `revision = schedule_revision` koşulu (indeksli) | Geçmişten son durumu türetmek gerekir; pahalı |
| Bildirim tazeliği | Ek alan gerekir | Olay kimliğinden sürüm yeniden hesaplanabilir; `Notification` tablosuna kolon eklenmez | B ile aynı |
| Denetim ("kim, kimin adına") | Geçmiş tablosunda | Geçmiş tablosunda (append-only, güncelleme/silme yolu yok) | Doğal |
| Tenant izolasyonu | `SchoolId` + filtre | Aynı; geçmiş `PermanentTenantEntity` (soft delete yok) | Aynı |
| Modüller arası bağımlılık | — | Notifications modülü sürümü bilmez; tazeliği ParentMeetings'in kaydettiği `INotificationSourceFreshness` portu söyler | — |
| Göç maliyeti | 2 tablo | 2 tablo + şubede 2 sayaç ve 3 damga kolonu | 1 tablo |

## Sonuçları

- Veli yanıtı Yayınlandı durumunda ve başlangıç anından önce değiştirilebilir. Değer değişmediyse geçmiş yazılmaz; çift dokunuş iz bırakmaz.
- Katılamama nedeni geçmişte de yalnız rehber öğretmen ve idareye görünür.
- Dispatch ve ertelenmiş push gönderilmeden önce kaynak tazeliği sorulur. Örneğin tarih değişikliğinin hemen ardından iptal edilen bir toplantıda kuyruktaki "saat değişti" bildirimi atılır, yalnız iptal bildirimi gider.
- Sabah hatırlatması ve "yanıt bekleniyor" çağrısı sürüm başına bir kez gider. Erteleme yeni sürüm doğurduğu için hatırlatma yeniden gidebilir.
- Geçmiş tablosuna `Update`/`Remove` çağrısı mimari testle yasaklanır.

## Geri alma maliyeti

**Düşük–orta.** A'ya dönmek için güncel sürüm satırları tek satıra indirilir, eski sürüm satırları silinir ya da arşivlenir (tek göç). Olay kimliği ve tazelik denetimi sürüme dayandığı için bildirim tarafının da birlikte değişmesi gerekir. Geçmiş tablosu her iki modelde aynı kalır.

## Uygulama durumu (2026-10-22, oksis-api 02fbcdc1)

Kodda gerçekleşti; `accepted` durumu kodla uyumlu.

- Yanıt satırı şube × veli × plan sürümüyle tekildir; güncel yanıt güncel sürümdeki satırdır, "yanıt yok" satır yokluğudur.
- Sürüm artışında güncel yanıtı olan her veli için geçmişe "plana bağlı sıfırlama" satırı yazılır; değeri değişmeyen yanıt geçmiş yazmaz.
- Hatırlatma damgaları "gönderildiği sürüm" olarak tutulur; bildirim tazeliği bildirim modülüne kaydedilen tazelik portuyla sorulur.
- Yanıt geçmişi ve işlem kaydı için güncelleme/silme çağrısı mimari testle yasaktır.
- **İnceltme:** karar "tarih ya da saat" diyor; kodda sürümü yalnız **tarih ve başlangıç saati** artırır. Yalnız bitiş saati değişikliği yanıtları korur ve yer değişikliğiyle aynı bilgi sürümünü artırır (ayrı işlem kaydı türüyle).
- **Ek (kararda yok):** başlamış ama fiili verisi olmayan toplantı ertelenince yayına geri döner; davetli listesi yeniden canlanır.

## İlgili

- [[Veli Toplantıları]]
- [[0025-sube-toplantisi-ayri-agregat]]
- [[0012-bildirim-ilkeleri]]
