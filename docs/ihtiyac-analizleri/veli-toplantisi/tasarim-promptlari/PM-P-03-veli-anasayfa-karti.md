# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-P-03 · Veli anasayfası: toplantı günü kartı**

> Kaynak: İhtiyaç analizi §3 (Anasayfa), §6.3 (6), §10.2 PM-P-03, K-15, BR-PM-19/22/23, N-05. Platform: **Web + Mobil (eşit)**. Rol: Veli (Bronz). Bu bir **kart bileşeni**dir (tam ekran değil); mevcut veli anasayfasına yerleştirilir. Önce `00-ortak-baglam.md`, sonra bu blok. Emsal: `apps/mobile/src/features/home/components/parent-home.tsx` ("Çocuklarım", "Yaklaşanlar", "Okuldan duyurular" bölümleri), `home-parts.tsx`, `due-parts.tsx`.

```
# BİLEŞEN: Veli anasayfası — toplantı günü kartı (PM-P-03)

**Rol:** Veli (Nermin Yalçın). **Amaç:** Toplantı günü sabahı anasayfayı açan veli, bugün hangi çocuğunun hangi saatte nerede toplantısı olduğunu görsün; henüz yanıt vermediyse kartın İÇİNDEN yanıtlasın.

Mevcut veli anasayfası iskeletine UYUM zorunludur: yeni bölüm başlığı/ekran başlığı icat etme; kart, mevcut kart kabuğunun (beyaz yüzey, hafif gölge) bir örneğidir. Anasayfa tam çerçevesini YENİDEN ÇİZME — yalnız mevcut anasayfa üzerinde kartın yerini ve kendi çerçevelerini göster.

## Ne zaman görünür
Şube toplantısı Yayınlandı/Başladı iken ve toplantı GÜNÜ boyunca. İptal edilmişse iptal bandıyla o gün görünür. Tamamlandı → kalkar. Çocuk başına AYRI kart (iki çocuğun aynı gün toplantısı varsa iki kart, en fazla 2 + "Tümü" bağlantısı → PM-P-01).

## Yeri
Mobil: veli anasayfasında "Çocuklarım" bölümünün hemen altında, "Yaklaşanlar" listesinin üstünde. Web: veli dashboard kart ızgarasında üst satır.

## Kart içeriği
- Üst satır: ikon (takvim-kullanıcılar) + etiket "Veli toplantısı" + durum rozeti (Yanıt bekleniyor / Katılacağım / Katılamayacağım; Başladı'da ek "Başladı" canlı rozeti).
- Çocuk: avatar + "**Defne Yalçın · 9-A**" (çocuk adı ve şube HER ZAMAN).
- Saat ve yer: "**Bugün 18:30**" (büyük) · "9-A Sınıfı".
- Yanıt alanı:
  - **Yanıtsızsa:** kartın içinde iki büyük düğme "Katılacağım" / "Katılamayacağım" (≥ 48 pt). "Katılamayacağım" → alt panel/satır içi açılım: isteğe bağlı neden (+ "Bu açıklamayı yalnız öğretmen ve okul idaresi görür") — PM-P-02 ile aynı davranış.
  - **Yanıtlıysa:** durum satırı + küçük "Yanıtı değiştir" bağlantısı (Başladı olana dek).
  - **Başladı:** yanıt kilitli; yalnız durum gösterilir.
- Bantlar (PM-P-01 ile aynı dil, tek satıra sıkıştırılmış): Çakışma ("Aynı saatte başka toplantınız var"), İptal ("Bu toplantı iptal edildi", düğmesiz, gerekçe yok), Değişiklik ("Saat değişti — yeniden yanıt verin").
- Kartın boş yerine dokunma → PM-P-02.
Veli diğer velinin yanıtını görmez; fiili katılım veya not göstermez.

## Varyantlar (ayrı çerçeveler)
(a) Yanıtsız, iki düğme; (b) Katılacağım yanıtlı; (c) Katılamayacağım yanıtlı (nötr); (d) Başladı — kilit; (e) İptal bandı; (f) İki çocuk, iki kart + çakışma bandı (Defne 18:30 ve Emir 18:00–19:00 aynı gün); (g) Tek çocuklu veli.

## Örnek veri
Salı, 20 Ekim 2026. Kart 1: Defne Yalçın · 9-A · Bugün 18:30 · 9-A Sınıfı · Yanıt bekleniyor. Kart 2: Emir Yalçın · 6-B · Bugün 18:00 · 6-B Sınıfı · Katılacağım. Çakışma bandı her ikisinde.

## Durumlar
- Yükleniyor: kart iskeleti.
- Boş: toplantı yoksa kart HİÇ görünmez (boş kart/"toplantı yok" metni ÇİZME).
- Hata: kart yerinde küçük "Toplantı bilgisi yüklenemedi · Tekrar dene"; anasayfanın diğer bölümleri etkilenmez. Yanıt kaydı hatası: kart eski durumda + Toast "Yanıtınız kaydedilemedi".
- Çocuksuz/bağlantısız veli: kart görünmez (mevcut çocuk kapısı kalıbı geçerli).
- Çevrimdışı: düğmeler devre dışı + "Bağlantı yok".
- Sezon kapalı: kart görünmez.

## Bu ekranda ÇİZME
Anasayfanın tamamı · toplantı günü dışında "yaklaşan toplantı" kartı (bkz. karar notu) · fiili katılım · diğer veli · geri sayım · harita/yol tarifi.
```

## Sipariş sonrası kontrol listesi
- [ ] Çocuk adı + şube kartta her zaman yazıyor mu?
- [ ] Yanıtsız kartta düğmeler kartın içinde ve ≥ 48pt mi; yanıtlıda "Yanıtı değiştir" var mı?
- [ ] Toplantı yokken kart görünmüyor mu?
- [ ] Mevcut anasayfa kart dili korunmuş, yeni bölüm başlığı yok mu?
- [ ] İptal bandında gerekçe yok mu; Başladı'da kilit var mı?
- [ ] Mobilde zeminde yüzen metin yok mu?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **Görünme penceresi:** "toplantı günü kartı" — yanıtsız veli için yayından toplantı gününe kadar "yaklaşan" bir anasayfa kartı istenip istenmediği belirsiz (yanıt oranı için değerli olabilir; mevcut "Yaklaşanlar" listesi bu işi görebilir mi?). Prompt yalnız toplantı günü varsayar.
- Anasayfa yerleşimi (Çocuklarım altı / Yaklaşanlar üstü) öneridir.
- İki çocuk, iki kart + "Tümü" sınırı (en fazla 2) öneri.
- Tamamlandı sonrası kartın aynı gün kalıp kalmayacağı belirsiz (prompt kaldırır).
