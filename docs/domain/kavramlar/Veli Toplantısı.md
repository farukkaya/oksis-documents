---
aliases: [ParentMeeting, Toplantı Organizasyonu, Okul Geneli Veli Toplantısı]
tags: [domain/parent-meetings]
table: academic.parent_meetings
status: active
last-synced: 2026-10-22 (02fbcdc1)
---

# Veli Toplantısı

<!-- generated:start -->

## Nedir

İdarenin tek seferde, okul geneli ya da seçili şubeler için planladığı veli toplantısının **çatı kaydı**. Yalnız başlığı, açıklamayı, sihirbazın varsayılan tarih/saat/yerini ve taslak/yayın durumunu taşır. Toplantının asıl hayatı şube düzeyindedir: her şubenin tarihi, saati, yeri, durumu, iptali ve ertelemesi ayrı bir [[Şube Toplantısı]] kaydında yaşar ([[0025-sube-toplantisi-ayri-agregat]]). Bu kayıt "bu şube toplantıları aynı plandan doğdu" bilgisini ve rapordaki toplamın eksenini sağlar.

Aynı adı taşıyan modül notu [[Veli Toplantıları]]'dır; bu not modülün çatı kavramıdır.

## Yaşam döngüsü

`Draft → Published`. Başka durum yoktur.

- **Draft** — başlık, açıklama, varsayılanlar ve şube kümesi birlikte düzenlenir; kümeden çıkarılan taslak şube silinir. Taslak organizasyon şubeleriyle birlikte silinebilir.
- **Published** — silinemez. Yalnız başlık ve açıklama değişebilir; bu düzenleme sessizdir (bildirim yok) ve işlem kaydına "başlık düzenlendi" olarak yazılır. Varsayılanlar yayından sonra anlamsızdır, değişmez.
- "Kaç şube tamamlandı, kaç şube iptal" bilgisi saklanmaz, şubelerden türetilir.
- Sezon arşivlendiğinde taslak organizasyon şubeleriyle birlikte silinir (sistem işi).

## Kurallar

- **Sezon okulun aktif sezonudur;** seçilen şubeler o sezonun aktif şubeleri olmalıdır, değilse 404. Aktif sezon yoksa ya da organizasyonun sezonu artık aktif değilse idarenin yazma uçları `SeasonArchived` ile reddedilir.
- **Varsayılanlar yalnız başlangıç değeridir.** Şubenin tarih/saat/yeri varsayılandan bağımsızdır; geçmiş-an denetimi varsayılanda değil şube zamanında yapılır.
- **Yayın tek işlemdir.** Bütün taslak şubeler aynı kayıtta yayınlanır; biri geçmiş bir ana düşerse hiçbiri yayınlanmaz ve hata hangi şubeden geldiğini söyler. Taslak şubesi kalmamış organizasyon yayınlanamaz (`EmptySections`). Bildirimler kayıttan **sonra** üretilir.
- **Yayından sonra şube eklenebilir;** eklenen şube taslak beklemeden doğrudan yayına girer ve yalnız o şubenin velilerine haber gider. Yayınlanmış şube organizasyondan çıkarılamaz.
- **Bir şube bir organizasyonda bir kez yer alır** (`DuplicateSection`; veritabanında da tekil).
- Başlık zorunlu ve en çok 200, açıklama en çok 2000 karakterdir. Açıklama bildirim metnine girmez.
- İdarenin organizasyon yazmaları kaydın sürüm damgasını ister; eskimiş damga `Concurrency` (409) döner.

## İlişkiler

- [[Şube Toplantısı]] — bir organizasyonun 1..N şube toplantısı; aralarında navigasyon yok, yalnız kimlik
- [[Sezon]] — organizasyon tek bir sezona bağlıdır; arşiv olayı taslağı siler
- [[Şube]] — dolaylı: şube toplantıları üzerinden

## Geçtiği modüller

- [[Veli Toplantıları]] — kavramın sahibi; planlama, yayın ve organizasyon raporu

<!-- generated:end -->

## Notlar

<El yazısı alan. Senkron buraya dokunmaz.>

## Açık Sorular

- Organizasyonun sınıf yorumu şube yayınını `ParentMeetingPublishService` adlı bir servise bağlıyor; kodda böyle bir servis yok, işi yayın handler'ı tek kayıtta yapıyor. Yorum mu düzeltilmeli, servis mi çıkarılmalı?
