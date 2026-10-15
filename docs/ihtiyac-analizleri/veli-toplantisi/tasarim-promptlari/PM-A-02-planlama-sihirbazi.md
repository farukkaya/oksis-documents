# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-02 · Planlama sihirbazı (oluştur / taslak düzenle / şube ekle)**

> Kaynak: İhtiyaç analizi §6.1, §10.2 PM-A-02, BR-PM-02/03/04/06/07/08/23/34/40, K-1, K-2, K-5. Platform: **Web + Mobil (eşit)**. Rol: İdare. Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN: Planlama sihirbazı (PM-A-02)

**Rol:** İdare. **Amaç:** Dönem başı toplantısını tüm okula (ya da seçili şubelere) 5 dakikanın altında kurup yayınlamak; ortaokulu bir saate, liseyi başka saate koyabilmek; yayından ÖNCE kimlere ulaşılacağını ve kimlere ulaşılamayacağını görmek.

## Üç adım (adım çubuğu: 1 Şubeler · 2 Tarih ve saat · 3 Önizleme)

### Adım 1 · Şubeler
Kademe → seviye → şube AĞACI (Ortaokul: 5, 6, 7, 8 · Lise: 9, 10, 11, 12; her şube onay kutulu satır). Üst kısayol çipleri: "Tüm okul" · "Tüm ortaokul" · "Tüm lise". Üst düğümler üç durumlu onay kutusu (tam/kısmi/boş). Her şube satırında: şube adı, rehber öğretmen adı, öğrenci sayısı ("28 öğrenci"). Rehber öğretmeni olmayan şube (11-A) **uyarı rozetiyle** görünür ("Rehber öğretmen yok") ama SEÇİLEBİLİR. Alt çubukta canlı özet: "12 şube · 318 öğrenci seçildi".
"Şube ekle" modunda organizasyonda zaten bulunan şube satırı devre dışıdır ("Zaten ekli" notu).

### Adım 2 · Tarih ve saat
Üstte form: **Başlık** (zorunlu), **Açıklama** (çok satırlı; velilere gider), **Varsayılan**: tarih · başlangıç saati · bitiş saati (isteğe bağlı) · yer.
Altında **şube tablosu** (web) / **şube kart listesi** (mobil): şube · rehber öğretmen · tarih · başlangıç · bitiş · yer. Başta tüm satırlar varsayılanı devralır (devralınan hücre soluk "varsayılan" göstergesiyle; elle değiştirilen hücre vurgulu). Satır içi düzenleme. Çoklu seçim + "Seçilenlere uygula" çubuğu (tarih/saat/yer alanlarından hangilerinin uygulanacağını seçtiren küçük panel): ortaokul şubelerini seç → 17:00–18:00; lise şubelerini seç → 18:30–19:30.
Satır doğrulama: **geçmiş an** (tarih + başlangıç saati birlikte "şimdi"den önce) satır hatası "Geçmiş bir an seçilemez"; bitiş başlangıçtan önceyse "Bitiş saati başlangıçtan sonra olmalı". Bugün seçilebilir, ama saati geçmişse hata verir. Hafta sonu, tatil, akşam saati için UYARI veya ENGEL yoktur. Rehbersiz şube satırında kalıcı mini uyarı "Toplantıyı idare yürütür".

