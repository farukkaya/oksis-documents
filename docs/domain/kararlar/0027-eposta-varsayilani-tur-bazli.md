---
tags: [decision, domain/messaging]
date: 2026-10-15
status: accepted
---

# 0027 — E-posta kanalı alıcı tercihiyle açılır; "seçilebilir" ve "varsayılan açık" tür bazında ayrı bayraklardır (E-44)

## Bağlam

2026-10-09'dan beri e-posta bildirim kanalı bütün olaylarda kapalı (`EmailNotificationChannel.RecipientCanChooseEmail => false`). Teslim kuralı "olay kataloğu ∩ alıcı tercihi"dir (K-34), ama alıcının e-posta tercihi yoktu. Veli toplantısı modülü (K-13) e-postayı açıyor ve bunu E-44 ile bütün türler için alıcı seçimi olarak tanımlıyor. S-6 kararı: tercih satırı yokken varsayılan **tür bazlıdır**. Veli toplantısının yayın, değişiklik ve iptal bildirimlerinde açık, diğer türlerde kapalıdır.

Kodda iki engel vardı:

1. `NotificationEventType.DefaultEmailEnabled` bugün yedi satırda `true`: ATT_THRESHOLD, GRADE_PUBLISHED, REPORT_CARD_PUBLISHED, PAYMENT_REMINDER, PAYMENT_RECEIVED, ANNOUNCEMENT_URGENT, HOMEWORK_MISSING. Bu değerler kaldırılmış okul matrisinin kalıntısıdır ve "teslimde kullanılmaz" diye belgelenmiştir. Kapı doğrudan bu alana bağlansaydı, E-44 açıldığı gün bu türler tercih soran olmadan e-posta göndermeye başlardı (güvenlik ön incelemesi K2).
2. Tek bir alan iki ayrı soruyu taşıyamaz: "alıcı bu türde e-postayı seçebilir mi?" ve "seçmediyse açık mı?". Push'ta K-34 "kapsamda ama varsayılanı kapalı" hâlini yasaklar. E-postada ise bu hâl istenen davranıştır: not bildirimi e-postayla alınabilmeli, ama kendiliğinden gelmemelidir.

## Seçenekler

- **A — Mevcut alanı varsayılan say, eski değerleri koru:** en ucuz yol, ama yedi türü sessizce açar. Reddedildi.
- **B — Tek alan, eski değerleri sıfırla:** `DefaultEmailEnabled` "destekler ve varsayılan açık" anlamına gelir. Bu durumda varsayılanı kapalı bir türde alıcı e-postayı seçemez ve E-44'ün "bütün türler" hükmü karşılanmaz.
- **C — İki bayrak (önerilen):** yeni `SupportsEmail` (alıcı seçebilir mi; `SupportsSms` ile simetrik) eklenir. `DefaultEmailEnabled` gerçek anlamında kullanılır (satır yokken varsayılan). Göç bütün mevcut satırlarda varsayılanı sıfırlar ve yalnız veli toplantısının üç türünde açar.
- **D — Varsayılan kodda sabit bir listede:** katalog ile kod iki ayrı gerçek hâline gelir (`TB-44` dersi).

## Karar

**C önerilir.**

- `NotificationEventType.SupportsEmail` alanı eklenir. `DefaultEmailEnabled` bütün mevcut satırlarda `false` yapılır. Yalnız `PARENT_MEETING_PUBLISHED`, `PARENT_MEETING_CHANGED` ve `PARENT_MEETING_CANCELLED` için `SupportsEmail = DefaultEmailEnabled = true` olur.
- `NotificationPreference.EmailEnabled` eklenir ve boş bırakılabilir (`bool?`). `null`, "katalog varsayılanı" demektir. Mevcut push satırlarına dokunulmaz.
- Kanal kapıları, ucuzdan pahalıya:
  1. Yapılandırma: sağlayıcı izin listesinde olmalı. Üretimde liste dışı bir sağlayıcı başlangıç hatası verir.
  2. Kapsam: olay matris eşlemesi; push eşlemesi değil.
  3. Katalog: `SupportsEmail`.
  4. Tercih: `EmailEnabled ?? DefaultEmailEnabled`.
  5. Adres: alıcının adresi olmalı.
