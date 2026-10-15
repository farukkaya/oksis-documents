# OKSİS — Veli Toplantısı Modülü · Claude Design Prompt · **00 · ORTAK BAĞLAM BLOĞU**

> Kaynak: `OKSİS — Veli Toplantısı Modülü · İhtiyaç Analizi (Final).md` (REQ-043, 15 Ekim 2026) §2–§13; marka: `docs/frontend/tasarim-sistemi/marka-tokenlari.md`; mobil: `docs/frontend/tasarim-sistemi/mobil-tasarim-haritasi.md`; bileşenler: `docs/frontend/bilesenler/_envanter.md`; sözlük: `oksis-ui/CLAUDE.md` "MEB Terminology Glossary".
>
> **Nasıl kullanılır:** Claude Design'da her ekran için yeni oturum aç. Önce aşağıdaki bloğu (tek parça) yapıştır, ardından ilgili ekran dosyasının (`PM-*.md`) prompt bloğunu yapıştır. Ekran dosyaları bu bloktaki tanımlara (rozet adları, veri evreni, kurallar) atıf yapar; blok yapıştırılmadan ekran prompt'u eksik kalır.
>
> **Wire notu:** `packages/api/src/generated/schema.ts` içinde henüz `parentMeeting*` tipi YOK (modül için kod yok, 2026-10-15). Ekran dosyalarındaki alan adları analiz §15 sözlük önerisinden türetilmiş **önerilerdir**; teknik analiz ve backend sözleşmesi çıkınca kod kazanır, prompt güncellenir. Tasarım, analizde olmayan veri alanı uydurmaz.
>
> **Ekran dosyaları:** PM-A-01 liste · PM-A-02 sihirbaz · PM-A-03 organizasyon detayı · PM-A-04 şube detayı · PM-A-05 tarih/iptal diyaloğu · PM-A-06 rapor · PM-A-07 ayarlar · PM-A-08 not denetim kaydı · PM-T-01 Toplantılarım (öğretmen) · PM-T-02 katılım takibi · PM-T-03 görüşme notu paneli · PM-T-04 öğretmen anasayfa kartı · PM-P-01 Toplantılarım (veli) · PM-P-02 ayrıntı + yanıt · PM-P-03 veli anasayfa kartı · PM-X-01 bildirim tercihleri (e-posta) · PM-E-01 e-posta şablonu.

