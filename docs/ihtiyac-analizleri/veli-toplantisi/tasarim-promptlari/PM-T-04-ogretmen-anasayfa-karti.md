# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-T-04 · Öğretmen anasayfası: toplantı günü kartı + Bekleyen İşler satırı**

> Kaynak: İhtiyaç analizi §3 (Anasayfa), §6.2 (2, 8), §10.2 PM-T-04, K-15, BR-PM-12, §9.1 ("Üretilmeyecek bildirimler"). Platform: **Web + Mobil (eşit)**. Rol: Sınıf Rehber Öğretmeni. Bu bir **bileşen/kart ailesi**dir (tam ekran değil); mevcut öğretmen anasayfasına yerleştirilir. Önce `00-ortak-baglam.md`, sonra bu blok. Emsal: `apps/mobile/src/features/home/components/teacher-home.tsx` (A kare "şu an" · B/C sayaç · D Bekleyen İşler · E kısayol iskeleti) ve `due-parts.tsx`, `home-parts.tsx`; web dashboard kartları.

```
# BİLEŞEN AİLESİ: Öğretmen anasayfası — veli toplantısı kartı ve Bekleyen İşler satırı (PM-T-04)

**Rol:** Sınıf Rehber Öğretmeni (Selma Arslan). **Amaç:** Toplantı günü, anasayfayı açar açmaz toplantının saatini/yerini ve kaç velinin geleceğini görmek ve tek dokunuşla katılım takibine geçmek; toplantıyı tamamlamayı unuttuysa Bekleyen İşler'de hatırlatma görmek.

Mevcut öğretmen anasayfa iskeletine UYUM zorunludur: anasayfada yeni bölüm başlığı/ekran başlığı icat etme; kart, mevcut kart kabuğunun (beyaz yüzey, 14–20 radius, hafif gölge) bir örneğidir. Anasayfa tam çerçevesini YENİDEN ÇİZME — yalnız mevcut anasayfa üzerinde kartın yerini ve kendi çerçevelerini göster.

## 1) TOPLANTI GÜNÜ KARTI
**Ne zaman görünür:** şube toplantısı Yayınlandı/Başladı iken ve toplantı GÜNÜ boyunca (toplantı günü 00:00'dan Tamamlandı olana dek). Tamamlandığında kart anasayfadan kalkar (toplantı günü tamamlanmışsa bir daha çıkmaz). İptal edilmişse iptal bandıyla o gün için görünür (gerekçe yok).
**Yeri:** mobilde A kare ("şu an") bölümünün hemen altında, sayaç satırlarının üstünde, tek kart. Web'de dashboard kart ızgarasında üst satır, "Bugünkü dersler" yanında.
**İçerik:**
- Üst satır: ikon (takvim-kullanıcılar) + etiket "Veli toplantısı" + durum rozeti (Yayınlandı: bilgi tint / Başladı: dolu lacivert + canlı nokta).
- Şube ve saat: "9-A · 18:30–19:30", yer: "9-A Sınıfı".
- Sayaç satırı: "**Katılacak 18 / 32 veli**" (ilerleme çubuğuyla; 18/32). Başladı iken ikinci sayaç: "**Geldi 15**" (ayrı kutucuk dili).
- Birincil düğme: "**Katılım ekranına git**" (PM-T-02). Yayınlandı'da (başlamadan önce) düğme aynı ama ikincil: "Yanıtları gör".
- Çok şubeli rehber öğretmen (nadir): kartlar dikey yığılır, en fazla 2 + "Tümü" bağlantısı (PM-T-01).
**Varyantlar (ayrı çerçeveler):** (a) Yayınlandı, toplantı günü sabahı; (b) Başladı, canlı; (c) İptal bandı: hata tint "Bugünkü 9-A veli toplantısı iptal edildi" (kart tek satıra iner, düğme "Ayrıntı"); (d) rehber öğretmeni olmayan şube — bu rol için kart yok.
**Web:** aynı içerik, kart genişliği 360–440 px, düğme kartın altında tam genişlik; fare/klavye odak halkası.

## 2) BEKLEYEN İŞLER SATIRI
Anasayfanın D listesi ("Bekleyen İşler": notu eksik sınav, kontrol bekleyen ödev, düzeltme talebi ile aynı tipte tek satır dili) içine yeni satır tipi.
- Ne zaman: şube toplantısı **Başladı** durumunda kalmışsa (başlangıç anı geçti, Tamamlandı değil) — yani öğretmen toplantıyı tamamlamayı unutmuş olabilir. Tamamlandığında satır kaybolur. Otomatik kapanış (3 gün) olursa satır da kalkar (bildirim üretilmez; satır tek hatırlatmadır).
- Görünüm: sol ikon (takvim-onay) · etiket çipi "Toplantı" · başlık "**Tamamlanmayı bekleyen toplantı · 9-A**" · ikinci satır "Salı, 20 Ekim · katılım ve notlar işlenmeyi bekliyor" · sağda ">" . Dokununca PM-T-02.
- Diğer bekleyen iş satırlarıyla ortak dil (yükseklik, tipografi); aynı listede en fazla 3 satır sınırı korunur (öncelik sırası mevcut anasayfa kuralıyla).
- Varyant: iki gün önce başlamış toplantı için ikinci satır "Salı, 20 Ekim · 2 gün önce başladı · 1 gün sonra otomatik kapanacak" (otomatik kapanış 3 gündür; bu bilgi satırı öneridir).

## Örnek veri
Kart: 9-A · Salı, 20 Ekim 2026 · 18:30–19:30 · 9-A Sınıfı · Katılacak 18 / 32 veli · (Başladı) Geldi 15. Bekleyen İş: "Tamamlanmayı bekleyen toplantı · 9-A".

## Durumlar
- Yükleniyor: kart iskeleti (anasayfanın mevcut iskelet diliyle).
- Boş: toplantı yoksa kart HİÇ görünmez (boş kart/"toplantı yok" metni ÇİZME). Bekleyen iş yoksa satır yok.
- Hata: kartın verisi yüklenemezse anasayfanın mevcut bölüm-hata diliyle küçük "Toplantı bilgisi yüklenemedi · Tekrar dene" satırı; anasayfanın diğer kartları etkilenmez.
- İzin yok: rehber öğretmeni olmayan öğretmen/branş öğretmeni — kart görünmez.
- Çevrimdışı: son bilinen sayılar + sönük "güncel olmayabilir" etiketi.
- Sezon kapalı: kart görünmez.

## Bu ekranda ÇİZME
Anasayfa tamamını yeniden tasarlama · toplantı planlama/iptal kısayolu · veli yanıtı bildirimi · öğretmen bazlı başarı rozeti/kıyas · geri sayım sayacı · "Yakında" kutusu.
```

