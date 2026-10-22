---
tags: [frontend, bilesen, gereksinim, bulgu]
tarih: 2026-10-18
durum: pilot-onaylandi
ilgili: 2026-10-17-bulgu-0007-0028-tasarim-ve-yerlesim-kararlari.md
---

# Ortak katman bileşenleri · gereksinimler (BLG-0028)

Web panelinde ekranın üstüne açılan üç ortak bileşenin davranış sözleşmesi: **Modal**, **Yan panel**, **Sihirbaz sayfası**. Görsel kararlar ve sınıflandırma karar raporunda; bu belge "ne yapmalı" sorusunu kabul ölçütleriyle cevaplar ve pilotla birlikte kodun referansıdır.

| Bileşen | Görsel karar | Önizleme |
|---|---|---|
| Modal | B · Lacivert başlık | https://claude.ai/artifact/H2vVjemPTWmKRp86Q8iQEA |
| Yan panel | C · Kenar şeridi (öğrenci detayı 2. tur: A · düzgün hali) | https://claude.ai/artifact/1z5bXgDmNRuct7JE5qciVH |
| Sihirbaz sayfası | B · Özet sütunu (2. tur) | https://claude.ai/artifact/Rbwa2dYAwku2KFiFUMUszi |

## 0. Kapsam ve kısıtlar

- **K-1 · Yalnız frontend.** Hiçbir revizyon API istek/yanıt modeli, uç nokta veya üretilmiş şema değişikliği doğurmaz. Bir ekranın taşınması alan eklemeyi veya çıkarmayı gerektiriyorsa o ekran durdurulur ve kullanıcıya sorulur.
- **K-2 · Tek tema.** OKSİS web paneli açık temalıdır; renkler mevcut token'lardan (`--p`, `--a`, `--s`, `--line`, durum renkleri) gelir. Lacivert yüzey rolün ana rengidir (`--p`); öğretmen/veli/öğrenci portallarında kendi `--p` değeriyle çizilir.
- **K-3 · Kademeli geçiş.** Yeni bileşenler eklenir, eski kabuklar ekran ekran taşınır. Pilot onaylanmadan yaygınlaştırma yok.
- **K-4 · Kapsam dışı.** Mobil uygulama (ayrı bileşenleri var), navigasyon yan çubuğu, sayfa içi kalıcı bölümler (`LoadPanel`, `ExemptionPanel`, `ImportReviewPanel`, `ExamReviewPanel`).

## 1. Ortak katman altyapısı (üç bileşen için)

| No | Gereksinim | Kabul ölçütü |
|---|---|---|
| L-1 | Modal ve örtülü panel tek bir **katman yığını** paylaşır. | Açık katmanlar sırayla kaydedilir; yığın tek kaynaktır (tek context/store). |
| L-2 | **Esc yalnız en üstteki katmanı** kapatır. | Panel üstünde modal açıkken bir Esc yalnız modalı kapatır; ikinci Esc paneli. Hiçbir bileşen `window` üzerinde bağımsız Esc dinlemez. |
| L-3 | Katmanlar gövdeye **portal** ile çizilir. | Kaydırılan bir kapsayıcının içinden açılan katman kesilmez, `z-index` savaşı olmaz. |
| L-4 | Modal veya örtülü panel açıkken **arka sayfa kaymaz**; katmanın kendi kaydırması dışarı taşmaz. | Gövde kilidi açılır/kapanır; kayan gövdelerde `overscroll-behavior: contain`. |
| L-5 | **Odak yönetimi**: açılınca ilk anlamlı öğeye (onayda güvenli düğmeye) gider, Tab katman içinde döner, kapanınca açan öğeye döner. | Odak efekti yalnız açılışta bir kez çalışır; ebeveyn yeniden çizimi odağı çekmez. |
| L-6 | **İç içe sınırı**: modal üstünde en fazla bir modal (onay); panelden en fazla bir modal. | Üçüncü katman açma girişimi geliştirme ortamında konsol uyarısı verir. |
| L-7 | **Kapatma kilidi**: iş sürerken (kaydetme, yayınlama) Esc, ✕, örtü ve Vazgeç kapalıdır. | `busy` durumunda dört yol da etkisiz; ana düğme kendi içinde döner. |
| L-8 | **Kaydedilmemiş değişiklik koruması**: kirli formda kapatma girişimi önce onay sorar. | "Değişiklikler kaydedilmedi" onayı (modal, uyarı tonu); varsayılan odak "Forma dön". Temiz formda onay sorulmaz. |
| L-9 | **Erişilebilir ad**: her katman başlığa `aria-labelledby` ile bağlıdır. | Adsız katman geliştirme ortamında uyarı verir. |
| L-10 | **Hareket azaltma**: `prefers-reduced-motion` açıkken giriş/çıkış animasyonu yok. | |
| L-11 | **Imperatif onay**: `await confirm({ title, body, tone, confirmLabel })` true/false döner; yazarak onay için `typedValue` alır. | JSX'te ayrı durum tutmadan kullanılır; yığına normal katman olarak girer. |