```
## ÜRÜN BAĞLAMI

OKSİS: Türkiye'de K-12 özel/devlet okulları için çok kiracılı (multi-tenant) okul yönetim platformu. Tasarlanacak modül: **Veli Toplantısı**. Ürün dili Türkçe, hitap "siz". Kullanıcılar Türk okul idarecisi, sınıf rehber öğretmeni ve velidir; teknoloji meraklısı değildirler, telefondan hızlı iş görürler.

Modülün özü: İdare (okul yöneticisi + müdür yardımcısı) tek bir **organizasyon** altında okul geneli ya da seçili şubeler için toplantı planlar; ama her **şubenin toplantısı ayrı yönetilir** (farklı gün/saat/yer, tek şube iptali, tek şube tarih değişikliği). Veli **Katılacağım / Katılamayacağım** yanıtını (beyan) kendisi verir, dilediği zaman değiştirir; idare telefonla ulaştığı velinin adına yanıt girebilir ve geçmiş kaybolmaz. Toplantı günü şubenin sınıf rehber öğretmeni, beyandan bağımsız olarak **fiili katılımı** ("Geldi") işaretler ve **öğrenci başına görüşme notu** tutar. Not VELİYE ve ÖĞRENCİYE ASLA gösterilmez. İdare raporunda beyan ile fiili katılım iki ayrı eksendir.

## PLATFORM

- **WEB:** Next.js yönetim paneli, masaüstü tuvali 1440 × 900 (1280'e kadar düzgün daralır). Sol kenar çubuğu + üst bar + sayfa kabuğu; sezon/dönem seçici üst barda. Ekran başlığı için mevcut `PageHeader` (başlık, alt başlık, sağda birincil eylem, isteğe bağlı geri oku). Mevcut kalıpları kullan: `Dialog`/`ConfirmDialog` (tone info/warn/danger), `EmptyState` (filtered ayrımı), `Note` (info/warn/success/danger şeridi), `Toast`, `Pager`, `SelectCheckbox`, `MultiSelect`, `KpiCard/KpiRow`, `att-badge` tipi rozetler, sağdan çekmece (drawer). Yeni bileşen icat ETME; gerekiyorsa mevcut bileşenin varyantı olarak çiz ve "yeni bileşen adayı" diye ayrıca işaretle.
- **MOBİL:** React Native / Expo, tuval 402 × 874, iOS tarzı modern UI. YALNIZ LIGHT tema (koyu tema TASARLAMA). Alt sekme çubuğu (Material 3 indicator stili, 5 sekme): yönetici *Anasayfa · Duyurular · Okul · Raporlar · Daha fazla*; öğretmen *Anasayfa · Programım · Notlar · Ödev · Daha fazla*; veli *Anasayfa · Devamsızlık · Notlar · Ödevler · Daha fazla*. Veli Toplantısı kendi sekmesi OLMAZ; "Daha fazla" menüsünden ve anasayfa kartından/bildirimden girilir (yeri Faruk'la netleşecek — ekranda sekme çubuğunda "Daha fazla" seçili göster).
- **ÖZELLİK EŞİTLİĞİ:** Her ekran web ve mobilde aynı işlevi sunar (karar K-16). Farklı olan yalnız yerleşimdir. Web tablosu mobilde kart listesine, sağ çekmece mobilde alt panele (Sheet) döner.

## MARKA TOKEN'LARI (marka-tokenlari.md — başka hex YOK)

- Çekirdek: Lacivert `#1B2B5E` · Mavi `#3A4F9C` · Vurgu `#4F6BFF` · Yüzey `#EEF1FA` · Mürekkep `#0D1117`. Mobil türev nötrler: inkSoft `#4A5375`, inkSofter `#8A92AE`, border `#DDE3F1`, boş-durum ikon zemini `#E9EEF7`, iskelet `#E4E9F4`. İmza gradyanı (135°) yalnız logo/hero/uygulama ikonunda; bu modülün ekranlarında KULLANILMAZ.
- Portal renkleri (yalnız header/seçili sekme/birincil düğme vurgusu): İdare lacivert `#1B2B5E` · Öğretmen `#0E7A5A` · Veli `#B05A0A`. Bu renkler DURUM rengi olarak kullanılmaz.
- Durum/semantik (tek geçerli set): Başarı `#16A34A` (yazı `#15803D`, zemin `#E7F6EC`) · Uyarı `#D97706` (yazı `#A14808`, zemin `#FCEFDD`) · Hata `#DC2626` (yazı `#B91C1C`, zemin `#FBE7E7`) · Bilgi `#4F6BFF` (yazı `#3A4F9C`, zemin `#E9EDFF`) · Taslak (kullanıcı onaylı sapma) yazı `#6D4BD8`, zemin `#ECEAFB`.
- Tipografi: yalnız **Plus Jakarta Sans**. `JetBrains Mono` yalnız saat/sayı/kimlik sütunlarında (web). Web tip ölçeği kapalı küme: display 56/64 · h2 38/44 · h3 24/28 · body 17/26 · label 12/18 uppercase +8%. Mobilde kapalı ölçek yok: kullandığın her yazı boyutunu ekranın yanında listele (mevcut kullanım: 21 ekran başlığı · 18 alt ekran başlığı · 17 bölüm başlığı · 15 satır başlığı · 14 gövde · 13 etiket · 12 açıklama · 11 mikro).
- Radius: web `--radius: 1rem`; yüzeyler 16px (küçük) / 20px (büyük); rozet/çip tam yuvarlak. Mobil: kart 14–20, buton 12, çip 9999. Gölge çok hafif (`0 1px 2px rgba(13,17,23,0.04)`). Glow, glassmorphism, aşırı gradyan YOK.
- İkon: mevcut çizgi ikon seti (OksisIcon / Lucide benzeri, 1.75 çizgi). Emoji yok.
- Spacing: 4px ızgara. Dokunma hedefi minimum 44 × 44.

## MOBİL YAPISAL STANDARTLAR (birebir uy)

