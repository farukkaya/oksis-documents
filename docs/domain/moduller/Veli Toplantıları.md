---
aliases: [Parent Meetings, api/v1/parent-meetings, Veli Toplantısı Modülü]
tags: [domain/parent-meetings, module]
status: planned
last-synced: 2026-10-15 (kod yok — teknik analiz aşaması)
---

# Veli Toplantıları

> **Durum:** planlandı. Kod yok (`feature/veli-toplantilari`, 2026-10-15). Davranışın kaynakları: `ihtiyac-analizleri/veli-toplantisi/…İhtiyaç Analizi (Final).md` (K-1…K-16, BR-PM-01…42) ve `teknik-analizler/veli-toplantisi/Veli Toplantısı Modülü · Teknik Analiz (V1).md`. Backend tamamlanınca bu not `domain-map` ile koddan yeniden üretilir.

## Ne yapar

İdare (okul yöneticisi + müdür yardımcısı) okul geneli ya da seçili şubeler için **tek bir organizasyon** altında veli toplantısı planlar. Her şubenin toplantısı **ayrı yaşar**: kendi tarihi, saati, yeri, durumu, iptali ve ertelemesi vardır. Yayında şubenin davetli velileri (bilgi görme yetkisi olan, iptal edilmemiş ve süresi dolmamış veli-öğrenci ilişkileri) bildirim alır. Veli "Katılacağım / Katılamayacağım" der ve Başladı'ya kadar fikrini değiştirebilir. İdare velinin adına yanıt girebilir. Toplantı günü şubenin **güncel** sınıf rehber öğretmeni (ya da toplantıyı yürüten idareci) beyandan bağımsız olarak fiili katılımı işaretler ve öğrenci başına görüşme notu yazar. Not veliye ve öğrenciye asla gösterilmez; her okuması kayda geçer ve saklama süresi dolunca metni silinir. İdare beyan ile fiili katılımı iki ayrı eksende raporlar.

## Kullandığı kavramlar

- [[Veli Toplantısı]] (organizasyon) — çatı kayıt; `Draft | Published`
- [[Şube Toplantısı]] — `Draft → Published → Started → Completed`, `Cancelled`; plan sürümü ([[0026-yanit-plan-surumune-baglanir]])
- [[Davetli Veli]] — öğrenci × veli; Başladı anında dondurulur
- [[Katılım Yanıtı]] / [[Yanıt Geçmişi]] — veli × plan sürümü; geçmişe yalnız eklenir
- [[Fiili Katılım]] — veli başına "Geldi"; işaretsiz olanlar tamamlanınca "Gelmedi" sayılır
- [[Görüşme Notu]] — öğrenci başına; ayrı agregat ([[0025-sube-toplantisi-ayri-agregat]]); okuma denetimi; süreli imha
- [[Şube]] / [[Sezon]] / [[Kişi]] / [[Veli-Öğrenci İlişkisi]] / [[Okul Ayarları]] / [[Bildirim]]

## Ana kurallar (özet)

- Geçmiş bir **an** planlanamaz; bugünün ileri bir saati olabilir.
- Tarih ya da saat değişince yanıtlar sıfırlanır ve velilere bildirim gider. Yalnız yer değişirse yanıtlar sıfırlanmaz.
- Sorumlu kayıtta dondurulmaz; her işlemde şubenin güncel rehber öğretmeni yetkilidir. Başka öğretmen atanmaz.
- Otomatik kapanış başlangıç gününden 3 gün sonradır. Tamamlandıktan sonra 7 gün düzenleme penceresi vardır.
- Notu okuyabilenler: güncel rehber öğretmen, idare ve notu yazan öğretmen (okuldan ayrılınca bu erişim biter).
- Bildirimler: yayın, öğretmene bilgi, değişiklik, iptal, sabah hatırlatması, başlangıç (okul ayarı) ve 2 gün önce yanıtsızlara çağrı. E-posta yalnız yayın, değişiklik ve iptalde gider ve varsayılanı açıktır ([[0027-eposta-varsayilani-tur-bazli]]). Hesabı olmayan veli için yalnız SMS niyeti kaydedilir; SMS gönderilmez.

## Kararlar

- [[0025-sube-toplantisi-ayri-agregat]] (proposed)
- [[0026-yanit-plan-surumune-baglanir]] (proposed)
- [[0027-eposta-varsayilani-tur-bazli]] (proposed)

## Bağımlılıklar

Okur: Sınıflar ve Şubeler (`ClassRoom.HomeroomTeacherId`, mevcut), Kullanıcılar (kişi, veli-öğrenci ilişkisi, rol ataması), Sezon Yönetimi (arşiv olayı), Okul Yönetimi (ayarlar, saat dilimi). Yazar: Bildirimler (soyutlamalar üzerinden). Hiçbir modül bu modüle bağımlı değildir.