## 2. Modal

### 2.1 Yapı

- **Başlık (lacivert yüzey):** üstte modül adı (kicker, ör. "Devamsızlık · Mazeret talebi"), ikon karosu, başlık, isteğe bağlı bağlam etiketleri (ör. öğrenci · şube · tarih), sağ üstte ✕. Tonlar: varsayılan lacivert, **tehlike** koyu kırmızı, **uyarı** koyu turuncu.
- **Gövde:** beyaz, tek kayan bölge.
- **Alt çubuk:** gövdenin üstüne gölgeyle yükselir; solda **not satırı** (kararın sonucu/etkisi: "Karar veliye bildirim olarak gider", "Geri alınamaz"), sağda düğmeler (ikincil sonra birincil).

### 2.2 Gereksinimler

| No | Gereksinim | Kabul ölçütü |
|---|---|---|
| M-1 | Boyutlar **sm 400 · md 560 · lg 760**. | Başka genişlik tanımlanamaz; daha büyüğü sayfa veya panel olur. |
| M-2 | Yükseklik ekranı aşmaz; başlık ve alt çubuk sabit, gövde kayar. | 320×720'de de alt çubuk görünür. |
| M-3 | **640px altında** aynı içerik alttan açılan tam genişlik panel olur; düğmeler tam genişlik. | Bileşen değişmez, yalnız yerleşim. |
| M-4 | Örtüye tıklama yalnız form **temizken** kapatır. | Kirli formda örtü tıklaması L-8 onayını açar. |
| M-5 | Alan hatası alanın altında; sunucu hatası gövdenin üstünde şerit. Girilen veri silinmez. Gönderimde odak ilk hatalı alana gider. | Hata yalnız toast ile verilmez. |
| M-6 | **Yazarak onay** yalnız geri alınamaz ve geniş etkili silmelerde (şube, sezon, toplu silme). Etki listesi gövdede; ad doğru yazılana kadar sil düğmesi kapalı. | |
| M-7 | **Kısmi sonuç durumu** (BLG-0026): toplu işlem sonrası aynı modal sonuç durumuna geçer; başlık uyarı tonuna döner, başarılı sayısı + başarısız satırlar nedenleriyle, yalnız başarısızları yeniden dene. | Başarısız satır yoksa başarı durumu. |
| M-8 | Salt okunur **bilgi** varyantı: eylem yok, tek "Kapat". | |
| M-9 | Rol: `dialog`; onaylarda `alertdialog`; `aria-modal="true"`. | |

## 3. Yan panel

### 3.1 Yapı

- **Kenar şeridi (64px, lacivert, sol kenar):** üstte kapat; altında önceki/sonraki okları ve sıra sayacı ("3/28"); sonra simge + kısa etiketli dikey sekmeler.
- **Başlık (beyaz, sade):** modül adı, avatar/ikon, ad, bağlam etiketleri (uyarı etiketi turuncu).
- **Gövde:** beyaz, tek kayan bölge, bölümlere ayrılmış.
- **Alt çubuk:** modalla aynı (not satırı + düğmeler).

### 3.2 Gereksinimler