- E-posta yalnız bilgilendirmedir: katılım düğmesi ya da bağlantısı yoktur (K-13). İçerik okul, şube, tarih, saat ve yerdir. Neden, gerekçe ve not hiçbir zaman e-postaya girmez.

## Değerlendirme

| Ölçüt | A | B | C | D |
|---|---|---|---|---|
| Mevcut türleri sessizce açar mı | **Evet** | Hayır | Hayır | Hayır |
| "Seçilebilir ama varsayılanı kapalı" | — | Yok | Var | Var |
| Tek gerçek (katalog) | Evet | Evet | Evet | Hayır |
| Tenant izolasyonu | Tercih `TenantEntity` (okul × hesap) — dördünde aynı | | | |
| Modüller arası bağımlılık | Yalnız Notifications modülü; üretici modüller kanal bilmez (`NoChannelDecisionInCommandsTests` korunur) | | | |
| Göç maliyeti | yok | 1 veri göçü | 2 kolon + 1 veri göçü | 1 kolon |

## Sonuçları

- E-44 açıldığında hiçbir mevcut olay kendiliğinden e-posta göndermez. Alıcı bir türü açarsa o türde e-posta gider.
- Tercih ekranı bütün rollere e-posta sütunu ekler. Sütun yalnız `SupportsEmail` türlerinde etkindir. E-posta adresi olmayan hesapta sütun pasiftir.
- [[0012-bildirim-ilkeleri]]'nin "Uygulama durumu" notu e-postanın kapsam listesinde çalıştığını söylüyor. Bu not kodla çelişiyor ve bu ADR kabul edilince düzeltilmelidir.
- `EmailNotificationChannel` doc'undaki "tercihsiz e-posta yasak" ilkesi değişir: tercihsiz e-posta yalnız katalogda varsayılanı açık türlerde gider. Bu türler bugün yalnız veli toplantısının üç bilgilendirme türüdür.
- Varsayılan-açık e-postanın KVKK dayanağı (aydınlatma metni, vazgeçme yolu) hukuk teyidine bağlıdır. Teyit gelene kadar E-44 üretime alınmaz.

## Geri alma maliyeti

**Düşük.** Kapı tek sınıftadır. Geri almak için `RecipientCanChooseEmail` kapısı yeniden `false` yapılır. Kolonlar zararsız kalır. Veri göçünün `Down` adımı eski `DefaultEmailEnabled` değerlerini sabit listeden geri yazar. Alıcıların yazdığı e-posta tercihleri korunur.

## Uygulama durumu (2026-10-22, oksis-api 02fbcdc1)

Kodda gerçekleşti; `accepted` durumu kodla uyumlu.

- Bildirim türü kataloğunda "seçilebilir" ve "varsayılan açık" iki ayrı bayraktır; varsayılan yalnız veli toplantısının yayın, değişiklik ve iptal türlerinde açıktır. Eski yedi türün varsayılanı sıfırlandı.
- Alıcı tercihi boş bırakılabilir; boş, katalog varsayılanı demektir.
- Kanal kapıları kararda sayılan sırayla (yapılandırma, kapsam, katalog, tercih, adres) uygulanır; kapsam push eşlemesine değil olay matrisi eşlemesine bağlıdır.
- Veli toplantısının diğer dört türünde e-posta seçilemez.
- **Sapma:** karar "üretimde liste dışı bir sağlayıcı başlangıç hatası verir" diyor; kodda liste dışı yapılandırma uyarı loglar ve kanalı kapalı tutar, uygulama başlar. Başlangıcı yalnız bozuk izin listesi ve **üretimde** izin listesindeki tüketici e-posta sağlayıcısı durdurur.
- **Doğrulanamayan:** KVKK hukuk teyidi ve "teyit gelene kadar üretime alınmaz" koşulu koddan izlenemez; yapılandırma kapısı varsayılan kapalıdır.

## İlgili

- [[0012-bildirim-ilkeleri]]
- [[Bildirimler]]
- [[Veli Toplantıları]]