### Adım 3 · Önizleme ve yayın
Özet kartları: şube sayısı · davetli öğrenci sayısı · davetli veli sayısı. **Kanal kırılımı tahmini** (yatay segment çubuğu + sayılar): "Uygulama içi + push ile ulaşacak 352 veli" · "Ayrıca e-posta tercihi açık 214 veli" · "Hesabı olmayan, hiçbir kanaldan ulaşılamayacak 37 veli" (uyarı tonu; yanında Note: "Bu velilere SMS gönderilemiyor. Yayından sonra 'Hesapsız veliler' listesinden telefonla arayabilirsiniz.").
**Çakışan veliler** bölümü: aynı veliye düşen, zaman aralıkları kesişen iki şube toplantısı (bitiş yoksa 60 dk varsayılır): "1 veliye aynı saatte iki toplantı düşüyor" + açılır liste (veli adı → çakışan iki şube/saat). Çakışma ENGELLEMEZ; uyarı tonu; "Planı değiştir" bağlantısı Adım 2'ye döner.
**Rehbersiz şubeler** bölümü: "11-A — rehber öğretmen yok; toplantıyı idare yürütür."
Altta düğmeler: "Taslak kaydet" (ikincil; bildirim gitmez) ve "Yayınla" (birincil). "Yayınla" → onay diyaloğu (web ConfirmDialog / mobil ConfirmSheet): "12 şube için yayınlanacak. 352 veliye bildirim gidecek (uygulama içi + push), 214'üne ayrıca e-posta. 11 sınıf rehber öğretmenine bilgi gidecek." Düğmeler: "Yayınla" / "Vazgeç".

