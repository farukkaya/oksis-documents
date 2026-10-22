---
aliases: [ParentMeetingSection, ParentMeetingInvitee, ParentMeetingResponse, ParentMeetingResponseHistory, ParentMeetingPresence, ParentMeetingAuditEntry, Şube Veli Toplantısı, Davetli Veli, Katılım Yanıtı, Yanıt Geçmişi, Fiili Katılım]
tags: [domain/parent-meetings]
table: academic.parent_meeting_sections
status: active
last-synced: 2026-10-22 (02fbcdc1)
---

# Şube Toplantısı

<!-- generated:start -->

## Nedir

Bir [[Veli Toplantısı]]'nın tek bir şubedeki hâli: kendi tarihi, saati, yeri, durumu, iptali ve ertelemesi olan, veliyi davet eden, beyanını toplayan ve toplantı günü kimin geldiğini kaydeden birim. Modülün asıl kökü budur; organizasyon yalnız çatıdır ([[0025-sube-toplantisi-ayri-agregat]]). Sorumlusu kayıtta tutulmaz: her istekte şubenin **güncel** rehber öğretmeni yetkilidir.

Şube toplantısının beş parçası bu notun içinde yaşar, ayrı kavram değildir:

- **Davetli veli** (`ParentMeetingInvitee`) — öğrenci × veli çifti; yayında doğar, başlangıçta donar. Düşen satır silinmez, nedeniyle işaretlenir.
- **Katılım yanıtı** (`ParentMeetingResponse`) — velinin bir plan sürümündeki beyanı: "Katılacağım" ya da "Katılamayacağım". "Yanıt yok" bir satır değil, satırın yokluğudur. İkiz çocuklu velinin tek yanıtı vardır.
- **Yanıt geçmişi** (`ParentMeetingResponseHistory`) — her beyan değişikliğinin, veli adına girişin ve plana bağlı sıfırlamanın yalnız eklenen izi: kim, ne zaman, kimin adına, hangi kanaldan.
- **Fiili katılım** (`ParentMeetingPresence`) — veli başına "Geldi" işareti. Beyandan ön dolmaz; beyan ile fiili katılım iki ayrı eksendir.
- **İşlem kaydı** (`ParentMeetingAuditEntry`) — organizasyon ve şube olaylarının yalnız eklenen izi (oluşturma, yayın, şube ekleme, erteleme, yer/bitiş saati değişikliği, başlık, iptal, başlama, tamamlama, sezon arşivi, davetli ekleme/düşme, katılım işareti).

## Yaşam döngüsü

`Draft → Published → Started → Completed`; `Published | Started → Cancelled`. `Completed` ve `Cancelled` terminaldir.

- **Draft** — yalnız idare görür; rehber öğretmene ve veliye görünmez. Tarih/saat/yer değişikliği plan sürümünü artırmaz. Silinebilir.
- **Published** — yayın anında davetliler çözülür (boş davetli kümesi geçerlidir; rehbersiz ya da öğrencisiz şube önizlemede uyarılır, engellenmez). Yanıt penceresi açıktır. Davetli listesi canlıdır: veli bağı, şube ataması, nakil ve kişi arşivi olaylarında anında; olay üretmeyen değişimlerde (bilgi görme yetkisi, bağ süresinin dolması) saatlik işle uzlaştırılır.
- **Started** — sistem işi başlangıç anı geldiğinde geçirir (5 dakikada bir koşar); an gelmeden başlatılamaz. Bu anda davetli listesi donar, beyan herkes için kapanır, fiili katılım ve [[Görüşme Notu]] açılır.
- **Completed** — üç yoldan: elle (güncel rehber ya da idare), otomatik (toplantı gününden 3 gün sonra gece işi) ya da sezon arşivi. Tamamlandıktan sonra 7 gün boyunca fiili katılım ve not düzeltilebilir. İşaretlenmemiş veli bu durumda "gelmedi" sayılır; bu değer yazılmaz, türetilir.
- **Cancelled** — idare toplu iptal eder; gerekçe isteğe bağlıdır. Kayıtlar korunur, velilere iptal haberi gider.
- **Erteleme `Started`'dan geri döndürür:** başlamış toplantının tarihi ya da başlangıç saati değişirse toplantı henüz yapılmamış sayılır, `Published`'a döner ve davetli listesi yeniden canlanır — ama yalnız hiç fiili katılım ve dolu not yoksa.
- **Sezon arşivi:** `Started` şube tamamlanır, `Published` şube sistemce (gerekçesiz) iptal edilir ve velilere iptal haberi gider, `Draft` şube silinir. Üçünde de aktör yoktur.

