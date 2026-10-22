---
aliases: [ParentMeetingNote, ParentMeetingNoteVersion, ParentMeetingNoteAuditEntry, Veli Görüşme Notu, Görüşme Notu Erişim İzi]
tags: [domain/parent-meetings]
table: academic.parent_meeting_notes
status: active
last-synced: 2026-10-22 (02fbcdc1)
---

# Görüşme Notu

<!-- generated:start -->

## Nedir

Veli toplantısında rehber öğretmenin (ya da idarenin) bir öğrenci için yazdığı serbest metin not — **şube toplantısı × öğrenci başına tek kayıt**. Modülün en dar erişimli verisidir: özel nitelikli kişisel veri içerebilir, veliye ve öğrenciye hiçbir zaman gösterilmez, her okuması kayda geçer, saklama süresi dolunca metni silinir.

Not defterindeki not değeri (Not, `Mark`) ve onun denetim izi (Not Denetim Kaydı, `GradeAuditEntry`) ile **ilgisi yoktur**; ad benzerliği yüzünden karıştırılmasın.

Notun iki parçası bu notun içinde yaşar, ayrı kavram değildir:

- **Sürüm** (`ParentMeetingNoteVersion`) — her kaydın metniyle birlikte saklanan sürümü; "notu kim yazdı" sorusunun cevabı buradadır.
- **Erişim izi** (`ParentMeetingNoteAuditEntry`) — okuma, oluşturma, güncelleme ve imha olaylarının yalnız eklenen izi.

## Yaşam döngüsü

Boş → ilk kayıt (sürüm 1) → yeni sürümler → imha.

- Boş gövdeyle kayıt açılmaz; boş panel bir satır doğurmaz.
- İmha yumuşak silme değildir: notun ve **bütün sürümlerinin** metni kalıcı olarak boşaltılır, imha anı damgalanır; kaydın varlığı ve erişim izi kalır. İmha edilmiş nota yazılamaz.

## Kurallar

- **Yazma penceresi fiili katılımla aynıdır:** [[Şube Toplantısı]] `Started` ya da `Completed` + 7 gün. Arşivlenmiş sezonda kapalıdır. Öğrenci şubenin aktif davetli ağacında olmalıdır.
- **Okuma kapısı** (tek yerde yazılır): idare (yönetme izni), şubenin **güncel** rehber öğretmeni ve notun herhangi bir sürümünü yazmış, hâlâ bu okulda çalışan öğretmen (görevi süren öğretmen profili, aktif kişi, güncel sezonda aktif öğretmen rolü, yazma izni). Yazan okuldan ayrılınca erişimi biter. Eski rehber, başka öğretmen ve veli kapsam dışıdır (404). Taslak şubede not yoktur. Davetli listesinden düşmüş öğrencinin eski notu okunabilir kalır.
- **Yazma yalnız idare ve güncel rehberdir.** Yazan ama artık rehber olmayan öğretmen notu okur, yazamaz (403).
- **Platform hesabı ve öğrenci** modülün hiçbir iznini almaz; veli okuma iznini alır ama not kapısından geçemez.
- **Her okuma izlenir** — not henüz yokken boş paneli açmak da. İlk kayıt "oluşturma", sonraki sürüm "güncelleme", süreli silme "imha" (sistem aktörlü) olarak yazılır. İz metin, özet ya da karakter sayısı taşımaz (şema testiyle kilitli), genel denetim loguna gitmez, okul kapsamlı tabloda kalır ve not imha edilse de durur. İzi yalnız idare listeler; gün süzgeci okulun yerel gününe göredir.
- **Sürüm çakışması:** istemci gördüğü sürümü gönderir; güncel sürüm değilse çakışma döner — güncel sürüm numarası, son düzenleyen ve zaman, **metin değil**. Eşzamanlı iki ilk yazım da çakışma olarak döner.
- **Otomatik kayıt birleştirmesi:** aynı yazar son sürümü 5 dakika içinde otomatik kaydederse yeni sürüm açılmaz ve iz yazılmaz. Elle kayıt her zaman yeni sürüm ve iz üretir.
- **Metin düz metne indirgenir** (satır sonları birleştirilir, kontrol karakterleri atılır, uçlar kırpılır) ve en çok 4000 karakterdir.
- **Metin hiçbir olaya, bildirime, rapora, dışa aktarıma, loga ya da hata mesajına girmez.** Metni okuyabilen tipler mimari testle sınırlıdır (not okuma, not kaydetme, imha işi); rapor ve listeler yalnız "notu var" bayrağını alır.
- **Ayrı köktür** ([[0025-sube-toplantisi-ayri-agregat]]): şube toplantısını yükleyen liste, rapor ve bildirim yolları not tablosuna dokunmaz. Yine de not yazımı şube satırına iz bırakır; eşzamanlı erteleme, iptal ya da tamamlama ile çakışma 409 olur.
- **Erteleme kapısı:** dolu bir not "fiili veri" sayılır; başlamış toplantının tarihi değiştirilemez.
- **Süreli imha:** sezon arşivlendikten sonra okul ayarındaki saklama süresi (varsayılan 2 yıl, 0–10 yıl) dolunca gece işi metni siler. Okul başına tek bir KVKK silme olayı üretilir; olayda yalnız sayılar ve okul vardır, öğrenci ya da kişi kimliği yoktur.

## İlişkiler

- [[Şube Toplantısı]] — notun bağlı olduğu toplantı; ayrı kök, kimlikle bağlı; yazma penceresi oradan okunur
- [[Kişi]] — öğrenci, yazan ve okuyan; hepsi kişi kimliği
- [[Sezon]] — saklama sayacı sezonun arşivlendiği andan başlar
- [[Okul Ayarları]] — saklama süresi
- [[Rol Ataması]] — "yazan öğretmen" kapısı güncel sezondaki aktif öğretmen rolünü arar

## Geçtiği modüller

- [[Veli Toplantıları]] — kavramın sahibi; rehber ekranı, idare not erişim izi ve gece imha işi

<!-- generated:end -->

## Notlar

<El yazısı alan. Senkron buraya dokunmaz.>

## Açık Sorular

- Saklama süresi 0 yıl seçilebiliyor; bu durumda notlar sezonun arşivlendiği gecenin işinde silinir. Bilinçli bir alt sınır mı?
- "Yazan öğretmen" kapısı güncel sezonda aktif öğretmen rolü arıyor; sezon geçişinden hemen sonra (yeni sezon güncel, rol ataması henüz yapılmamış) yazan öğretmenin eski notlara erişimi kesiliyor olabilir. Doğrulanmadı.