## Modlar
1. **Yeni toplantı** (boş sihirbaz).
2. **Taslak düzenle:** Adım 1–2 dolu gelir; üstte "Taslak · son kaydedilen 14 Ekim 14:20" şeridi.
3. **Şube ekle** (yayınlanmış organizasyona sonradan şube; PM-A-03'ten açılır): Adım 1'de yalnız eklenmemiş şubeler seçilebilir; Adım 3 onay metni: "Eklenen şubeler doğrudan Yayınlandı olur; yalnız bu şubelerin velilerine duyuru gidecek (N şube, M veli)." Diğer şubeler etkilenmez. Başlık/açıklama salt okunur.

## WEB yerleşimi (1440)
Rotalar: /parent-meetings/new, /parent-meetings/[id]/edit, /parent-meetings/[id]/add-sections. PageHeader (geri oku + "Yeni toplantı"), altında yatay numaralı adım çubuğu (tamamlanan adımda tik). İçerik tek sütun, maks. 1080 px; Adım 2 şube tablosu tam genişlik (sütun başlıkları sabit, satırlar kaydırılır). Sağ altta sticky eylem çubuğu: Geri · İleri · (Adım 3'te) Taslak kaydet · Yayınla. Yarım kalan sihirbazdan çıkarken diyalog: "Kaydedilmemiş değişiklikler var. Taslak olarak kaydedilsin mi?"

## MOBİL yerleşimi (402×874)
Her adım **tam yükseklik Sheet** (size="full"); adım değişince panel kapanıp sonraki açılır, adım çizgisi YOK, panel alt başlığı adımın adıdır ("Şubeler" / "Tarih ve saat" / "Önizleme"). Altta sabit eylem çubuğu (İleri / Geri). Adım 2'de şube tablosu **kart listesine** döner: her şube bir kart (şube + rehber + tarih/saat/yer satırları; dokununca düzenleme alt paneli); toplu uygulama için "Seç" modu + alt çubukta "Seçilenlere uygula". Adım 3 özet kartları dikey, çakışma listesi akordeon. Yayın onayı ortalanmış ConfirmSheet.

## Örnek veri
- Adım 1: Tüm okul seçili, 12 şube, 318 öğrenci; 11-A rehbersiz uyarılı.
- Adım 2: Varsayılan Salı, 20 Ekim 2026 · 18:30–19:30. Ortaokul 6 şube 17:00–18:00 olarak toplu uygulanmış; yerler "<Şube> Sınıfı", 12-A "Konferans Salonu". Hata örneği: 8-A satırında "Çarşamba, 14 Ekim 2026 08:00" → "Geçmiş bir an seçilemez".
- Adım 3: 12 şube · 318 öğrenci · 389 davetli veli · 352 uygulama/push · 214 e-posta tercihli · 37 hesapsız. Çakışma örneği için 6-B satırı 18:00–19:00 yapılmış göster: "Nermin Yalçın: 6-B (Emir) 18:00–19:00 ile 9-A (Defne) 18:30–19:30 kesişiyor" (Kadir Yalçın için de aynı; veli sayısı 2 → "2 veliye aynı saatte iki toplantı düşüyor"; bu varyantta metni 2'ye uyarla).

## Durumlar
- Yükleniyor: Adım 1 ağaç iskeleti; Adım 3 "Davetli veliler hesaplanıyor…" iskeleti.
- Şube listesi boş (sezonda şube yok): EmptyState "Bu sezon için şube tanımlı değil" + "Okul yapısı ayarlarına git".
- Önizleme hatası: Adım 3 içinde hata + "Tekrar dene"; Yayınla devre dışı.
- Yayın/kayıt hatası: Toast hata; sihirbaz açık kalır, veri kaybolmaz.
- Yayın anında başlangıç anı geçmişe düşmüşse: Dialog tone=warn "Bazı şubelerin başlangıç anı geçti. Lütfen tarih ve saati güncelleyin." — ilgili satırlar işaretli, Adım 2'ye dönülür.
- İki idareci aynı anda: kaydederken Note "Kayıt değişti, yenileyin" + "Yenile".
- Sezon kapalı: sihirbaz açılmaz; EmptyState "Kapalı sezonda toplantı oluşturulamaz."
- İzin yok (yönetim izni yok): 403 durumu.
- Öğrencisi olmayan şube: Adım 1'de "Öğrenci yok" notu; seçilebilir ama önizlemede uyarı.
- Çevrimdışı: adımlar gezilebilir, Yayınla/Taslak kaydet devre dışı + "Bağlantı yok".

## Bu ekranda ÇİZME
Veli bazlı davetli seçimi/çıkarma (davetli küme sistemce çözülür) · sorumlu öğretmen seçimi (sorumlu şubenin güncel rehber öğretmenidir) · branş öğretmeni ataması · birebir randevu zaman dilimi · yinelenen toplantı · e-posta metni düzenleme · SMS kanalı ayarı · yapay zekâ başlık/metin önerisi.
```

## Sipariş sonrası kontrol listesi
- [ ] Rehbersiz şube SEÇİLEBİLİR mi (yalnız uyarılı)? Engellenmişse reddet.
- [ ] Çakışma ENGELLEMEZ, yalnız uyarı mı?
- [ ] Geçmiş an satır hatası var mı; hafta sonu/tatil için engel/uyarı uydurulmuş mu?
- [ ] Bitiş saati isteğe bağlı mı?
- [ ] Önizlemede hesapsız veli sayısı ve "SMS gitmeyecek" notu var mı? SMS "gidecek" izlenimi var mı?
- [ ] Yayın onay diyaloğunda şube/veli sayıları var mı?
- [ ] "Şube ekle" modunda yalnız yeni şubelerin velilerine duyuru gideceği yazıyor mu?
- [ ] Mobilde adımlar Sheet (full) mu, tablo kart listesine dönmüş mü, adım çizgisi yok mu?
- [ ] Sorumlu öğretmen seçimi alanı sızmış mı? (olmamalı)
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **Mobil sihirbaz kalıbı çelişkisi:** analiz §10.2 "adımlar tam ekran" der; `mobil-tasarim-haritasi.md` (2026-10-12) oluşturma akışlarını alt panel (Sheet) ister. Prompt Sheet size="full" kullanır. Teyit.
- "Şube ekle" ayrı ekran olarak envanterde yok (BR-PM-06); prompt sihirbazın kısıtlı modu olarak tanımlar.
- Kanal kırılımı sayıları önizleme ucundan gelecek; "e-posta tercihi açık" S-6 varsayılanına (N-01/N-03/N-04 için açık) bağlı.
- Taslağın otomatik kaydı belirtilmemiş; prompt yalnız elle "Taslak kaydet" + çıkışta sorma varsayar.