## Sipariş sonrası kontrol listesi
- [ ] Mevcut anasayfa kartı dilinden sapma/yeni bölüm başlığı var mı?
- [ ] Toplantı yoksa kart görünmüyor mu (boş kart yok)?
- [ ] "Geldi" sayacı yalnız Başladı'da, beyan sayacından ayrı dilde mi?
- [ ] Bekleyen İşler satırı yalnız Başladı (tamamlanmamış) durumda mı; en fazla 3 satır korunuyor mu?
- [ ] İptal bandında gerekçe yok mu?
- [ ] Web ve mobil çerçeve var mı; dokunma hedefi 44pt mi?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **Kartın görünme penceresi:** envanter "toplantı günü kartı" der; yayından toplantı gününe kadar ön-hatırlatma kartı (ör. 2 gün önce) istenip istenmediği belirsiz. Prompt yalnız toplantı günü varsayar.
- Tamamlandı sonrası kartın aynı gün kalıp kalmayacağı belirsiz (prompt kaldırır).
- İptal edilen toplantı için öğretmen kartı (prompt iptal bandı gösterir) envanterde yok.
- Yerleşim: mobilde A kare altında / sayaç üstünde, web'de üst satır — öneri; mevcut home slotlarıyla uyum teyidi gerekir.
- "Otomatik kapanışa N gün kaldı" ikinci satırı envanterde yok (öneri).
