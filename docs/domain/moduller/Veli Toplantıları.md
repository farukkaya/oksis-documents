---
aliases: [ParentMeetings, api/v1/parent-meetings, Veli Toplantısı Modülü, Parent Meetings]
tags: [domain/parent-meetings, module]
status: completed
last-synced: 2026-10-22 (02fbcdc1)
---

# Veli Toplantıları

<!-- generated:start -->

## Ne yapar

İdare (okul yöneticisi ve müdür yardımcısı) okul geneli ya da seçili şubeler için **tek bir organizasyon** altında veli toplantısı planlar; her şubenin toplantısı ayrı yaşar, ayrı ertelenir, ayrı iptal edilir, ayrı tamamlanır. Yayında şubenin davetli velileri haber alır ve "Katılacağım / Katılamayacağım" der; toplantı başlayana kadar fikrini değiştirebilir, idare de aynı pencerede velinin adına yanıt girebilir. Toplantı günü şubenin **güncel** rehber öğretmeni (ya da idare) beyandan bağımsız olarak kimin geldiğini işaretler, öğrenci başına görüşme notu yazar ve toplantıyı tamamlar. Not veliye ve öğrenciye hiçbir zaman gösterilmez; her okuması izlenir, saklama süresi dolunca metni silinir. İdare beyan ile fiili katılımı iki ayrı eksende raporlar ve Excel'e aktarır.

Modülün karakteri üç ilkeye dayanır: **sorumlu dondurulmaz** (yetki her istekte şubenin güncel rehberinden okunur), **yanıt plan sürümüne bağlıdır** (tarih değişince eski beyan geçmişte kalır, sessizce taşınmaz) ve **not ayrı köktür** (liste, rapor ve bildirim yolları notun metnine dokunamaz).

## Kullandığı kavramlar

- [[Veli Toplantısı]] — çatı kayıt; `Draft | Published`; başlık, açıklama, varsayılanlar
- [[Şube Toplantısı]] — asıl kök; `Draft → Published → Started → Completed`, `Cancelled`; davetli, yanıt, yanıt geçmişi, fiili katılım ve işlem kaydı içinde
- [[Görüşme Notu]] — öğrenci başına; ayrı kök, sürümlü, okuma denetimli, süreli imha
- [[Şube]] — toplantının birimi; rehber öğretmen her istekte buradan okunur
- [[Sezon]] — organizasyon aktif sezona bağlanır; arşiv olayı açık toplantıları kapatır, notların saklama sayacını başlatır
- [[Öğrenci Kaydı]] — davetli kümesinin öğrenci ekseni (aktif ve dondurulmuş kayıt)
- [[Veli-Öğrenci İlişkisi]] — davetli kümesinin veli ekseni; yalnız bilgi görme yetkili, iptal edilmemiş, geçerlilik aralığındaki bağ
- [[Kişi]] — veli, öğrenci, öğretmen ve aktör hep kişi kimliğidir; çağıranın kimliği oturumdan çözülür
- [[Okul Ayarları]] — dört ayar: sabah hatırlatma saati (05:00–12:00, varsayılan 08:00), "başlıyor" haberi (varsayılan kapalı), "yanıt bekleniyor" çağrısı (varsayılan açık), not saklama süresi (3–10 yıl, varsayılan 3; sayaç sezon arşivinden). Aralıklar modülün politika sınıfındadır; diğer süreler (3 gün otomatik kapanış, 7 gün düzeltme penceresi, 2 gün önce çağrı) sistem sabitidir
- [[Okul]] — okulun saat dilimi ve günü; bütün zaman kararları buradan
- [[İzin]] — üç anahtar: okuma, yazma (katılım, not, tamamlama), yönetme (planlama, erteleme, iptal, veli adına yanıt, rapor, ayarlar)
- [[Bildirim]] / [[Bildirim Türü]] — yedi tür; ayrı "Veli toplantısı" grubu

## Ana akışlar

1. **Planlama** — Form bağlamı aktif sezonun aktif şubelerini verir. Yayın öncesi önizleme yan etkisizdir: davetli sayıları, hesabı olmayan ve e-postası olmayan veliler, aynı velinin başka toplantısıyla zaman çakışması (bitiş yoksa 60 dakika varsayılır; uç uca değen toplantı çakışmaz). Rehbersiz şube ve geçmiş başlangıç uyarıdır. Organizasyon taslak doğar; taslakta başlık, varsayılanlar ve şube kümesi birlikte değişir, taslak silinebilir. Bir organizasyonda bir şube bir kez yer alır.

