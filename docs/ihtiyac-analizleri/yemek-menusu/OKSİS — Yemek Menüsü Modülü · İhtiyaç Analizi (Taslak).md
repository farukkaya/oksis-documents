
| | |
|---|---|
| **Belge türü** | İhtiyaç analizi — taslak (eğitimci + ürün perspektifi), karar girdisi |
| **Talep** | Yemek Menüsü modülü (katalog anahtarı `cafeteria`) |
| **Tarih** | 15 Ekim 2026 |
| **Durum** | **Taslak — Faruk'un kararlarını bekliyor** (§9). Kilitli karar yok; aşağıdaki "öneri"ler karar değildir. |
| **Hedef kitle** | Özel okullar (anaokulu → lise). MEB genelgeleri varsayılan kısıt sayılmadı; mevzuat yalnız not düzeyinde (§4). |
| **Kod dalı** | `feature/yemek-menusu` (oksis-api-yemek-menusu, oksis-ui-yemek-menusu) — modül için **kod yok** (2026-10-15 tarandı, §5) |
| **Önceki belgeler** | `genel/mvp-kapsami.md` §3: "Yemekhane / Kantin — Sprint 5+"; `genel/urun-tanimi.md`: "Yemekhane / servis modülü" kapsam dışı listesinde |

> **Okuma notu:** Bu belge davranışı ve kapsam seçeneklerini tanımlar; veri modeli ve API teknik analizde çıkar. Geçen kod adları (`mealMenu`, `dish` vb.) **öneridir**; MEB sözlüğüne (oksis-ui `CLAUDE.md`) girmeden kodlanmaz (§11).
>
> **Doğrulama işaretleri:** **[D]** = birincil kaynakta doğrulandı · **[K]** = kısmen / dolaylı kaynak (okul sitesi, haber, arama özeti) · **[T]** = doğrulanamadı, tahmin.

---

## 1. Yönetici Özeti

Özel okulların neredeyse tamamı yemek hizmeti veriyor ve menüyü bugün **aylık PDF/Excel** olarak web sitesine, kampüs panolarına ve (K12NET kullananlar) veli portalına koyuyor. Velinin derdi basit: **"Çocuğum bugün ne yiyecek?"** ve alerjisi olan çocukların velisi için **"Bugün içinde süt/fındık/gluten var mı?"**. PDF bu iki soruyu telefonda kötü yanıtlar (yakınlaştır, kaydır, doğru ayı bul).

K12NET 2023'te bu alana iki ayrı modülle girdi: **yemek hizmeti (menü)** modülü — yemek kataloğu, kategoriler, günlük/haftalık menü, porsiyon/açıklama, görsel, kademeye uyarlama, öğrenci/veli portalında görüntüleme — ve **kafeterya (sipariş/ödeme)** modülü. Türkçe modül sayfası ayrıca öğretmenin öğrenci başına **"Yedi/Yemedi"** işaretlemesini anıyor. Kalori, alerjen, Excel içe aktarma ve bildirim konusunda K12NET'e dair doğrulanabilen bir kaynak bulunamadı.

**Öneri (MVP):** "Dijital pano" düzeyinde, ama yapılandırılmış veriyle bir menü: idare (ya da yetki verdiği personel) menüyü **gün × öğün × yemek** olarak girer, **menü grubu** ile anaokulu/ilkokul/ortaokul-lise farkını karşılar, yemeklere isteğe bağlı **alerjen etiketi** koyar, **taslak → yayın** ile yayımlar. Veli/öğrenci/öğretmen anasayfada **"Bugünün menüsü"** kartını, ayrıntıda haftalık/aylık görünümü görür; yayında ve **yakın tarihli değişiklikte** bildirim gider. Yemekhane yoklaması (yedi/yemedi), öğrenci alerji/diyet kaydı, Excel içe aktarma ve ücret/kantin **MVP dışında** kalır.

> **Tek cümle:** Müdür yardımcısı diyetisyenin gönderdiği Ekim menüsünü yarım saatte girer, "geçen haftayı kopyala" ile hızlanır, "Yayınla"ya basar; veli sabah 07:30'da telefonda "Bugün: Mercimek çorbası · Fırın tavuk · Pirinç pilavı · Ayran — Süt içerir" satırını görür.

---

## 2. Araştırma Bulguları

### 2.1 K12NET — yemek menüsü nasıl uygulanıyor

K12NET'in kamuya açık bir yardım sayfası (adım adım kılavuz) bulunamadı; `dokuman.k12net.com` altındaki veli portalı kılavuzunda (eski `.doc`) yemek/menü geçmiyor. Bulgular üç kaynaktan: K12NET'in İngilizce haber yazıları (2023), Türkçe modül sayfası ve K12NET kullanan okulların siteleri.