## Kurallar

- **Geçmiş bir an planlanamaz;** bugünün ileri bir saati olabilir. Denetim oluşturmada, taslak düzenlemede, yayın anında yeniden ve ertelemede yapılır (`StartInPast`). Bitiş saati varsa başlangıçtan sonradır; yer zorunludur (en çok 200 karakter). Saatler dakikaya kırpılır; mutlak an okulun saat diliminden çözülür.
- **Plan sürümü** ([[0026-yanit-plan-surumune-baglanir]]): yalnız **tarih ya da başlangıç saati** değişikliği plan sürümünü artırır. Artışta güncel yanıtı olan her veli için geçmişe "plana bağlı sıfırlama" satırı yazılır ve güncel yanıt "yanıt yok"a döner. **Yer** ya da **yalnız bitiş saati** değişikliği yanıtları korur, ayrı bir bilgi sürümünü artırır. Üç değişiklik türünde de velilere haber gider.
- **Plan değişikliği önizlemesiz uygulanmaz.** İdare önce etki önizlemesini alır (sıfırlanacak yanıt, haber alacak veli, eklenecek çakışma sayıları); uygulama isteği önizlemenin etki özetini ve her şubenin beklenen plan sürümünü taşır. Arada herhangi bir şey değiştiyse istek `PreviewRequired` ya da `Concurrency` ile reddedilir. Toplu plan değişikliği ve toplu iptal tek istekte en çok 200 şubedir.
- **Başlamış toplantıda fiili katılım ya da dolu not varsa tarih/saat değiştirilemez** (`HasFieldData`).
- **Beyan penceresi:** veli kendi yanıtını yalnız toplantı yayındayken ve başlangıç anından önce verir ya da değiştirir. Başlatma işi geç koşsa da pencere başlangıç anında kapanır (`ResponseLocked`). İdarenin veli adına yanıtı **aynı penceredir** (2026-10-15 kararı); toplantı başladıktan sonra anlam fiili katılımdadır.
- **Yanıtın sahibi oturumdan çözülür,** istekten alınmaz. Veli adına girişte geçmiş satırı aktör olarak idareciyi, "adına" olarak veliyi ve isteğe bağlı bir kanal notunu (en çok 200 karakter; "telefonla") taşır.
- **Katılamama nedeni** isteğe bağlıdır (en çok 500 karakter), yalnız "Katılamayacağım"da tutulur ve yalnız idare/rehber görünümüne girer; bildirime, loga ve işlem kaydına girmez. Değeri değişmeyen yanıt geçmiş yazmaz. **Yanıt bildirim üretmez.**
- **Davetli:** şubenin sezondaki mevcudu (aktif ya da dondurulmuş kayıt) × öğrencinin iptal edilmemiş, bilgi görme yetkili ve geçerlilik aralığındaki velileri. Bilgisi kısıtlı veliyi davet eden istisna bu modüle uygulanmaz. Tekillik şube × öğrenci × velidir. Düşme nedenleri: bağ iptali, bilgi kısıtı, bağ süresinin dolması, öğrencinin şubeden ya da okuldan ayrılması. Geri dönen çift aynı satırı yeniden açar ve "yayından sonra eklendi" işaretlenir; yayın haberi yalnız ona gider. Başlatma anındaki son uzlaştırma yayın haberi göndermez.
- **Canlı bağ kapısı:** veli adına yanıt ve fiili katılım işareti, davetli satırı donmuş olsa bile velinin bağının **şu an** canlı olmasını ister; değilse 404. Velinin kendi görünürlüğü de aynı süzgeçtendir; uzlaştırmayı beklemez.
- **Fiili katılım:** veli başına tek satır; satır yoksa "işaretlenmedi". İşaretsiz veliye "gelmedi" yazmak satır açmaz. Pencere `Started` ya da `Completed` + 7 gündür (`FieldDataClosed`). İşaretleyen şubenin güncel rehber öğretmeni ya da idaredir; değişmeyen işaret işlem kaydı yazmaz.
- **Elle tamamlama** işlem kaydına o anki davetli, gelen ve işaretsiz sayılarını yazar; not metni yazılmaz.
- **İptal gerekçesi** en çok 500 karakterdir; şube satırında ve işlem kaydının iptal satırında durur, veliye ve bildirime gitmez.
- **Hatırlatma damgaları plan sürümüne bağlıdır:** sabah hatırlatması, "yanıtınız bekleniyor" çağrısı ve "toplantı başlıyor" haberi sürüm başına bir kez gider; erteleme yeni sürüm doğurduğu için yeniden gidebilir. Sürüm o günün sabah saatinden sonra doğduysa hatırlatma gönderilmeden damgalanır. "Başlıyor" haberi başlangıçtan 30 dakika sonrasına kalmışsa gitmez.
- **Eşzamanlılık:** şube satırı sürüm damgası taşır. Çocuk satır (yanıt, davetli, işaret) ya da not yazan her istek şubeye bir "son etkinlik" izi bırakır; böylece eşzamanlı erteleme, başlatma, iptal ya da tamamlama ile çakışma 409 olur, eski sürüme sessiz yazım olmaz.
- **Yalnız eklenen tablolar:** yanıt geçmişi ve işlem kaydı yumuşak silme taşımaz; güncelleme ve silme çağrısı mimari testle yasaktır. Sistem geçişlerinde (başlama, otomatik tamamlama, sezon arşivi) aktör boştur.
- **Sezon aktif değilse** (hazırlık ya da arşiv) fiili katılım, not, tamamlama, veli yanıtı ve idarenin veli adına yanıtı kapalıdır (`SeasonArchived`); idare yazmalarıyla aynı ölçüt.
- **Hata sözleşmesi:** kodlar `ParentMeeting.` önekiyle gelir. Doğrulama, geçmiş an, bitiş-başlangıç ve boş şube listesi 400; kayıt yok, kapsam dışı ve davetli olmayan hedef 404 (varlık sızdırılmaz); kalan her şey (durum, kilit, kapalı pencere, fiili veri, sürüm, önizleme, arşiv, yinelenen şube) 409.