- **Sekme kökü ekranda** üstte `OksisHomeHeader` (portal renkli, 2. satır: bağlam), içerikte İKİNCİ büyük sayfa başlığı KONMAZ. **Alt (stack) ekranda** ince `STopBar`: geri oku + başlık (18/700). Veli toplantısı ekranlarının hepsi "Daha fazla"dan açılan ALT ekrandır (STopBar) — aksi belirtilmedikçe.
- **Zemin üzerine doğrudan metin YASAK:** sayfa zemini (`#EEF1FA`/açık nötr) üstüne hiçbir metin basılmaz; bölüm başlıkları, bilgi satırları, boş durum metinleri dahil her metin bir kart (`#FFFFFF`) ya da yumuşak yüzey içinde yaşar.
- **Oluşturma/düzenleme akışları** üretimde alt panel (`Sheet`) olur: içerik kadar panel ya da `Sheet size="full"`; çok adımlı akışta adım değişince panel kapanıp sonraki adım açılır, adım çizgisi yok, alt başlık adımın adıdır. Onay diyalogları ortalanmış `ConfirmSheet` kartıdır.
- Yatay sayfa kaydırması yok (yalnız çip şeritleri kayar). Safe area gözetilir. Sticky alt eylem çubuğu klavye açıkken ekranı yutmaz.
- Boş ekran kalıbı: 56×56 daire (`#E9EEF7`) içinde 24px nötr ikon · başlık 17/700 · açıklama 14, maks. 260px, ortalanmış — kart içinde.

## DOMAİN SÖZLÜĞÜ (UI'da birebir bu Türkçe terimler)

- **Veli Toplantısı** = organizasyon (`parentMeeting`): başlık, açıklama, varsayılan tarih/saat/yer, kapsadığı şubeler. Kendi durumu yalnız Taslak / Yayınlandı.
- **Şube Toplantısı** (`parentMeetingSection`): organizasyonun bir şubeye inmiş kaydı; kendi tarihi, başlangıç saati, isteğe bağlı bitiş saati, yeri, durumu, sorumlusu. İptal, tarih değişikliği, katılım ve notlar buna bağlıdır.
- **Sınıf Rehber Öğretmeni** (`homeroomTeacher`): şubenin sorumlusu. Branş öğretmeni modülde YOKTUR. Rehber öğretmeni olmayan şube planlanabilir ("Rehber öğretmen yok — toplantıyı idare yürütür").
- **Davetli Veli** (`meetingInvitee`) · **Katılım Yanıtı** (`meetingResponse`: Katılacağım / Katılamayacağım / Yanıt yok) · **Katılamama Nedeni** · **Yanıt Geçmişi** · **Veli Adına Yanıt** · **Plan Sürümü** (tarih/saat değişince artar; arayüzde "Sürüm 2" olarak kısaca görünebilir) · **Fiili Katılım** (`meetingPresence`: Geldi / işaretlenmedi → Tamamlandı'dan sonra "Gelmedi") · **Görüşme Notu** · **Not Denetim Kaydı** · **Düzenleme Penceresi** (Tamamlandı'dan sonra 7 gün) · **Hesapsız Veli** (OKSİS hesabı olmayan veli).
- ASLA kullanılmayacak sözcükler: "yoklama", "devamsızlık", "randevu", "assignment". Fiili katılım "yoklama" DEĞİLDİR. "Rehber" tek başına kişi rehberi modülünü çağrıştırır; her zaman "Sınıf Rehber Öğretmeni" yaz.
- Tarih biçimi: "Salı, 20 Ekim 2026" · saat 24 saatlik "18:30" · aralık "18:30–19:30". Sayılar Türkçe ayraçlı (1.240).

## YAŞAM DÖNGÜSÜ VE DURUM ROZETLERİ (tek tanım — her ekranda aynı)

Şube toplantısı durumu (rozet = ikon + metin; yalnız renk asla):

