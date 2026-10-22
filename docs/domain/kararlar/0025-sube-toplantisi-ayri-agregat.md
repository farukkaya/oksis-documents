---
tags: [decision, domain/academic, domain/parent-meetings]
date: 2026-10-15
status: accepted
---

# 0025 — Şube toplantısı organizasyondan bağımsız bir agregattır; organizasyon yalnız çatıdır, görüşme notu ayrı agregattır

## Bağlam

[[Veli Toplantıları]] modülünde idare okul geneli ya da seçili şubeler için tek seferde bir toplantı planlar (K-2). Ama her şubenin toplantısı ayrı yaşar: kendi tarihi, saati, yeri, durumu var. Tek bir şube iptal edilebilir, ertelenebilir ya da tamamlanabilir; diğerleri bundan etkilenmez. Toplantı günü öğretmen her veli için "Geldi" işaretler ve öğrenci başına görüşme notu yazar. Notlar özel nitelikli kişisel veri içerebilir; bu yüzden okuma dahil denetlenir ve süre dolunca metinleri silinir (K-11, K-12).

Kayıtların hangi agregat sınırıyla tutulacağı kararlaştırılmalıydı. Bu sınır eşzamanlılığı, otomatik işlerin ne yükleyeceğini ve not metninin hangi okuma yollarına girebileceğini belirler.

## Seçenekler

- **A — Tek agregat:** organizasyon kök olur, şubeler, davetliler, yanıtlar ve notlar onun çocuklarıdır.
- **B — Organizasyon çatı, şube toplantısı ayrı kök, not ayrı kök (önerilen):** organizasyon yalnız başlık, açıklama, varsayılanlar ve taslak/yayın durumunu taşır. Şube toplantısı kendi kökü olur; davetli, yanıt ve fiili katılım onun çocuklarıdır. Görüşme notu ayrı bir köktür.
- **C — Organizasyon yok:** şube toplantıları ortak bir grup anahtarıyla bağlanır (ödevin çoklu şube kalıbı, [[0003-coklu-sube-ayri-odev-kaydi]]).

## Karar

**B önerilir.** Organizasyon (`ParentMeeting`) ile şube toplantısı (`ParentMeetingSection`) ayrı agregatlardır ve birbirine yalnız kimlikle bağlanır. Görüşme notu (`ParentMeetingNote`) kendi denetim ve sürüm kayıtlarıyla ayrı bir agregattır. Notu şube toplantısını yükleyen hiçbir yol getiremez.

## Değerlendirme

| Ölçüt | A — tek agregat | B — önerilen | C — organizasyonsuz |
|---|---|---|---|
| Eşzamanlılık | 40 şubenin bütün yazmaları tek `RowVersion`'da yarışır; toplantı akşamı "Geldi" işaretleri birbirini 409'a düşürür | Şube başına sürüm; yalnız aynı şubeye yazanlar yarışır | B ile aynı |
| Otomatik işler (başlat, kapat, hatırlat) | Her geçişte bütün organizasyon yüklenir | Yalnız ilgili şube yüklenir | B ile aynı |
| Not gizliliği | Kökü yükleyen her sorgu notu yükleyebilir | Not ayrı kök; liste, rapor ve bildirim yolları not tablosuna hiç dokunmaz | Not için B ile aynı |
| "Okul geneli toplantı" kavramı (liste, rapor, sonradan şube ekleme) | Var | Var (çatı kayıt) | Yok; başlık ve açıklama her şubede kopyalanır ve ayrışabilir |
| Tenant izolasyonu | Kök üzerinden | Her tabloda `SchoolId` + global filtre + açık yüklem; şube satırları sezonu denormalize taşır | B ile aynı |
| Modüller arası bağımlılık | Yalnız kimlik | Yalnız kimlik; şube → `ClassRoomId`, sorumlu → güncel `HomeroomTeacherId` (dondurulmaz) | Aynı |
| Göç maliyeti | 1 kök tablosu + çocuklar | 3 kök tablosu (+ çocuklar); ek join yok, çünkü şube satırı sezonu taşır | 2 kök tablosu |

## Sonuçları

- Organizasyonun durumu yalnız `Draft`/`Published`'dır. "Kaç şube tamamlandı/iptal" bilgisi şubelerden türetilir, saklanmaz.
- Organizasyonu yayınlamak, onun bütün taslak şubelerini tek işlemde yayınlayan bir uygulama servisidir; agregatlar arası tutarlılık bu işlemle sağlanır.
- Yanıtlar şubenin çocuğudur. Yüksek yazma hacmine karşı handler kökü filtreli `Include` ile yükler (yalnız o velinin satırları).
- Erteleme kuralı "fiili katılım ve not girilmemişse" der. Not ayrı kök olduğu için bu bilgi domain'e handler'ın sorgusundan parametre olarak gelir.
- Not okuma kapısı ayrı yazılır: rehber öğretmen, idare ve notu yazan (okulda aktif) öğretmen. Bkz. teknik analiz §4.2.

## Geri alma maliyeti

**Orta.** A'ya geçmek göç gerektirmez (tablolar aynı kalır), ama handler'ları ve eşzamanlılık sözleşmesini yeniden yazmayı gerektirir. C'ye geçmek organizasyon tablosunun verisini şubelere kopyalayan bir göç ve rapor/liste uçlarının yeniden yazılmasını gerektirir. Notu şube agregatına katmak teknik olarak ucuzdur, ama gizlilik sınırını kaldırdığı için güvenlik incelemesinden yeniden geçmelidir.

## Uygulama durumu (2026-10-22, oksis-api 02fbcdc1)

Kodda gerçekleşti; `accepted` durumu kodla uyumlu.

- Organizasyon (`ParentMeeting`), şube toplantısı (`ParentMeetingSection`) ve görüşme notu (`ParentMeetingNote`) üç ayrı köktür; organizasyondan şubeye navigasyon yoktur. Davetli, yanıt ve fiili katılım şubenin çocuklarıdır.
- Organizasyon durumu yalnız `Draft`/`Published`'dır; tamamlandı/iptal özeti şubelerden türetilir.
- Yanıt yazımı filtreli yükleme yapar (yalnız o velinin satırları); "fiili veri var mı" bilgisi erteleme kuralına handler sorgusundan parametre olarak gelir.
- Not okuma kapısı tek sınıfta yazılıdır; not metnini okuyan tipler mimari testle sınırlıdır.
- **Sapma:** "Organizasyonu yayınlamak … bir uygulama servisidir" ifadesi için ayrı bir servis yok; işi yayın handler'ı tek kayıtta yapıyor (domain yorumu hâlâ var olmayan `ParentMeetingPublishService`'e atıf yapıyor).
- **Ek (kararda yok):** şube satırı "son etkinlik" iziyle çocuk satır ve not yazımlarının eşzamanlılık koruyucusu yapıldı; ayrı kökteki not yazımı da şubeye iz bırakır.

## İlgili

- [[Veli Toplantıları]]
- [[0003-coklu-sube-ayri-odev-kaydi]]
- [[0004-odev-hedefi-yayinda-dondurulur]] — davetli listesi aynı mantıkla Başladı anında dondurulur
- [[0026-yanit-plan-surumune-baglanir]]