2. **Yayın** — Bütün taslak şubeler tek işlemde yayınlanır; biri geçmiş ana düşerse hiçbiri yayınlanmaz. Her şube için davetliler bu anda çözülür. Kayıttan sonra velilere yayın haberi, rehber öğretmene bilgi haberi gider (rehber değişiklik ve iptalde de bilgi haberi alır). Yayınlanmış organizasyona sonradan şube eklenebilir; eklenen şube doğrudan yayına girer, haber yalnız onun velilerine gider. Yayından sonra organizasyonda yalnız başlık ve açıklama sessizce düzenlenir.

3. **Plan değişikliği ve iptal** — İdare bir ya da birden çok şubenin tarih/saat/yerini önizleme + uygulama çiftiyle değiştirir; önizlemenin etki özeti uygulama isteğinde taşınır, arada veri değiştiyse istek reddedilir. Yalnız tarih ya da başlangıç saati değişikliği yanıtları sıfırlar ([[0026-yanit-plan-surumune-baglanir]]); yer ve yalnız bitiş saati değişikliği yanıtları korur. Başlamış ama henüz katılım/not girilmemiş toplantı ertelenebilir ve yayına geri döner. İptal toplu, gerekçesi isteğe bağlıdır; velilere ve rehbere iptal haberi gider.

4. **Veli yüzü** — Veli yalnız kendi aktif davetli olduğu ve bağı şu an canlı olan şube toplantılarını görür (yaklaşan / geçmiş / bugün); taslak görünmez. Kartta çocuklar, açıklama ve aynı velinin diğer toplantılarıyla çakışma bandı vardır; ikiz çocuklu velinin tek yanıtı vardır. Yanıt yayında ve başlangıçtan önce verilir; değişmeyen yanıt iz bırakmaz; yanıt hiçbir bildirim üretmez.

5. **İdarenin veli adına yanıtı** — Velinin penceresiyle aynı pencerede, hedef velinin bağı şu an canlıysa; geçmişe idareci aktör, veli "adına", isteğe bağlı kanal notuyla yazılır. Yanıt geçmişi idare ve güncel rehber tarafından veli başına okunur.

6. **Toplantı günü (rehber yüzü)** — Başlatma işi başlangıç anında şubeyi `Started`'a geçirir, davetli listesini dondurur ve okul ayarı açıksa "başlıyor" haberini gönderir. Rehber "Toplantılarım" listesinde yalnız bugün rehberi olduğu şubeleri görür (yaklaşan / geçmiş / bugün / tamamlanmayı bekleyen). Veli başına "Geldi" işaretler, öğrenci başına görüşme notu yazar, toplantıyı elle tamamlar; tamamlama anındaki davetli, gelen ve işaretsiz veli sayıları kayda geçer. İşaretlenmeyen veli tamamlanınca "gelmedi" sayılır. Tamamlandıktan sonra 7 gün boyunca katılım ve not düzeltilebilir.

7. **Raporlama** — Organizasyon raporu (idare) ve şube raporu (idare + güncel rehber) beyanı ve fiili katılımı ayrı sütunlarda sayar; katılım oranının paydası organizasyon toplamlarına bağlıdır, şubeler doğal sırada (9-A, 9-B, 10-A) listelenir. Raporda yalnız "notu var" bayrağı vardır. Excel'e aktarım yalnız idarenindir; not metni dosyaya girmez.

8. **Not erişim izi** — İdare, görüşme notlarının kim tarafından, hangi öğrenci ve toplantı için, hangi eylemle (okuma, oluşturma, güncelleme, imha) ve hangi sürümde açıldığını süzerek listeler. İz metin taşımaz.

9. **Bildirim ve zamanlanmış işler** — Yedi tür: yayın (veli), rehbere bilgi, değişiklik (veli), iptal (veli), sabah hatırlatması (veli + rehber, toplantı günü), toplantı başlıyor (veli, okul ayarı) ve toplantıdan 2 gün önce yanıtsızlara çağrı (okul ayarı). Hepsinde push varsayılanı açıktır. E-posta yalnız yayın, değişiklik ve iptalde seçilebilir ve varsayılanı açıktır; diğer dört türde seçilemez ([[0027-eposta-varsayilani-tur-bazli]]). Bildirim metnine açıklama, neden, gerekçe ve not girmez. Hesabı olmayan veliye bildirim gitmez; yayın, değişiklik ve iptalde yalnız SMS niyeti kaydedilir, SMS gönderilmez. Kuyruktaki haber gönderilmeden önce şubenin güncel durumu ve sürümüyle tazeliği sorulur; eskimiş haber atılır. İşler: başlatma (5 dk), hatırlatma süpürmesi (15 dk), davetli uzlaştırması (saatlik), otomatik kapanış (gece; toplantı gününden 3 gün sonra) ve not imhası (gece). Olaylardan (veli bağı, şube ataması, nakil, kişi arşivi, sezon arşivi) gelen işler tetikleyen komutun kaydı bittikten sonra kuyruğa alınır; hata tetikleyen komutu bozmaz.