## İlişkiler

- [[Veli Toplantısı]] — çatı; N şube toplantısı → 1 organizasyon, yalnız kimlik
- [[Şube]] — tek şube; modüller arası yabancı anahtar yok, yalnız kimlik. Sorumlu her istekte şubenin güncel rehber öğretmeninden okunur; yalnız o sezonun aktif şubesi seçilebilir
- [[Sezon]] — denormalize taşınır; arşiv olayı açık şubeleri kapatır
- [[Öğrenci Kaydı]] — davetli kümesinin öğrenci ekseni (yayın anındaki ve sonradan değişen mevcut)
- [[Veli-Öğrenci İlişkisi]] — davetli kümesinin veli ekseni ve canlı bağ kapısı
- [[Kişi]] — öğrenci, veli, aktör; hepsi kişi kimliği
- [[Görüşme Notu]] — ayrı kök; şube toplantısını yükleyen hiçbir yol notu getirmez
- [[Okul Ayarları]] — sabah hatırlatma saati, "başlıyor" ve "yanıt bekleniyor" anahtarları
- [[Bildirim Türü]] — yedi tür; olay kimliği plan sürümünü içerir, gönderim anında tazelik sorulur

## Geçtiği modüller

- [[Veli Toplantıları]] — kavramın sahibi
- [[Bildirimler]] — bildirim kaynağı; kuyruktaki haber gönderilmeden önce şubenin durumu ve sürümüyle tazeliği denetlenir, eskimiş haber atılır

<!-- generated:end -->

## Notlar

<El yazısı alan. Senkron buraya dokunmaz.>

## Açık Sorular

- **İptal gerekçesi saklanıyor ama hiçbir uç döndürmüyor.** Domain yorumu "yalnız idare DTO'sunda görünür" diyor; idare DTO'larında gerekçe alanı yok. Bilinçli mi, eksik mi?
- **İşlem kaydını okuyan uç yok.** Yazılıyor ama okuma yolu (idare zaman çizelgesi gibi) bulunamadı.
- İşlem kaydı sınıf yorumu hem "iptal gerekçesi asla girmez" hem "yalnız iptal satırında" diyor; kod gerekçeyi iptal satırına yazıyor. Yorumdaki çelişki düzeltilmeli.
- Yanıtlandı (2026-10-22): bütün yazma uçları tek ölçütü kullanır, sezon aktif değilse reddeder.