| No | Gereksinim | Kabul ölçütü |
|---|---|---|
| P-1 | İki çalışma biçimi: **yanaşık** (varsayılan) ve **örtülü**. | Ekran çağırırken seçer; varsayılan yanaşık. |
| P-2 | **Yanaşık**: sayfa düzeninin parçası; liste daralır, kullanılır kalır; örtü, odak tuzağı ve `aria-modal` yok; rol `complementary`/`region`, erişilebilir adlı. | Panel açıkken listede başka satıra tıklamak paneli o kayda geçirir. |
| P-3 | **Örtülü**: modal davranışı (örtü, kilit, odak tuzağı, `aria-modal`, L-1…L-5). Geniş tablolu ekranlar (not defteri, yoklama girişi) bunu kullanır. | |
| P-4 | **Kırılma noktaları**: 1280px altında yanaşık kendiliğinden örtülüye; 640px altında tam ekran. Telefonda kenar şeridi üstte yatay çubuğa döner. | |
| P-5 | **Genişlik** 420 (dar detay), 560 (sekmeli detay) veya 600 (özet + sekmeli zengin detay; öğrenci detayı). | Başka genişlik yok. |
| P-6 | **Adres durumu**: açık kayıt `?detay=<id>` ile adreste; `replace` ile yazılır (gezinme geçmişi kirlenmez). Yenileyince ve bağlantıyla açıldığında panel açık gelir; panel kapanınca parametre silinir. | |
| P-7 | **Önceki/sonraki**: listenin geçerli sıralama ve filtresine göre; ilk/son kayıtta ilgili ok kapalı. Panel odaktayken ↑/↓ kısayolu. | Listede seçili satır vurgulu ve görünür alana kaydırılır. |
| P-8 | **Sekmeler** en fazla 4. Fazlası olan detay (Öğretmen, 8 sekme) detay sayfasına taşınır. | `tablist`/`tab`/`tabpanel` ve `aria-controls` eksiksiz. |
| P-9 | **Veri durumları** ayrı yuvalar: yükleniyor (iskelet), hata (açıklama + yeniden dene), boş (ne olmadığını ve nasıl oluşacağını söyler). Başlık bilinen bilgiyle hemen çizilir, yalnız gövde bekler. | **Hata hiçbir zaman boş durum gibi görünmez.** |
| P-10 | **Panel içinde düzenleme**: tek alanlık küçük düzeltme yerinde yapılabilir; birden fazla alanlı düzenleme ve her kritik karar modal açar. Panelde form, sihirbaz ve onaysız kritik eylem olmaz. | Ör. mazeret onay/ret, istisna geri alma, sürüm geri yükleme modal onayıyla. |
| P-11 | Panel başka bir panel açmaz; sayfaya giden eylem paneli kapatıp gider (gizli olarak bağlı tutulmaz). | |

## 4. Sihirbaz sayfası

### 4.1 Yapı

2. tur kararı (2026-10-18): ilk pilotta uygulanan "A · Yol haritası" kullanıcı tarafından reddedildi — uygulama menüsünün yanında ikinci lacivert sütun ve sayfa başlığının altında aynı ölçekte ikinci bir sihirbaz başlığı. Yerine **B · Özet sütunu**:

- **Kabuk değişmez.** Tam menü ve üst çubuk olduğu gibi kalır; lacivert yalnız menüdedir. Sihirbaz ikinci lacivert yüzey açmaz.
- **Tek başlık.** Sayfanın adı yalnız üst çubuktaki kırıntıdadır ("Öğrenciler › Yeni öğrenci kaydı"). Sihirbaz ayrı bir sayfa başlığı bandı açmaz; dönem/okul etiketleri tekrarlanmaz.
- **Form kartı (sol, en fazla 780px):** üst kenarda 5 parçalı ince ilerleme çizgisi; içinde "Adım n / N", adım adı (tek başlık, 19px), kısa açıklama ve form.
- **Kayıt özeti sütunu (sağ, 320px, beyaz kart, yapışkan):** başlık "Kayıt özeti" + "k / N tamam"; her adım numara/tik, ad ve verilen cevap; tamamlanan adım tıklanınca o adıma döner (üzerine gelince "Düzenle"); geçerli adım açık mavi zeminle vurgulu; altta güven notu ("Kayıt tamamlanmadan hiçbir veri oluşturulmaz").
- **Alt çubuk (sabit, yükselen):** Geri · not satırı · Vazgeç · Devam / son adımda eylem adı.
- **Biçim düzeltmeleri:** çalışmayan alanlar (öğrenci fotoğrafı) gösterilmez; adım adları cümle düzeninde; segment seçiciler zeminli ve 42px; tarih alanı diğer alanlarla aynı yazı.

### 4.2 Gereksinimler