**Yetki — izin ve görünüm.** Okuma okul yöneticisi, müdür yardımcısı, öğretmen ve velidedir; yazma okul yöneticisi, müdür yardımcısı ve öğretmendedir; yönetme yalnız okul yöneticisi ve müdür yardımcısındadır. Platform hesabı ve öğrenci hiçbir izni almaz — platform hesabı okul içi veli toplantısını, hele görüşme notunu görmez. İkinci katman görünümdür ve istemciden gelmez: yönetme izni → idare; değilse şubenin güncel rehberi → rehber; değilse kapsam dışı (404). Taslak şube yalnız idareye açıktır.

**Hata sözleşmesi.** Kodlar `ParentMeeting.` önekiyle gelir; doğrulama, geçmiş an, bitiş-başlangıç ve boş şube listesi 400; kapsam dışı, kayıt yok ve davetli olmayan hedef 404; not yazma yetkisi olmayan okuyucu 403; durum, kilit, kapalı pencere, fiili veri, sürüm çakışması, eskimiş önizleme, arşiv ve yinelenen şube 409. Toplu uçlarda hata hangi öğeden geldiğini alan olarak taşır.

## Kapsam dışı

- **SMS gönderimi.** Hesabı olmayan veli için yalnız niyet kaydedilir; kanal yok.
- **Başka öğretmeni sorumlu atamak.** Sorumlu her zaman şubenin güncel rehberidir; kayıtta tutulmaz.
- **Toplantı başladıktan sonra beyan.** Veli için de idare için de pencere başlangıçta kapanır; anlam fiili katılımdadır.
- **Veliye ve öğrenciye görüşme notu.** Hiçbir yüzde yoktur.
- **Öğrenci yüzü.** Öğrenci modülün hiçbir iznini almaz.
- **Rehberin Excel aktarımı.** Dışa aktarım yalnız idarenindir.
- **Yanıt değişikliği bildirimi.** Velinin ya da idarenin yanıtı kimseye haber üretmez.

<!-- generated:end -->

## Notlar

> **Aşağısı 2026-10-15 planlama aşamasının el yazısı notudur (kod yazılmadan önce).** Senkron buna dokunmadı. Üst bloktaki üretilmiş içerik bunun yerini alıyor; ayıklamak ya da silmek kullanıcının kararıdır. Bu bölümdeki [[Davetli Veli]], [[Katılım Yanıtı]], [[Yanıt Geçmişi]] ve [[Fiili Katılım]] bağlantıları ayrı not olarak açılmadı; artık [[Şube Toplantısı]] notunun içindedir.

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

## Açık Sorular

- Yanıtlandı (2026-10-22): sezon ölçütü tektir (aktif değilse bütün yazmalar reddedilir); not saklama alt sınırı 3 yıldır; öğretmenin not okuma kapısı sezondan bağımsızdır, okuldan ayrılınca biter.
- **İptal gerekçesi ve işlem kaydı okunmuyor:** ikisi de yazılıyor, hiçbir uç döndürmüyor (bkz. [[Şube Toplantısı]]).
- **"Hiçbir modül bu modüle bağımlı değildir" artık tam doğru değil:** okul ayarlarının domain sınıfı ayar aralıklarını bu modülün politika sınıfından okuyor; Bildirimler modülünün olay eşlemesi yedi türü ve şube toplantısı kaynak tipini biliyor. Bağımlılık soyutlama üzerinden mi kalmalı, yoksa aralıklar okul ayarlarına mı taşınmalı?
- **Eski el yazısı özetle iki çelişki** (yukarıdaki planlama bölümü): (1) "Tarih ya da saat değişince yanıtlar sıfırlanır" — kodda yalnız tarih ve **başlangıç** saati sıfırlar, yalnız bitiş saati değişikliği yanıtları korur. (2) Kararlar "(proposed)" yazıyor; üç kararın frontmatter'ı zaten `accepted`.
- E-posta kanalının yapılandırma kapısı varsayılan kapalı ve ADR 0027'ye göre varsayılan-açık e-postanın KVKK dayanağı hukuk teyidine bağlı; teyidin durumu koddan çıkmıyor.