| Durum | Rozet görünümü | Anlam |
|---|---|---|
| **Taslak** | mor tint (`#ECEAFB` / `#6D4BD8`), kalem ikonu | Planlanıyor; yalnız idare görür, kimseye bildirim gitmedi |
| **Yayınlandı** | bilgi tint (`#E9EDFF` / `#3A4F9C`), takvim ikonu | Duyuruldu; yanıt toplanıyor |
| **Başladı** | DOLU lacivert (`#1B2B5E`, beyaz yazı) + küçük "canlı" nokta, kısa pulse (reduced-motion'da statik) | Başlangıç anı geldi (otomatik); fiili katılım ve not açık |
| **Tamamlandı** | başarı tint (`#E7F6EC` / `#15803D`), onay-daire ikonu | Öğretmen/idare kapattı ya da sistem 3 gün sonra otomatik kapattı |
| **İptal** | hata tint (`#FBE7E7` / `#B91C1C`), iptal-daire ikonu; yanında "İptal" bandı | Yayından sonra iptal edildi; kayıtlar korunur |

Organizasyon durumu yalnız **Taslak / Yayınlandı**; "12 şube: 11 yayında · 1 iptal" gibi özet şube durumlarından türetilir (küçük sayaç çipleri).

Veli beyanı rozeti (`meetingResponse`):

| Değer | Görünüm |
|---|---|
| **Katılacağım** | başarı tint, tik ikonu |
| **Katılamayacağım** | NÖTR (inkSoft yazı, `#E9EEF7` zemin), çarpı ikonu — bu bir hata değil, velinin kararıdır; hata/uyarı renginde olmaz |
| **Yanıt yok** | uyarı tint (`#FCEFDD` / `#A14808`), saat ikonu. VELİ yüzeyinde metin "Yanıt bekleniyor" (suçlayıcı değil), idare/öğretmen yüzeyinde "Yanıt yok" |
| **Veli adına** işareti | küçük ek çip "İdare girdi" (telefon ikonu) — beyan rozetinin yanında, ayrı renk değil |

Fiili katılım rozeti (`meetingPresence`) — beyan rozetinden GÖRSEL OLARAK AYRI bir dilde (yuvarlak kare "kutucuk" biçimi, beyan rozeti hap biçimi; böylece iki eksen karışmaz):
- **Geldi:** dolu başarı (`#16A34A` zemin, beyaz tik).
- **İşaretlenmedi** (toplantı Tamamlandı olana dek): nötr kesik çizgili kutu, "—". "Gelmedi" DENMEZ.
- **Gelmedi** (yalnız Tamamlandı sonrası): nötr koyu çarpı. Kırmızı değil.

**"Veliye gösterilmez" etiketi** (kalıcı, kapatılamaz, görüşme notu olan HER yüzeyde): göz-çizgili ikon + metin **"Veliye gösterilmez — yalnız sınıf rehber öğretmeni ve idare görür"**. Görünüm: lacivert %7 zemin (`rgba(27,43,94,0.07)`), lacivert yazı 12/600, 1px lacivert %22 kenarlık, tam genişlik şerit; sabit, kaydırmayla kaybolmaz (not paneli başlığına yapışık). Bunun dışında bu etiketin başka anlamı yoktur; durum rozeti gibi çizilmez.

Çakışma / değişiklik / iptal bantları (veli kartlarında): çakışma = uyarı tint "Aynı saatte başka toplantınız var"; değişiklik = bilgi tint "Saat değişti — lütfen yeniden yanıt verin"; iptal = hata tint "Bu toplantı iptal edildi".

## DEĞİŞMEZ ÜRÜN KURALLARI (ihlal edilirse tasarım reddedilir)

1. **Not metni, katılamama nedeni ve diğer velinin yanıtı** yalnız yetkili yüzeylerde görünür. Veli YALNIZ kendi yanıtını ve kendi nedenini görür; aynı öğrencinin diğer velisinin yanıtı/nedeni/varlığı veliye görünmez. Katılamama nedeni öğrenciye hiçbir yüzeyde görünmez.
2. **Öğrenci bu modülün kullanıcısı DEĞİLDİR** — öğrenci için ekran, bildirim, kart tasarlama.
3. **Beyan ≠ fiili:** fiili katılım beyandan ÖN DOLDURULMAZ. İki sayaç, iki rozet dili.
4. **Yalnız idare** planlar, yayınlar, tarih değiştirir, iptal eder, veli adına yanıt girer. Rehber öğretmen kendi şubesinde görür/işaretler/not yazar/tamamlar; veli adına yanıt GİREMEZ, planlayamaz. Başka öğretmen atanmaz (vekil/devir kavramı yok); öğretmen gelemezse idare iptal eder ya da toplantıyı kendisi yürütür.
5. **Başladı sonrası veli yanıtı kilitlidir** (salt okunur). İdare veli adına güncelleyebilir.
6. **Tamamlandı ve İptal terminaldir.** Tamamlandı'dan sonra 7 gün düzenleme penceresi, sonra salt okunur.
7. **E-posta yalnız bilgilendirmedir:** düğme, bağlantı, "katıl" çağrısı YOK. Veli yanıtını yalnız OKSİS uygulamasında/panelinde verir.
8. **V1'de SMS gönderilmez.** Hesapsız veliye hiçbir kanal ulaşmaz; çözümü idarenin "telefonla ara + veli adına işaretle" akışıdır. SMS'i çalışıyormuş gibi gösterme.
9. **Analizde olmayan işlev, alan, rapor, filtre ya da düğme ÇİZME.** Tasarımda görünen her şey için bir kaynak (alan/uç) olmalıdır; "backend'e eklenecek", "yakında" bantları üretim tasarımına girmez. Yapay zekâ özeti/önerisi, e-Okul aktarımı, görüntülü toplantı, birebir randevu, öğretmen kıyas/sıralaması YOK.
10. **Her ekran durum matrisi tanımlar:** yükleniyor (iskelet) · boş · hata (Tekrar dene) · izin yok (403) · bulunamadı/kapsam dışı (404) · sezon kapalı (salt okunur) · çevrimdışı/bağlantı koptu. Boş ekran yasak.
11. **Durum yalnız renkle taşınmaz** (ikon + metin eşlik eder). WCAG 2.1 AA, metin kontrastı ≥ 4.5:1, klavyeyle gezilebilirlik (web), ekran okuyucu etiketi (mobil), reduced-motion saygısı. Geçişler 150–250 ms ease-out, fonksiyonel.
12. **Suçlayıcı dil yok:** "Yanıt vermediniz!" değil "Yanıt bekleniyor". Veli kartında çocuk adı ve şube HER ZAMAN yazar.

## ÖRNEK VERİ EVRENİ (KURGU — gerçek kişi/kurum değil; birebir bu isimler)

- Okul: **Altınay Lisesi** (Ortaokul 5–8 + Lise 9–12). Sezon: **2026–2027 · 1. Dönem**. "Bugün": **Perşembe, 15 Ekim 2026** (toplantı günü ekranlarında: **Salı, 20 Ekim 2026**).
- İdare: **Neslihan Çetin** (Müdür Yardımcısı, tasarımdaki idare kullanıcısı) · Okul Müdürü **Rıza Tuncer**.
- Organizasyon: **"1. Dönem Veli Toplantısı"** — açıklama: "Dönem başı bilgilendirme ve öğrenci gelişim görüşmeleri. Lütfen kimlik kartınızı yanınızda getiriniz." — **12 şube**, varsayılan Salı 20 Ekim 2026; Ortaokul şubeleri 17:00–18:00, Lise şubeleri 18:30–19:30; yer = şubenin kendi sınıfı ("9-A Sınıfı"), 12-A için "Konferans Salonu". Önceki toplantı (geçmiş): **"Sezon Başı Tanışma Toplantısı"** — Salı, 22 Eylül 2026 · 4 şube (9-A, 9-B, 10-A, 12-A) · hepsi Tamamlandı · yanıt oranı %81.
- Şubeler ve sınıf rehber öğretmenleri: 5-A Aylin Demirtaş · 5-B Tolga Kılıç · 6-A Meral Özkan · 6-B Hasan Yıldırım · 7-A Pınar Sevim · 8-A Okan Bulut · **9-A Selma Arslan** · 9-B Burak Tekin · 10-A Gülay Erdem · **10-C Cemil Aksoy** (bu şube İPTAL durumunda; gerekçe idareye özel: "Rehber öğretmen raporlu") · **11-A — rehber öğretmen YOK** (uyarı rozeti) · 12-A Serpil Koç.
- Davetli veli sayıları (iptal edilen 10-C hariç, toplam 400; 10-C ayrıca 32): 5-A 34 · 5-B 35 · 6-A 36 · 6-B 36 · 7-A 37 · 8-A 36 · 9-A 32 · 9-B 38 · 10-A 40 · 11-A 36 · 12-A 40. Toplam beyan: Katılacağım 214 · Katılamayacağım 34 · Yanıt yok 152 (yanıt oranı %62). Hesapsız veli toplam 37.
- Ana ekran şubesi **9-A** (28 öğrenci, **32 davetli veli**): Katılacağım **18** · Katılamayacağım **4** · Yanıt yok **10** (bunlardan **3'ü hesapsız veli**; hesaplı 7) · fiili Geldi **15**.
- 9-A öğrencileri (örnek): Ahmet Kerem Polat, Bilge Naz Aydın, Can Efe Sönmez, **Defne Yalçın**, Ece Su Korkmaz, Furkan Aras, Gökçe Tan, Hira Nur Bozkurt, İlayda Kaplan, Kerem Ali Öztürk, Lale Sevgi Çınar, Mustafa Eren Doğru, Nehir Şahin, Ozan Taşkın.
- Veli örnekleri: **Nermin Yalçın** (Defne'nin annesi, Katılacağım) · **Kadir Yalçın** (Defne'nin babası, ayrı yaşıyor; bilgi görme yetkisi var; Yanıt yok) — aynı veli çiftinin **ikinci çocuğu Emir Yalçın (6-B)**: çoklu çocuk + çakışma örneği için. Diğer: Sevgi Polat (Ahmet Kerem'in annesi, Katılamayacağım — neden: "Vardiyam çakışıyor, öğretmenle telefonla görüşebilir miyiz?"), Hacer Aydın, Murat Sönmez (Yanıt yok), Dilek Korkmaz, Ramazan Aras, Ayhan Tan (hesapsız veliler), Sevinç Çınar (Lale Sevgi'nin annesi; idare telefonla yanıt işledi).
- Telefon (kurgu, maskeli gösterim): "0555 010 20 30" biçimi; gerçek numara kullanma. E-posta alan adı: `ornek.example`.
- Örnek görüşme notu (zararsız, özel nitelikli veri İÇERMEYEN): "Matematikte ek çalışma önerildi; haftada 2 saat etüt planlandı. Veli sınıf WhatsApp grubundan çıkmak istiyor." — notlara sağlık/aile durumu/din gibi özel nitelikli veri örneği YAZMA.
- Plan sürümü örneği: 6-B toplantısı 17:00'dan 18:00'e taşındı → Sürüm 2; 29 yanıt sıfırlandı.

## YASAKLAR

Aşırı gradyan, glow, glassmorphism, emoji, stok fotoğraf, koyu tema, "Buraya tıklayın", süs animasyonu, canlı geri sayım, öğretmen bazlı kıyas/sıralama, sağlık/özel nitelikli veri içeren örnek metin, gerçek kişi adı, yeni font/renk/ikon ailesi, geliştirici notu bantları.

## ÇIKTI BEKLENTİSİ (her ekran için)

1. Web 1440 ve mobil 402×874 çerçeveleri: dolu durum + ilgili rol/durum varyantları.
2. Durum matrisi çerçeveleri: yükleniyor · boş · hata · izin yok · (ekrana özel kenar durumlar).
3. Bileşen sayfası: ekrana özel rozet/kart anatomisi (kullanılan token adlarıyla).
4. Kullanılan mobil yazı boyutlarının ve yeni-bileşen-adaylarının listesi.
5. Prototipte rol ve durum değiştiren senaryo seçici (dev-only; üretime taşınmayacağı işaretli).
```

---

## Ortak sipariş sonrası kontrol listesi (her ekran dosyasının listesine EK olarak)

- [ ] Başka hex/font/radius sızmış mı? (yalnız marka-tokenlari.md seti)
- [ ] Beyan rozeti ile fiili katılım rozeti görsel olarak ayrışıyor mu? "Gelmedi" Tamamlandı öncesinde görünüyor mu? (görünmemeli)
- [ ] "Katılamayacağım" kırmızı/uyarı rengine kaymış mı? (nötr olmalı)
- [ ] Not, neden veya diğer velinin yanıtı veli/öğrenci yüzeyine sızmış mı?
- [ ] Öğrenci için ekran/bildirim/kart çizilmiş mi? (çizilmemeli)
- [ ] E-posta tasarımında düğme/bağlantı var mı? SMS "gönderildi" gibi gösterilmiş mi?
- [ ] Analizde olmayan alan/filtre/rapor uydurulmuş mu? Geliştirici notu bandı var mı?
- [ ] Mobilde zeminde yüzen metin, sekme kökünde ikinci büyük başlık var mı?
- [ ] 44pt dokunma hedefi, ikon+metin ile durum, AA kontrast sağlanmış mı?
- [ ] Yükleniyor/boş/hata/izin yok/404/sezon kapalı/çevrimdışı çerçeveleri var mı?
- [ ] Mevcut bileşenler (`PageHeader`, `Dialog`, `ConfirmDialog`, `EmptyState`, `Note`, `Toast`, `Pager`, `MultiSelect`, `KpiCard`; mobil `Sheet`, `Toggle`, `Note`, `Fab`, `EmptyState`) yeniden kullanıldı mı; yeni bileşen adayları ayrıca işaretlendi mi?