| No | Gereksinim | Kabul ölçütü |
|---|---|---|
| W-1 | Bütün sihirbazlar bu sayfa bileşenini kullanır; modal veya çekmecede sihirbaz yoktur. | Kapsam: Öğrenci kaydı, Okul kaydı, Sınav oturumu, Davet hazırlama, Etkinlik oluşturma, Kulüp kaydı. |
| W-2 | **Adres**: `?adim=n`; tarayıcı geri tuşu önceki adıma döner; doğrudan ileri bir adıma bağlantıyla gelinirse ilk geçersiz adıma yönlendirilir. | |
| W-3 | **İleri ancak geçerliyse**: Devam önce geçerli adımı doğrular; eksikse alanlar işaretlenir, odak ilk hataya gider, not satırı eksik sayısını söyler. Geri her zaman serbest. | Şeritte ileri adımlara tıklanamaz; tamamlanan adımlara tıklanır. |
| W-4 | **Düzenleme kipi** (mevcut kaydı düzenleyen sihirbaz, ör. kulüp): tüm adımlar tamamlanmış sayılır, her adım açılabilir, Kaydet her adımda görünür. | |
| W-5 | **Özet adımı** (varsa): bölüm bazında "Düzenle" bağlantıları ilgili adıma döner. | |
| W-6 | **Son gönderim** tüm formu doğrular; hata varsa ilk hatalı adıma gidilir ve hangi adımda olduğu söylenir. | |
| W-7 | **Koruma**: herhangi bir alan doluyken Vazgeç, Kapat, kenar çubuğu gezinmesi ve tarayıcıdan ayrılma (`beforeunload`) önce onay sorar ("Kayıt yarıda kalacak"). | |
| W-8 | **Bitiş**: ortak başarı ekranı aynı sayfada (ne oldu, kimlik bilgileri, sonraki eylemler: ör. "Öğrenci detayını aç", "Yeni kayıt"). Sessizce kapanma veya başka sayfaya atma yok. | |
| W-9 | **Uzun işler** sırasında (gönderim) sayfa kilitlenir, alt çubuk durumu söyler. | |
| W-10 | **Dar ekran**: 1100px altında özet sütunu formun üstüne tek satırlık özet/ilerleme şeridine ("Adım 2/5 · Sırada: Sınıf", tıklanınca adım listesi açılır) iner; alt çubuk düğmeleri tam genişlik. | |

## 5. Süreç sayfaları ve detay sayfası

Sihirbaz olmayan ağır akışlar sayfa olur ama W-bileşenini zorla kullanmaz: Kayıt yenileme (tablo + aşamalar), Otomatik program üretimi, Nöbet otomatik dağıtımı (arka plan işi + geniş sonuç). Bunlar için ortak beklenti: adreste aşama, iş sürerken ayrılma uyarısı, sonuç ekranı aynı sayfada, hata durumunun açıkça gösterilmesi. Öğretmen detayı ayrı detay sayfası olur (sekmeler sayfa sekmesi). Ayrıntılı tasarım ayrı bulguda.

## 6. Taşıma ve pilot

1. Altyapı (L-1…L-11) ve üç bileşen, pilot ekranlarla birlikte gelir:
   - Modal: **mazeret karar modalı** (mazeret detayındaki onay/ret modal onayına taşınır).
   - Panel: **öğrenci detayı** (yanaşık + adres durumu + önceki/sonraki).
   - Sihirbaz: **öğrenci kaydı** sayfası.
2. Kullanıcı pilotu görüp onaylar.
3. Kalan yüzeyler sınıflandırmaya göre dalgalar halinde taşınır (karar raporundaki son sınıflandırma tablosu); her dalga sonunda eski kabuk sınıfları kaldırılır.

## 7. Bilinen bağımsız düzeltmeler

Bu bileşenleri beklemeden ayrı bir turda düzeltilir (2026-10-18'de ajana verildi): hatanın boş görünmesi (sürüm geçmişi, istisnalar, nöbet sürümleri, duyuru geçmişi), istisna geri almada onay ve gerekçe, kayıt sürerken kapatılabilen paneller, Esc katman hataları, mazeret reddinde gerekçe kaybı, toplu şube hatasının gösterilmemesi, sınav türü hatasının toast'ta kalması, okul sihirbazında adım doğrulaması, sezon açılış penceresi genişliği, odağın yeniden çizimde geri çekilmesi.

## 8. Pilot sonucu (2026-10-18)

Üç pilot ekran kullanıcı tarafından onaylandı ve tarayıcıda gözle doğrulandı:

- **Sihirbaz sayfası:** `/students/new` — B · Özet sütunu + ince ayarlar (tam genişlik, 1/2/3 sütunlu alan ızgarası, iki sütunlu özet adımı, tek panelli davet, olgular paneli).
- **Yan panel:** `/students` öğrenci detayı — **örtülü**, 600px, kenar şeridi; Genel sekmesi üç gösterge + bilgiler + veliler + son devamsızlık özeti. Yanaşık deneme reddedildi: geniş tablolu liste ekranlarında panel örtülü açılır.
- **Modal:** mazeret karar modalı — B · Lacivert başlık, tonu karara göre değişir (ret → tehlike), seçim ve gerekçe doğrulaması.

Öğrenilenler: prototip her zaman gerçek kabuk ve gerçek ekran üzerinde çizilir; ajan çıktısı kullanıcıya sunulmadan önce tarayıcıda gözle kontrol edilir; panel içeriği satırdaki bilgiyi tekrar etmez, özet + ayrıntı verir; boş alan bırakılmaz.