| # | Bulgu | Durum | Kaynak |
|---|---|---|---|
| K-1 | **Ayrı bir "yemek hizmeti yönetimi" (food service management) modülü** 8 Mart 2023'te duyuruldu: okullar menülerini öğrenci ve velilerle paylaşır. | **[D]** | [K12NET — "Delicious News from K12NET" (8 Mart 2023)](https://www.k12net.com/en/?p=455) · [haber listesi](https://k12net.com/en/news/page/2) |
| K-2 | Menü **yemek kataloğundan** kurulur: yiyecekler listelenir, **kategorilere** ayrılır, menü bunlardan hazırlanır. | **[D]** | aynı yazı |
| K-3 | **Günlük ve/veya haftalık** menü; her öğün için **açıklama, porsiyon, sunum** bilgisi; yemeklere **görsel** eklenebilir. | **[D]** | aynı yazı |
| K-4 | Menü **farklı sınıf kademelerine uyarlanabilir** (kademe bazlı menü). Uyarlamanın kademe mi, seviye mi, şube mi olduğu belirsiz. | **[D]** (ayrıntısı **[T]**) | aynı yazı |
| K-5 | **Öğrenci ve veli portalında** günlük/haftalık yemek planı görüntülenir; veli "çocuğunun beslenmesini takip eder". | **[D]** | aynı yazı |
| K-6 | Türkçe modül sayfası (Modül 39 "Yemekhane"): servis ürünleri tanımlanır; **günlük, haftalık, aylık** menü planı ve raporlar; **öğretmen öğrencinin yemeğini yiyip yemediğini "Yedi/Yemedi" ile işaretler**; veli ve öğrenci hem menüyü hem bu kayıtları portalda izler. | **[D]** | [K12NET — Modüller](https://k12net.com/okul-yonetim-yazilimi/k12net-moduller/) |
| K-7 | Ayrı bir **kafeterya modülü** (29 Mayıs / Haziran 2023): kafeterya personeli **sipariş ve ödemeyi** tek platformda yönetir, ürün seçimi, fiyat kataloğu, kayıt/raporlama. Ayrıca **Kantin** modülü (Modül 37, stok verisiyle çalışır). Menüden bağımsızdır. | **[D]** (ayrıntı kısmen **[K]**: yazının tam metni alınamadı) | [haber listesi](https://k12net.com/en/news/page/2) · [Modüller](https://k12net.com/okul-yonetim-yazilimi/k12net-moduller/) |
| K-8 | Okullar menüyü **K12NET veli portalı üzerinden aylık** paylaşıyor; aynı okul menüyü **kampüs panolarında** da asıyor (yani portal PDF/pano alışkanlığının yerini tamamen almıyor). | **[K]** (okul sitesi) | [Bilişim Koleji — Yemek Listesi](https://bilisim.k12.tr/yemek-listesi/) |
| K-9 | K12NET kullanan bazı okullar menüyü **yine kendi sitesinde PDF** olarak yayınlıyor (TED Kayseri: K12NET girişi ayrı, menü `EKIM-2026-11.pdf`; ENKA: K12NET sayfasının yanında ayrı "Yemek Menüsü" bağlantısı). Modülün her K12NET okulunda açık olmadığı ya da okulların ikisini birlikte kullandığı anlaşılıyor. | **[K]** | [TED Kayseri — Veli Alanı](https://www.tedkayseri.k12.tr/veli-alani/) · [TED Kayseri — Aylık Yemek Listesi](https://www.tedkayseri.k12.tr/aylik-yemek-listesi/) · [ENKA — k12net.com](https://www.enka.k12.tr/istanbul/tr/k12net-com-2/) |
| K-10 | **Kalori / besin değeri / alerjen alanı** K12NET kaynaklarında **geçmiyor**. Var olup olmadığı bilinmiyor. | **[T]** doğrulanamadı | — |
| K-11 | **Excel/şablonla menü içe aktarma** — kaynak yok. | **[T]** doğrulanamadı | — |
| K-12 | **Menü yayınında/değişiminde bildirim** (push/SMS) — kaynak yok. | **[T]** doğrulanamadı | — |
| K-13 | **Menü girişini kimin yaptığı** (rol/izin) — kaynak yok. Modül sayfası yalnız "tanımlayabilirsiniz" diyor. | **[T]** doğrulanamadı | — |
| K-14 | **Menü ile ücret/yemekhane kaydı bağı**: K12NET SSS'te kayıt zarfında "Eğitim ve Yemek Ücretleri" bölümü geçiyor → yemek ücreti **finans/kayıt** tarafında; menü modülüyle bağlantısı belirsiz. | **[K]** (arama özeti; SSS sayfası açılarak okunmadı) | [K12NET SSS](https://sss.k12net.com/sss.html) |
| K-15 | Mobil uygulama (K12NET Mobil / K12NET App) açıklamasında yemek menüsü özelliğinin anıldığı doğrulanamadı. | **[T]** | [App Store — K12NET Mobil](https://apps.apple.com/tr/app/k12net-mobil/id1155767502?l=tr) |

**Çıkarım (yorum):** K12NET'te menü bir **katalog + takvim** modeli; asıl farkı **anaokulu için "Yedi/Yemedi"** kaydı ve **ayrı kafeterya/kantin satış** modülleri. OKSİS'in MVP'sinde K12NET'in menü yüzünü (katalog, gün/öğün, kademe, portal) karşılamak gerçekçi; yoklama ve satış/ödeme ayrı modül ölçeğinde işler.

### 2.2 Diğer sistemler — kısa karşılaştırma

| Sistem | Menü özelliği | Durum | Kaynak |
|---|---|---|---|
| **e-Okul VBS** | Genel okul yemek menüsü yok; yalnız **pansiyon yemek menüsü** (pansiyonlu resmî okullar) anılıyor. | **[K]** (haber sitesi) | [CNN Türk — e-Okul VBS](https://www.cnnturk.com/turkiye/e-okul-vbs-giris-ile-devamsizlik-bilgisi-ve-not-sorgulama-yapilabiliyor-904197) |
| **Minik İz** (anaokulu) | Haftalık menüyü **yönetici** girer; velide "Yemek menüsü" sekmesi; öğretmen "Sınıfım"dan haftalık menüye geçer; öğretmen çocuk başına **yemek tüketimi: hepsi / yarım / az / yemedi** girer, veli günlük raporda görür. | **[D]** (tanıtım sayfası) | [minikiz.com.tr](https://minikiz.com.tr/) |
| **Ekid+** (okul öncesi) | Yönetici yemek listesini **uygulama içinde etkileşimli**, **dosya** olarak ya da **başka yerdeki listeye bağlantı (URL)** ile paylaşır; Excel'e aktarım; veli günün listesini görür. | **[D]** (tanıtım sayfası) | [Ekid+ — Yemek listesi](https://www.ekidplus.com/okul-oncesi-sistemi-yemek-listesi.html) |
| **Kuki** (okul öncesi) | Öğün ayrıntısında yağ, protein, karbonhidrat ve **alerjen** bilgisi. | **[K]** (arama özeti; site açılamadı) | [kuki.vilicore.com](https://kuki.vilicore.com/) |
| **Power Kids, KreşCloud, Anaokula** | Aylık/haftalık menü veli ekranında; bazıları besin değeri öne çıkarıyor (Power Kids yapay zekâ ile menü üretimi vaat ediyor — OKSİS için kapsam dışı). | **[K]** (arama özeti) | [powerkidsapp.com](https://powerkidsapp.com/) · [krescloud.com](https://krescloud.com/blog/anaokulu-kres-uygulamasi) · [anaokula.com](https://anaokula.com/) |
| Okulsis, Kolay Okul, Kidokit, Parentwiser | Menü özelliğine dair kaynak bulunamadı. | **[T]** | — |

**Örüntü:** Okul öncesi uygulamalarında menü neredeyse **standart** ve hep "çocuk bugün ne kadar yedi" kaydıyla birlikte geliyor. K-12 sistemlerinde (K12NET) menü daha çok **bilgilendirme**, yemek kaydı ise yan özellik.

### 2.3 Özel okullar menüyü bugün nasıl yayınlıyor

| Okul | Biçim | Öğünler / kademe | Kalori / alerjen | Kaynak |
|---|---|---|---|---|
| Bilişim Koleji | Aylık; kampüs panosu + K12NET veli portalı | Anaokulu 3 öğün (kahvaltı, öğle, ikindi), ilkokul 2 (kahvaltı, öğle), ortaokul/lise 1 (öğle); porsiyon yaşa göre | **Öğün başına kalori** var, **alerjen içerenler işaretli**; menüyü **diyetisyen** hazırlıyor; kendi mutfağı; yemek eğitim ücretine dâhil | [bilisim.k12.tr](https://bilisim.k12.tr/yemek-listesi/) |
| Akdeniz Boğaziçi Koleji | Aylık, sitede **HTML tablo** | Sabah kahvaltısı, öğle, ikindi kahvaltısı; "Anaokulu, İlkokul & Ortaokul" **tek liste**; tatil günleri işaretli | Yok | [akdenizbogazicikoleji.k12.tr](https://akdenizbogazicikoleji.k12.tr/yemek-menusu/) |
| Doğa Koleji (Eskişehir) | Aylık **PDF + Excel** bağlantısı | Dosya içinde | Dosya içinde | [dogakoleji.k12.tr](https://www.dogakoleji.k12.tr/eskisehir-kampusu/yemek-listesi-1384-51-58) |
| TED Kayseri | Aylık **PDF** | Dosya içinde | Dosya içinde | [tedkayseri.k12.tr](https://www.tedkayseri.k12.tr/aylik-yemek-listesi/) |
| Arel (anaokulu) | **Excel'den üretilmiş PDF**, yatay A4, tarih sütunlu, "grup" satırlı ("Kahvaltı" vb.) | Anaokulu için ayrı dosya | (içerik okunamadı) | [arel.k12.tr — anaokulu PDF](https://www.arel.k12.tr/files/files/yemeklistesi/yemeklistesi_anaokulu.pdf) |

**Çıkarımlar:**
- Menünün doğduğu yer **Excel** (diyetisyen/catering firması hazırlar) → okul PDF'e çevirip siteye ve panoya koyar. OKSİS'e giriş bu Excel'den elle aktarım olacak; içe aktarma ileride değer taşır.
- **Kademe farkı gerçek**: anaokulu daha çok öğün (kahvaltı + ikindi) ve farklı porsiyon/içerik alır; bazı okullar ise tek liste yayınlar. Model ikisini de taşımalı.
- **Kalori ve alerjen** iyi okullarda menünün parçası; zorunlu alan yapılmamalı ama yeri olmalı.
- Okul menüyü **dışarıya da** (web sitesi, pano) yayınlamak zorunda kalmaya devam edecek → yazdırılabilir çıktı değer taşır.

---

## 3. Okul İşleyişi Perspektifi

| Konu | Gerçeklik (eğitimci gözlemi) | Analize etkisi |
|---|---|---|
| **Menüyü kim hazırlar** | Kendi mutfağı olan okulda **diyetisyen** (kadrolu ya da danışman); dışarıdan yemek alan okulda **catering firmasının diyetisyeni**. Okul içinde sorumlu genellikle **idari işler müdürü / müdür yardımcısı**, küçük okulda **müdür** ya da **sekreterlik**. Anaokulunda **anaokulu müdürü/koordinatörü** ayrıca bakar. | Menüyü hazırlayan çoğu zaman **sistem dışı** kişi; sisteme **okul personeli** girer. Dış firma hesabı MVP'de gereksiz. |
| **Sıklık** | **Aylık** yaygın (ay başından birkaç gün önce); bazı okullar **haftalık** ya da **4 haftalık döngüsel** menü kullanır. Ay içinde **son dakika değişikliği** olur (malzeme gelmedi, mutfak arızası, etkinlik günü). | Veri **gün** bazlı tutulmalı; yayın birimi esnek (ay/hafta/tarih aralığı). Yayın sonrası düzenleme şart. |
| **Onay** | Pratikte diyetisyen hazırlar → idare göz atar → yayın. Resmî iki aşamalı onay nadirdir; büyük kampüslerde müdür onayı istenebilir. **[T]** | MVP'de **taslak → yayın** yeterli; onay adımı karar konusu (§9 Y-8). |
| **Okul öncesi farkı** | Günde 2–3 öğün (kahvaltı, öğle, ikindi); veli en çok **"yedi mi?"** sorar; öğretmen gün sonunda bunu söyler/yazar. Alerji takibi daha kritik (çocuk kendini koruyamaz). | Menü grubu + anaokulu öğünleri MVP'de; "yedi/yemedi" Faz 2 (§7). |
| **İlkokul** | Kahvaltı + öğle; beslenme çantası kültürü sürer (bazı okullar "beslenme günleri" listesi yayınlar). | "Beslenme listesi" ayrı ihtiyaç olabilir — açık soru (§10). |
| **Ortaokul / lise** | Tek öğün (öğle); öğrenci menüye kendisi bakar, veli ilgisi düşer. | Öğrenci görünümü değerli; bildirim gürültüsü yaratmamalı. |
| **Alerji / özel diyet** | Kayıtta veli sağlık formuyla bildirir; bilgi **revir/hemşire, sınıf öğretmeni, mutfak** arasında kâğıtla dolaşır. Çölyak, laktoz, fındık/yer fıstığı, vejetaryen ve dinî tercihler en sık. Mutfak "alternatif tabak" hazırlar. | Öğrenci alerji kaydı **sağlık verisi** (KVKK özel nitelikli) → MVP'de **yemek tarafında alerjen etiketi**, **öğrenci tarafında kayıt yok**; öğrenci-alerjen eşleştirmesi Faz 2 (§9 Y-7, Y-12). |
| **Velinin en çok merak ettiği** | 1) Bugün/yarın ne var; 2) çocuğumun alerjeni var mı; 3) yeterli/sağlıklı mı (kalori, sebze, kızartma); 4) (anaokulu) yedi mi; 5) menü değişti mi. | MVP önceliği: **bugün kartı + alerjen ikonu + değişiklik bildirimi**. |
| **Öğretmenin ihtiyacı** | Nöbetçi/anaokulu öğretmeni yemek saatinde menüyü bilir; alerjili öğrenciyi tanır. | Öğretmen de menüyü görür (salt okunur). |

---

## 4. Mevzuat Notları (yalnız not — kapsamı daraltmaz, karar Faruk'ta)

- **Alerjen bilgilendirme:** *Türk Gıda Kodeksi Gıda Etiketleme ve Tüketicileri Bilgilendirme Yönetmeliği* uyarınca **okul dâhil toplu tüketim yerlerinde**, 1 Ocak 2020'den beri sunulan yemekte 14 alerjenden biri varsa menüde ya da görünür yerde belirtilmesi gerekiyor. Yükümlülük **gıda işletmecisinde** (okul mutfağı/catering); OKSİS bunun için kanal olabilir ama **yasal bilgilendirmenin yerine geçtiğini iddia etmemeli**. Madde numarası **doğrulanmadı**. Kaynak: [Medikal Akademi haberi (2020)](https://www.medikalakademi.com.tr/alerjiye-neden-olabilen-14-besinin-menuelere-yazilmasi-zorunlulugu-basladi/), yönetmelik metni: [Lexpera](https://www.lexpera.com.tr/mevzuat/yonetmelikler/turk-gida-kodeksi-gida-etiketleme-ve-tuketicileri-bilgilendirme-yonetmeligi) (6/4/2024 değişikliği arama özetinden; **doğrulanmadı**).
- **14 alerjen listesi** (yukarıdaki kaynağa göre): gluten içeren tahıllar, kabuklular, yumurta, balık, yer fıstığı, soya, süt ve süt ürünleri, sert kabuklu meyveler, kereviz, hardal, susam, kükürt dioksit ve sülfitler, acı bakla (lupin), yumuşakçalar. Liste AB 1169/2011 Ek II ile aynı görünüyor **[K]**.
- **Özel okul yemek ücreti:** *Özel Öğretim Kurumları Yönetmeliği*'nin 3 Ocak 2025 değişikliğiyle (md. 53 ücret tespiti, md. 55 "diğer hizmetler" ücretinin kurum hesabına yatırılması) yemek dâhil tüm hizmetler ücret tespitine giriyor; MEB velilerin yemek için üçüncü kişilerle doğrudan anlaşmamasını istiyor. **Ücret MVP dışı** olduğu için etkisi yok; ücret fazında bakılmalı. Kaynak: [MEB haberi, 27 Mart 2025](https://www.meb.gov.tr/mebden-ozel-okullarin-yemek-kiyafet-kirtasiye-odemelerine-yakin-takip/haber/36752/tr) (madde metni **doğrulanmadı**).
- **Kantin genelgesi (2016):** kantinde satılabilecek ürünleri sınırlar (resmî + özel okullar); yemekhane menüsünü değil kantini kapsar. Kaynak: [MEB haberi, 7 Nisan 2016](https://www.meb.gov.tr/okul-kantinlerinde-satisi-yapilacak-urunler-icin-yeni-kriterler/haber/10814/tr) (güncel yürürlüğü **doğrulanmadı**).
- **Beslenme rehberleri:** MEB/Sağlık Bakanlığı "Beslenme Dostu Okul" programı ve [Okullarda Yiyecek ve İçecek Standartları](https://hsgm.saglik.gov.tr/depo/birimler/saglikli-beslenme-ve-hareketli-hayat-db/Dokumanlar/Beslenme/Okullarda_Yiyecek_ve_ecek_Standartlar.pdf) menü **içeriğine** dair rehberlik eder; yazılıma yükümlülük getirmez.
- **72 saat numune:** okul sitesi Sağlık Bakanlığı kuralı olarak anıyor ([Bilişim Koleji](https://bilisim.k12.tr/yemek-listesi/)); dayanak **doğrulanmadı**. Numune kaydı kapsam dışı.
- **KVKK:** öğrencinin alerji/diyet bilgisi **sağlık verisidir → özel nitelikli kişisel veri** ([6698 sayılı KVKK md. 6](https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf)). Dinî gerekçeli diyet tercihi de dolaylı olarak "din/inanç" verisi sayılabilir **[T]**. Bu nedenle öğrenci tarafı ayrı karar (§9 Y-12). OKSİS'te `ConsentType.HealthData` rıza türü zaten var (§5).

---

## 5. OKSİS'te Bugünkü Durum (kod taraması, 2026-10-15)

| Konu | Bulgu | Dosya |
|---|---|---|
| Modül kataloğu (UI) | `cafeteria` → "Yemekhane — Yemek menüsü ve yemekhane yoklaması", `tag: "free"`. Ayarlar sekmesinde ikon `coffee`. Ekran/rota **yok**. | `oksis-ui-yemek-menusu/packages/core/src/modules/constants.ts`, `apps/web/features/settings/module-tab.tsx` |
| Modül tohumlama (API) | `cafeteria` **`IsEnabled: true`**, `Tier: Standard` — yeni okulda **varsayılan açık**. UI mock'unda ise kapalı. | `oksis-api-yemek-menusu/src/Oksis.Application/Modules/Schools/Events/SchoolCreated/SeedDefaultModuleConfigsHandler.cs` |
| Menü / yemek / alerjen varlığı | **Yok.** | — |
| Öğrenci sağlık/alerji verisi | **Yok.** Yalnız `ConsentType.HealthData` rıza türü var. | `Oksis.Domain/Modules/Users/Enums/ConsentType.cs` |
| Kademe | `EducationLevel` (Preschool, Primary, …) — UI sözlüğünde `schoolStage`. | `Oksis.Domain/Modules/Academics/Enums/EducationLevel.cs` |
| Tatil günleri | `Holiday` (sezona bağlı, tarih aralıklı) — menü takviminde tatil gösterimi için kullanılabilir. | `Oksis.Domain/Modules/Schools/Entities/Holiday.cs` |
| Dosya | `StoredFile` / `FileAttachment` (ClamAV taramalı) — PDF eki seçeneği için hazır altyapı. | `Oksis.Domain/Modules/Documents/Entities/` |
| Bildirim | Uygulama içi + push hattı, sessiz saat, alıcı tercihi; yeni `NotificationKind` eklenmesi gerekir. | `Oksis.Domain/Modules/Notifications/` |
| Kampüs | Ayrı kampüs kavramı **yok** (okul = tenant). Çok kampüslü okul ayrı tenant'tır **[T]**. | — |

> **Çelişki / risk:** Katalog açıklaması "yemekhane yoklaması" vaat ediyor, API varsayılanı modülü **açık** başlatıyor, ama hiçbir ekran yok. MVP yoklamayı içermezse açıklama güncellenmeli; modül açık + menü boş durumda veli ne görecek kararı gerekir (BR-YM-19, §9 Y-14).

---

## 6. Roller ve Kullanım Senaryoları

### 6.1 Roller

| Rol | Menüdeki yeri | Not |
|---|---|---|
| **İdare** (okul yöneticisi + müdür yardımcısı) | Menüyü girer, düzenler, yayınlar; menü gruplarını, öğünleri, yemek kataloğunu yönetir. | Varsayılan yetkili. |
| **Menü sorumlusu personel** (idari işler, sekreterlik, anaokulu koordinatörü) | İdarenin izin verdiği personel aynı işleri yapar. | Sekreter bugün sistem rolü değil (sözlük notu) → **izinle** çözülmeli (§9 Y-9). |
| **Diyetisyen / catering firması** | Sistem dışı; menüyü Excel/PDF olarak okula iletir. | MVP'de hesap yok. |
| **Öğretmen** | Salt okunur görür (anasayfa kartı + ayrıntı). Anaokulu öğretmeni için ileride "yedi/yemedi" girişi. | |
| **Veli** | Çocuğunun menü grubundaki menüyü görür; çoklu çocukta çocuk başına doğru menü. Bildirim alır. | |
| **Öğrenci** | Kendi menü grubunu görür (ortaokul/lise için asıl kullanıcı). | Anaokulu/ilkokulda öğrenci hesabı genelde kullanılmaz **[T]**. |

### 6.2 Senaryolar

| Kod | Senaryo | Akış |
|---|---|---|
| **S-1** | Aylık menü girişi | Müdür yardımcısı diyetisyenin Excel'ini açar; OKSİS'te Kasım'ı seçer; "İlkokul–Lise" grubunda her gün için öğle yemeğine 4 yemek yazar (yazdıkça katalogdan öneri gelir, alerjen etiketleri katalogdan gelir); tatil günleri gri ve kilitli/uyarılı görünür; taslak kaydeder, ertesi gün kontrol eder, **Yayınla**. Veliler "Kasım menüsü yayınlandı" bildirimi alır. |
| **S-2** | Hızlı doldurma | Aynı hafta düzeni tekrarlanıyorsa "Geçen haftayı kopyala" / "Günü şu günlere kopyala". 4 haftalık döngüsel menü kullanan okul, bir ayı önceki aydan kopyalayıp düzeltir. |
| **S-3** | Anaokuluna ayrı menü | "Anaokulu" grubu kahvaltı + öğle + ikindi öğünleriyle tanımlı; anaokulu öğrencilerinin velileri yalnız bu grubu görür. |
| **S-4** | Son dakika değişikliği | Yayındaki menüde yarının balığı tedarik edilemedi; idare yarının öğle yemeğini değiştirir. Değişiklik yakın tarihe (ör. bugün/yarın) düştüğü için ilgili grubun velilerine "Yarınki menü değişti" bildirimi gider; menüde gün "güncellendi" işareti taşır. Ay sonundaki bir günü değiştirmek bildirim üretmez. |
| **S-5** | Veli sabah bakar | Veli anasayfada çocuğunun kartında "Bugünün menüsü"nü görür; dokununca haftalık görünüm açılır; yemeklerin yanında alerjen ikonları ("Süt", "Gluten") vardır. |
| **S-6** | Alerjili çocuğun velisi | Veli menüde "Sert kabuklu meyve" içeren günleri hızla görmek ister → ayrıntıda alerjen ikonu/listesi (MVP); "çocuğumun alerjeni içeren günleri vurgula" (Faz 2, öğrenci alerji kaydı gerektirir). |
| **S-7** | Okul sitesine/panoya asma | İdare ay görünümünü yazdırır/PDF'e alır ve panoya asar (MVP: yazdırılabilir görünüm). |
| **S-8** | Menü olmayan gün | Tatil, yarım gün, gezi günü: o gün "Menü yok" ya da "Tatil" görünür; boş gün veli tarafında "henüz girilmedi" ile karışmaz. |
| **S-9** | (Faz 2) Yedi/Yemedi | Anaokulu öğretmeni öğle sonrası sınıf listesinde "hepsi / yarım / az / yemedi" işaretler; veli gün sonunda görür. |

---

## 7. Önerilen Kapsam

### 7.1 MVP (öneri)

| # | Özellik | Gerekçe |
|---|---|---|
| M-1 | **Menü grubu**: okul en az bir grup tanımlar (varsayılan tek grup "Tüm okul"); grup kademelere (isteğe bağlı seviye) eşlenir; her öğrenci tam olarak bir gruba düşer. | Anaokulu/ilkokul/lise farkı (K-4, §2.3). Tek liste yayınlayan okul için sıfır ek iş. |
| M-2 | **Öğün tanımı** (okul düzeyi): ad + sıra; varsayılanlar Kahvaltı · Öğle Yemeği · İkindi Kahvaltısı. Grup hangi öğünleri sunduğunu seçer. | Kademe başına öğün sayısı farklı (Bilişim örneği). |
| M-3 | **Yemek kataloğu**: yemek adı + isteğe bağlı **alerjen etiketleri** (14'lük sabit liste) + isteğe bağlı kategori (Çorba, Ana yemek, Yardımcı, Tatlı/Meyve, İçecek). Girişte otomatik tamamlama; katalog girişle kendiliğinden büyür. | K12NET'in de modeli (K-2); alerjenin bir kez girilip her gün tekrarlanması; veri kalitesi. |
| M-4 | **Gün × öğün girişi** (web): ay/hafta ızgarası, öğün başına sıralı yemek listesi + isteğe bağlı öğün notu ve **öğün kalorisi** (sayı). | Aylık menü gerçekliği; kalori iyi okullarda var. |
| M-5 | **Kopyalama**: günü/haftayı/ayı başka tarihlere kopyala (hedefte dolu gün varsa onaylı üzerine yazma). | Giriş yükünü dramatik düşürür; döngüsel menüyü karşılar. |
| M-6 | **Taslak → Yayın** (tarih aralığı ve grup bazında); yayındaki günler düzenlenebilir, her düzenleme izlenir (kim/ne zaman). | Veli yarım menü görmez; son dakika değişikliği gerçekliği. |
| M-7 | **Tatil entegrasyonu**: `Holiday` günleri takvimde işaretli; tatil gününe menü girmek uyarı verir. | Boş günle tatil karışmaz (S-8). |
| M-8 | **Görüntüleme** (web + mobil): veli (çocuk başına), öğrenci, öğretmen, idare — **anasayfa "Bugünün menüsü" kartı** (hafta sonu/tatilde bir sonraki okul günü), haftalık liste, ay görünümü, alerjen ikonları + açıklaması. | Velinin 1 numaralı sorusu; PDF'e göre asıl fark. |
| M-9 | **Bildirim**: (a) yayında grup velilerine/öğrencilerine tek bildirim; (b) yayındaki **yakın tarihli** (bugün/yarın — eşik karar) gün değişince "menü değişti" bildirimi. Günlük "bugünün menüsü" push'u **yok**. | Değişiklik bilgisi değerli, günlük push gürültü. |
| M-10 | **Yazdırılabilir görünüm** (web, ay/hafta, grup başına). | Pano ve web sitesi yayını sürecek (§2.3). |
| M-11 | **İzin**: `cafeteria.manage` (giriş/yayın/katalog), görüntüleme modül açıkken rol bazlı. | Personel yetkilendirme (Y-9). |

### 7.2 Sonraki faz (öneri, öncelik sırasıyla)

| # | Özellik | Neden sonra |
|---|---|---|
| F-1 | **Excel şablonla içe aktarma** (OKSİS şablonu; serbest biçimli diyetisyen Excel'i değil) | Gerçek değer yüksek ama şablon tasarımı + hata raporu iş yükü büyük; pilot geri bildirimiyle öne çekilebilir. |
| F-2 | **Yemekhane yoklaması / tüketim kaydı** (anaokulu: hepsi/yarım/az/yemedi; K12NET "Yedi/Yemedi") + veli günlük görünümü | Katalogdaki "yemekhane yoklaması" vaadi; anaokulu için güçlü satış argümanı; ama yoklama ekranı + öğretmen iş yükü + raporlama ayrı dilim. `rollCall`/devamsızlıkla karışmamalı. |
| F-3 | **Öğrenci alerji/diyet kaydı** + menüde "çocuğunuz için uyarı" + öğretmen/mutfak listesi | Sağlık verisi (KVKK md. 6), rıza, erişim denetimi gerektirir; ayrı analiz. |
| F-4 | **Alternatif / diyet menüsü** (glütensiz, vejetaryen) — gün başına ikinci tabak | F-3 ile birlikte anlamlı. |
| F-5 | Yemek **görseli**, besin değeri ayrıntısı (protein/yağ/karbonhidrat) | K12NET'te görsel var; MVP için şart değil. |
| F-6 | **PDF dışa aktarma** dosyası + okul sitesine **herkese açık bağlantı/gömme** | Herkese açık bağlantı kimliksiz erişim demek → güvenlik kararı. |
| F-7 | **Döngüsel menü şablonu** (4 haftalık döngü tanımla, aylara uygula) | M-5 kopyalama çoğu ihtiyacı karşılar. |
| F-8 | **Beslenme listesi** (ilkokul: evden getirilecek beslenme günleri) | Ayrı ihtiyaç mı, menünün parçası mı — açık soru. |

### 7.3 Kapsam dışı (öneri)

- **Yemek ücreti, abonelik, yemekhane kaydı, öğün bazlı ücretlendirme** → Finans modülü işi (ve ÖÖK Yönetmeliği md. 53/55 alanı).
- **Kantin satış, ön ödemeli bakiye, kart/QR/parmak izi ile geçiş** (K12NET kafeterya/kantin modülleri, K-7).
- **Stok, satın alma, maliyet, porsiyon sayımı, numune kaydı** (mutfak operasyonu).
- **Catering firması / diyetisyen portalı** (dış kullanıcı hesabı).
- **Yapay zekâ ile menü üretimi/öneri** (pilot kapsamı dışı — proje kuralı).
- **Veli menü puanlama/yorum** — şikâyet kanalına dönüşme riski; ihtiyaç kanıtı yok.
- **e-Okul** ile ilgili her şey (e-Okul'da genel menü kavramı yok).

---

## 8. İş Kuralı Adayları (BR-YM-xx)

> Aday kurallar; Faruk'un §9 kararlarına göre kesinleşir. Köşeli parantez = karara bağlı.

**Yapı**
- **BR-YM-01** Menü verisi **gün** bazlıdır: (menü grubu, tarih, öğün) üçlüsü başına en çok bir öğün kaydı; öğün kaydı sıralı yemek listesi + isteğe bağlı not + isteğe bağlı kalori taşır.
- **BR-YM-02** Okulda her zaman en az bir menü grubu vardır; son grup silinemez. Varsayılan grup "Tüm okul"dur ve tüm kademeleri kapsar.
- **BR-YM-03** Bir kademe [ya da seviye] **en çok bir** gruba eşlenir; böylece her öğrencinin tek bir menüsü vardır. Eşlenmemiş kademe varsa yönetim ekranı uyarır, o öğrencilerin velisi "menü tanımlı değil" görür.
- **BR-YM-04** Öğrencinin grubu **güncel şubesinin seviyesinden/kademesinden** türetilir (kayıtta saklanmaz); şube değişince menü kendiliğinden değişir.
- **BR-YM-05** Öğünler okul düzeyinde tanımlanır; bir öğün, geçmişte kullanılmışsa silinmez, **pasifleştirilir**.
- **BR-YM-06** Yemek kataloğunda ad okul içinde tekildir (büyük/küçük harf ve baş/son boşluk duyarsız). Katalogdaki alerjen değişikliği **tüm** günlere yansır (yemek referansla bağlıdır) — geçmiş günler dâhil.
- **BR-YM-07** Alerjen etiketleri sabit 14'lük listeden seçilir; serbest alerjen yok (gerekirse yemek notuna yazılır). "Etiket yok" ≠ "alerjen içermez": arayüz, etiketsiz yemeği "alerjen bilgisi girilmemiş" olarak değil, yalnız etiket göstermeyerek sunar ve menü altında sabit uyarı taşır (BR-YM-17).

**Yaşam döngüsü**
- **BR-YM-08** Gün durumu: **Taslak** → **Yayında**. Taslak gün yalnız menü yetkilisine görünür.
- **BR-YM-09** Yayın, seçilen grup + tarih aralığındaki **tüm taslak günleri** birlikte yayınlar; aralıkta hiç dolu gün yoksa yayın yapılamaz. Boş günler yayında "menü yok" olarak görünür.
- **BR-YM-10** Yayındaki gün düzenlenebilir; her düzenleme kim/ne zaman/önce-sonra olarak izlenir. Yayından geri çekme [var/yok — Y-15].
- **BR-YM-11** Geçmiş tarihli (bugünden önce) gün [düzenlenemez / yalnız idare düzenler — öneri: düzenlenemez, yanlış veri kalmasın diye yalnız not eklenebilir].
- **BR-YM-12** Menü sezona bağlı değildir, **tarihe** bağlıdır; ancak arşivlenmiş sezonun tarih aralığındaki günler salt okunurdur.

**Takvim**
- **BR-YM-13** `Holiday` kapsamındaki günlere menü girişi uyarı verir ama engellemez (yarım gün tatil, etkinlikli gün olabilir); hafta sonu varsayılan olarak ızgarada gizlidir [hafta sonu eğitim veren okul için açılabilir — açık soru].
- **BR-YM-14** "Bugünün menüsü" kartı: bugün okul günü ve yayında menü varsa bugünü; değilse **bir sonraki yayındaki günü** tarihiyle ("Pazartesi'nin menüsü") gösterir; 7 gün içinde yayında gün yoksa kart gizlenir.

**Görünürlük**
- **BR-YM-15** Veli, her çocuğu için o çocuğun grubunun **yalnız yayındaki** günlerini görür; çoklu çocukta çocuklar aynı gruptaysa menü bir kez, farklıysa çocuk başına gösterilir.
- **BR-YM-16** Öğrenci kendi grubunu, öğretmen [tüm grupları — öneri], idare tüm grupları (taslak dâhil) görür.
- **BR-YM-17** Alerjen gösterimi bilgi amaçlıdır; menü görünümünde sabit not: "Alerjen bilgisi okulun beyanıdır; çocuğunuzun özel durumu için okulla görüşün." (metin önerisi; hukuki dil teyidi gerekir).

**Bildirim**
- **BR-YM-18** Yayın işlemi, etkilenen grubun velilerine ve öğrencilerine **tek** bildirim üretir ("Kasım menüsü yayınlandı"); aynı aralığın yeniden yayını/düzenlemesi yeniden "yayınlandı" bildirimi üretmez.
- **BR-YM-19** Yayındaki bir gün **bugün veya yarın** [eşik karar Y-10] içinde değişirse o grubun alıcılarına "menü değişti" bildirimi gider; aynı gün için 30 dk [öneri] içindeki ardışık düzenlemeler tek bildirimde birleşir. Daha uzak tarihli değişiklik bildirim üretmez, yalnız günde "güncellendi" işareti.
- **BR-YM-20** Bildirimler mevcut hattın kurallarına uyar (uygulama içi kapatılamaz, push sessiz saate ve alıcı tercihine tabi). Günlük otomatik "bugünün menüsü" push'u yoktur.

**Modül**
- **BR-YM-21** `cafeteria` modülü kapalıysa menü yüzeyleri (anasayfa kartı, rota, bildirimler) görünmez; veri silinmez.
- **BR-YM-22** Modül açık ama yayında hiç gün yoksa veli/öğrenci tarafında menü girişi gösterilmez [ya da boş durum — Y-14].

**Yetki**
- **BR-YM-23** Giriş, yayın, katalog, grup ve öğün yönetimi `cafeteria.manage` iznini ister; varsayılan olarak idarede. İzin verilen personel aynı işlemleri yapar; ayrı "onaylayıcı" yoktur [Y-8].

---

## 9. Faruk'un Vermesi Gereken Kararlar

| Kod | Karar | Seçenekler | Öneri ve gerekçe |
|---|---|---|---|
| **Y-1** | **Modül adı ve kapsamı** | (a) `cafeteria` "Yemekhane" adı altında menü ilk özellik, yoklama sonra; (b) ayrı "Yemek Menüsü" modülü, "Yemekhane" (yoklama/ücret) ayrı. | **(a).** Katalogda anahtar zaten var; okul tek "Yemekhane" anahtarını açar/kapatır. Açıklama MVP'de "Yemek menüsü" olarak güncellenir, yoklama gelince eklenir. |
| **Y-2** | **Giriş modeli** | (a) Yalnız **dosya** (PDF/görsel yükle — Ekid+ gibi); (b) **yapılandırılmış** (gün × öğün × yemek); (c) yapılandırılmış + isteğe bağlı PDF eki. | **(b).** "Bugünün menüsü" kartı, alerjen ikonu ve "değişti" bildirimi ancak yapılandırılmış veriyle olur — PDF bugünkü durumun dijital kopyası olur, fark yaratmaz. (c) iki gerçeklik kaynağı yaratır (PDF'te başka, ekranda başka). |
| **Y-3** | **Yayın birimi** | (a) Ay; (b) Hafta; (c) Serbest tarih aralığı (veri gün bazlı). | **(c)**, arayüzde "Bu ayı yayınla / Bu haftayı yayınla" kısayollarıyla. Okulların bir kısmı aylık, bir kısmı haftalık çalışıyor. |
| **Y-4** | **Kademe farkı** | (a) Okulda tek menü; (b) Kademe başına sabit menü; (c) **Menü grubu** (okul tanımlar, kademeye/seviyeye eşler). | **(c)**, varsayılan tek grup. Tek liste yayınlayan okul (Akdeniz Boğaziçi) de, üç ayrı menülü okul (Bilişim) de karşılanır. Şube bazında eşleme **önerilmez** (gereksiz karmaşa). Seviye düzeyinde eşleme gerekli mi — açık soru §10. |
| **Y-5** | **Öğünler** | (a) Sabit üç öğün; (b) Okul tanımlı liste. | **(b)**, varsayılan üç öğünle gelir. Kuşluk/ara öğün, etüt akşam yemeği gibi okul farkları var. |
| **Y-6** | **Besin değeri** | (a) Yok; (b) Öğün başına isteğe bağlı kalori; (c) Yemek başına kalori + makrolar. | **(b).** İyi okullarda öğün kalorisi menüde var; yemek başına besin değeri diyetisyen yükü ve veri kalitesi sorunu. |
| **Y-7** | **Alerjen** | (a) Yok (not alanına yazılır); (b) Yemek kataloğunda 14 alerjen etiketi, isteğe bağlı; (c) (b) + girişte zorunlu "alerjen kontrol edildi" onayı. | **(b).** Velinin 2 numaralı sorusu; mevzuat toplu tüketim yerinde alerjen bilgisi istiyor (§4). (c) idarenin yükünü ve sorumluluk algısını artırır. |
| **Y-8** | **Onay akışı** | (a) Yok: yetkili girer ve yayınlar; (b) Taslak → onaya gönder → idare onaylar → yayın. | **(a).** Menüyü hazırlayan zaten sistem dışında (diyetisyen); içeride tek kişi girer. Onay ihtiyacı pilotta çıkarsa duyuru moderasyonuna benzer okul ayarı olarak eklenir. |
| **Y-9** | **Kim girer** | (a) Yalnız idare; (b) idare + `cafeteria.manage` izni verilen personel; (c) + dış kullanıcı (diyetisyen/firma). | **(b).** Gerçekte işi idari işler/sekreterlik yapıyor; dış hesap KVKK ve güvenlik yükü getirir. |
| **Y-10** | **Bildirim** | (a) Hiç; (b) Yalnız yayında; (c) Yayında + yakın tarihli değişiklikte; (d) (c) + her sabah "bugünün menüsü" push'u. | **(c)**, yakınlık eşiği **bugün + yarın**. (d) her gün yüzlerce push = bildirim yorgunluğu; kart bu ihtiyacı karşılar. |
| **Y-11** | **Görenler** | Veli · öğrenci · öğretmen · idare; personel? | Hepsi görür (öğretmen tüm grupları). Öğrenci görünürlüğü anaokulu/ilkokulda pratikte boş kalır, sorun değil. |
| **Y-12** | **Öğrenci alerji/diyet kaydı** | (a) MVP'de; (b) Faz 2, ayrı analizle; (c) Hiç (okul kâğıtla sürdürür). | **(b).** Sağlık verisi = özel nitelikli veri; rıza, erişim listesi (öğretmen/mutfak/revir), denetim izi gerektirir. Menüdeki alerjen etiketi bu kayıt olmadan da tek başına değer üretir. |
| **Y-13** | **Yemekhane yoklaması (yedi/yemedi)** | (a) MVP'de; (b) Faz 2 (anaokulu öncelikli); (c) Hiç. | **(b).** K12NET ve tüm okul öncesi uygulamalarında var; anaokulu velisi için güçlü. Ama öğretmen giriş ekranı + veli raporu ayrı dilim; menü olmadan anlamsız olduğundan menüden sonra gelir. |
| **Y-14** | **Modülün varsayılan durumu** | API'de yeni okul için `cafeteria` **açık**. (a) Açık kalsın, menü yoksa veli tarafı gizli; (b) Varsayılan kapalı olsun. | **(b)** — modülü kullanmayan okulda boş menü girişi oluşmaz; açmak bilinçli idare eylemi olur. (a) da BR-YM-22 ile güvenli. Mevcut okulların durumu ayrıca kontrol edilmeli. |
| **Y-15** | **Yayından geri çekme** | (a) Yok (yalnız düzenleme); (b) Gün/aralık geri çekilebilir, veli tarafından kalkar. | **(b)**, bildirimsiz. Yanlış aya yayın gibi hatalar için gerekli; duyurudaki `withdraw` kavramına benzer. |
| **Y-16** | **Yazdırma** | (a) MVP'de tarayıcı yazdırma görünümü; (b) MVP'de PDF dosyası üretimi; (c) Faz 2. | **(a).** Pano/site ihtiyacını sıfıra yakın maliyetle karşılar; sunucu tarafı PDF Faz 2. |
| **Y-17** | **Platform eşitliği** | (a) Giriş web'de, görüntüleme web + mobil; (b) giriş mobilde de. | **(a).** Aylık ızgara girişi telefonda verimsiz; idare için mobilde salt okunur + belki tek gün düzeltme (açık soru). |
| **Y-18** | **Excel içe aktarma zamanlaması** | (a) MVP; (b) Faz 2 başı; (c) ihtiyaç kanıtlanınca. | **(b).** Kopyalama + katalog otomatik tamamlamayla aylık giriş ~30–45 dk **[T]**; pilotta ölçülsün, uzunsa öne çekilsin. |

---

## 10. Açık Sorular ve Riskler

### Açık sorular
1. **Pilot okul gerçekliği:** Altınay'da (ya da pilot okulda) yemek kendi mutfağından mı, catering mi? Menüyü kim hazırlıyor, hangi biçimde (Excel/PDF/Word) geliyor, kaç grup var? Bir örnek menü dosyası istenmeli — giriş tasarımını ve Excel şablonu kararını doğrudan etkiler.
2. **Grup eşleme düzeyi:** Kademe yeterli mi, yoksa seviye (ör. 1–2. sınıf ayrı porsiyon) gerekir mi?
3. **Hafta sonu / etüt / yatılı:** Hafta sonu eğitim, akşam etüdü yemeği, pansiyon (yatılı özel okul) var mı? Öğün tanımı esnekliği bunu karşılıyor ama ızgara hafta sonunu gizliyor.
4. **Beslenme listesi** (ilkokulda evden gelen beslenme günleri) bu modülün parçası mı, duyuru ile mi çözülür?
5. **Öğretmen yemeği** (personel menüsü) ayrı mı? Genelde aynı; ayrı ise ayrı grup olarak tanımlanabilir mi (grup → personel eşlemesi)?
6. **Geçmiş menü erişimi:** Veli geçmiş ayları görebilsin mi, ne kadar geri? (Öneri: sezon içi tamamı.)
7. **Alerjen uyarı metni** (BR-YM-17) için hukuki dil teyidi gerekir mi?
8. **Kalori birimi ve yemek başına porsiyon** bilgisi (K12NET'te porsiyon var) isteniyor mu?
9. **Çok kampüslü okullar:** Kampüs = ayrı tenant varsayımı doğru mu? Değilse menü grubu kampüs ayrımını da taşımak zorunda kalır.

### Riskler
| Risk | Etki | Azaltma |
|---|---|---|
| **Çift kayıt yükü:** okul menüyü yine Excel'de hazırlayıp OKSİS'e elle girer; zaman alırsa vazgeçer, PDF'e döner. | Modül boş kalır. | Katalog + kopyalama (M-3, M-5); pilotta süre ölçümü; F-1 Excel'i öne çekme seçeneği. |
| **Alerjen yanlış/eksik etiketi** veliye güvence algısı yaratır; bir alerji olayında "sistemde yazmıyordu" denir. | Hukuki/itibar. | Etiket isteğe bağlı + sabit uyarı (BR-YM-17); "etiket yok ≠ alerjen yok"; yükümlülük gıda işletmecisinde (§4). |
| **Katalogdaki alerjen değişikliğinin geçmiş günlere yansıması** (BR-YM-06) geçmiş kaydın "o gün ne söylendi" bilgisini bozar. | Denetimde tutarsızlık. | Teknik analizde yayın anı görüntüsü (snapshot) saklama seçeneği değerlendirilmeli. |
| **Bildirim gürültüsü:** sık değişiklik yapan okulda "menü değişti" bildirimleri çoğalır. | Velinin bildirimi kapatması. | Yakınlık eşiği + birleştirme (BR-YM-19); günlük push yok. |
| **Katalog açıklaması "yemekhane yoklaması" vaat ediyor**, API'de modül varsayılan açık (§5). | Beklenti/boş ekran. | Y-1, Y-14 kararları; açıklama güncellemesi. |
| **Sağlık verisi kayması:** idare alerjili öğrenci adlarını öğün notuna yazmaya başlar ("Ali'ye laktozsuz"). | KVKK özel nitelikli veri, geniş kitleye görünür. | Not alanında uyarı metni; öğrenci adı içeren not yazılmaması kılavuzu; F-3'te doğru yer. |
| **Belge çelişkisi:** `mvp-kapsami.md` yemekhaneyi "Sprint 5+" diye MVP dışında tutuyor. | Önceliklendirme belirsizliği. | Faruk bu analizi onaylarsa `mvp-kapsami.md` güncellenmeli. |

---

## 11. Terminoloji Önerisi (sözlüğe girmeden kodlanmaz)

| MEB/okul dili (TR) | Önerilen identifier | Not |
|---|---|---|
| Yemekhane (modül) | `cafeteria` | **Mevcut** katalog anahtarı; değiştirilmez. |
| Yemek Menüsü | `mealMenu` | "Menü" tek başına kullanılmaz — gezinme menüsüyle karışır. |
| Öğün | `mealPeriod` | `meal` tek başına belirsiz (yemek mi öğün mü). Varsayılanlar: Kahvaltı / Öğle Yemeği / İkindi Kahvaltısı. |
| Yemek (menü kalemi) | `dish` | Katalog öğesi. |
| Yemek Kataloğu | `dishCatalog` | |
| Menü Grubu | `mealMenuGroup` | Kademeye/seviyeye eşlenen menü kitlesi; duyurudaki `audience` ile karıştırılmamalı. |
| Alerjen | `allergen` | 14'lük sabit liste. |
| Yemekhane Yoklaması (Faz 2) | `mealAttendance` | `rollCall`/`attendance` **değil** — devamsızlık hesabına girmez (`activityRollCall` gibi ayrı kavram). |

---

## 12. Kaynaklar

**K12NET**
- K12NET — "Delicious News from K12NET" (8 Mart 2023): https://www.k12net.com/en/?p=455
- K12NET — Haberler (sayfa 2; kafeterya modülü, 2023): https://k12net.com/en/news/page/2
- K12NET — Modüller (Modül 37 Kantin, Modül 39 Yemekhane): https://k12net.com/okul-yonetim-yazilimi/k12net-moduller/
- K12NET — Öğrenci-Veli Portalı kullanım belgesi (yemek geçmiyor): https://dokuman.k12net.com/dokuman/okul/k12netveliportali.doc
- K12NET SSS: https://sss.k12net.com/sss.html
- App Store — K12NET Mobil: https://apps.apple.com/tr/app/k12net-mobil/id1155767502?l=tr

**Okul siteleri**
- Bilişim Koleji: https://bilisim.k12.tr/yemek-listesi/
- TED Kayseri: https://www.tedkayseri.k12.tr/veli-alani/ · https://www.tedkayseri.k12.tr/aylik-yemek-listesi/
- ENKA İstanbul: https://www.enka.k12.tr/istanbul/tr/k12net-com-2/
- Akdeniz Boğaziçi Koleji: https://akdenizbogazicikoleji.k12.tr/yemek-menusu/
- Doğa Koleji Eskişehir: https://www.dogakoleji.k12.tr/eskisehir-kampusu/yemek-listesi-1384-51-58
- Arel (anaokulu PDF): https://www.arel.k12.tr/files/files/yemeklistesi/yemeklistesi_anaokulu.pdf

**Diğer sistemler**
- Minik İz: https://minikiz.com.tr/
- Ekid+: https://www.ekidplus.com/okul-oncesi-sistemi-yemek-listesi.html
- Kuki: https://kuki.vilicore.com/ (açılamadı; arama özeti)
- Power Kids: https://powerkidsapp.com/ · KreşCloud: https://krescloud.com/blog/anaokulu-kres-uygulamasi · Anaokula: https://anaokula.com/
- e-Okul VBS (haber): https://www.cnnturk.com/turkiye/e-okul-vbs-giris-ile-devamsizlik-bilgisi-ve-not-sorgulama-yapilabiliyor-904197

**Mevzuat (not düzeyi)**
- Alerjen zorunluluğu (haber, 2020): https://www.medikalakademi.com.tr/alerjiye-neden-olabilen-14-besinin-menuelere-yazilmasi-zorunlulugu-basladi/
- Gıda Etiketleme ve Tüketicileri Bilgilendirme Yönetmeliği (Lexpera): https://www.lexpera.com.tr/mevzuat/yonetmelikler/turk-gida-kodeksi-gida-etiketleme-ve-tuketicileri-bilgilendirme-yonetmeligi
- MEB — özel okul yemek/kıyafet/kırtasiye ödemeleri (27 Mart 2025): https://www.meb.gov.tr/mebden-ozel-okullarin-yemek-kiyafet-kirtasiye-odemelerine-yakin-takip/haber/36752/tr
- MEB — kantin ürün kriterleri (7 Nisan 2016): https://www.meb.gov.tr/okul-kantinlerinde-satisi-yapilacak-urunler-icin-yeni-kriterler/haber/10814/tr
- Sağlık Bakanlığı — Okullarda Yiyecek ve İçecek Standartları: https://hsgm.saglik.gov.tr/depo/birimler/saglikli-beslenme-ve-hareketli-hayat-db/Dokumanlar/Beslenme/Okullarda_Yiyecek_ve_ecek_Standartlar.pdf
- 6698 sayılı KVKK: https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf
