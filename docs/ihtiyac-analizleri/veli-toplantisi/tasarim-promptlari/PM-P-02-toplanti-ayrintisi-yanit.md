# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-P-02 · Toplantı ayrıntısı + yanıt (Veli)**

> Kaynak: İhtiyaç analizi §6.3, §10.2 PM-P-02, K-7, K-9, BR-PM-16/18/19/21/22/23, N-01/N-03/N-04/N-05/N-07, §11, §12. Platform: **Web + Mobil (eşit)**. Rol: Veli (Bronz `#B05A0A`). Bildirimlerin derin bağlantı hedefi budur. Önce `00-ortak-baglam.md`, sonra bu blok. Emsal biçim: `EKRAN 6 Çocuğumun Ödevleri (veli, HW-P-01).md` detay bölümü.

```
# EKRAN: Toplantı ayrıntısı ve yanıt — veli (PM-P-02)

**Rol:** Veli (Nermin Yalçın). **Amaç:** Bildirimden gelen veli, TEK dokunuşla toplantının tüm bilgisini (çocuk, şube, tarih-saat, yer, açıklama) görüp yanıtını vermesi; sonra gerekirse değiştirmesi. Yanlış çocuğun toplantısına yanıt vermesi mümkün olmamalı: çocuk adı ve şube EN ÜSTTE ve nettir.

## Giriş
Bildirim derin bağlantısı (N-01, N-03, N-04, N-05, N-07) ve PM-P-01 / PM-P-03 kartı. Mobil: ALT ekran, STopBar "Veli Toplantısı" (geri). Web: veli panelinde sayfa /parent-meetings/[sectionId] (PageHeader geri oku). Bildirimden gelinirse geri → Toplantılarım.

## İçerik (kartlar halinde; mobilde her metin kart içinde)
1. **Kimlik kartı:** çocuk avatarı + "**Defne Yalçın · 9-A**" (büyük) · altında okul adı "Altınay Lisesi" · durum rozeti (Yayınlandı: "Yanıt bekleniyor" ile birlikte okunur; Başladı; Tamamlandı; İptal).
2. **Toplantı bilgisi kartı:** başlık "1. Dönem Veli Toplantısı" · tarih ("Salı, 20 Ekim 2026") · saat ("18:30–19:30") · yer ("9-A Sınıfı") — her biri ikonlu satır; sınıf rehber öğretmeni adı ("Selma Arslan") bilgi satırı olarak (envanterde ayrı belirtilmemiş; kartta öneri, bkz. karar notu).
3. **Açıklama kartı:** idarenin yazdığı metin ("Dönem başı bilgilendirme ve öğrenci gelişim görüşmeleri. Lütfen kimlik kartınızı yanınızda getiriniz.").
4. **Bantlar** (kimlik kartının hemen altında, PM-P-01 ile aynı dil): Değişiklik ("Saat değişti — lütfen yeniden yanıt verin"; plan sürümü arttıysa), Çakışma ("Aynı saatte başka toplantınız var" + çakışan çocuk/şube/saat), İptal ("Bu toplantı iptal edildi"; yanıt alanı gizli; gerekçe yok), Kilit ("Toplantı başladı; yanıtınız kilitlendi").
5. **YANIT KARTI** (ekranın odağı):
   - Başlık "Katılımınız".
   - Yanıtsız: iki BÜYÜK düğme alt alta (mobil) / yan yana (web): "**Katılacağım**" (birincil, dolu) ve "**Katılamayacağım**" (çerçeveli). ≥ 52 pt yüksek.
   - "Katılamayacağım" seçilince kart genişler: isteğe bağlı **Neden** metin alanı (en çok 300 karakter, sayaç) + Note: "**Bu açıklamayı yalnız öğretmen ve okul idaresi görür.**" + "Yanıtı gönder" düğmesi. Neden yazmadan gönderilebilir.
   - "Katılacağım" seçilince hemen kaydedilir.
   - **Yanıt sonrası onay:** kart üstünde başarı tonlu onay satırı "**Yanıtınız kaydedildi**" (tik) + Toast; kart yanıtlı hale döner: "Yanıtınız: Katılacağım" + "Yanıtı değiştir" (bağlantı). Katılamayacağım yanıtında kendi nedeni görünür ("Nedeniniz: …").
   - "Yanıtı değiştir": seçenekler yeniden açılır; mevcut seçili; değiştirilince yeni onay.
   - İdare veli adına girmişse küçük bilgi satırı (öneri): "Bu yanıt, telefon görüşmeniz üzerine okul tarafından girildi. Dilerseniz değiştirebilirsiniz." 
6. **Alt bilgi** (sönük, kart içinde): "Yanıtınızı toplantı başlayana kadar değiştirebilirsiniz."

Veli diğer velinin (Kadir Yalçın) yanıtını, nedenini ya da yanıt verip vermediğini GÖRMEZ; bu ekranda "diğer veli" kavramı hiç geçmez. Çoklu çocuk: her çocuğun toplantısı kendi sayfasıdır; aynı şubede kardeş varsa tek yanıt ve kimlik kartında iki çocuk adı ("Ayşe Nur ve Elif Su Polat · 9-A").

## WEB yerleşimi (1440)
Veli paneli kabuğu; ortada tek sütun (maks. 640 px). Yanıt kartı sağ kolon yerine içerik akışında; geniş ekranda kimlik + bilgi kartı solda, yanıt kartı sağda yapışkan (iki sütun, 1100 px üstünde). Klavye: düğmeler odaklanabilir, Enter ile seçim; alan hatası ve onay `aria-live`.

## MOBİL yerleşimi (402×874)
STopBar + kaydırılabilir kartlar; YANIT kartı kaydırma sonunda kaybolmasın diye ekranın altında yapışkan eylem çubuğu olarak da çizilebilir: yanıtsızsa alt çubukta iki düğme sabit ("Katılacağım" / "Katılamayacağım"); Katılamayacağım → alt panel (Sheet) neden alanıyla. Klavye açıkken panel yukarı kayar.

## Örnek veri
- Defne Yalçın · 9-A · Salı, 20 Ekim 2026 · 18:30–19:30 · 9-A Sınıfı · açıklama yukarıda · rehber öğretmen Selma Arslan.
- Varyantlar: (a) Yanıt bekleniyor; (b) Katılamayacağım seçili, neden alanı açık, Note görünür; (c) Katılacağım kaydedildi + onay; (d) Yanıtlı + "Yanıtı değiştir"; (e) Değişiklik bandı (Emir Yalçın · 6-B, 17:00 → 18:00, Sürüm 2); (f) Çakışma bandı; (g) İptal; (h) Başladı — kilit; (i) Tamamlandı (yanıt salt okunur, düğme yok).

## Durumlar
- Yükleniyor: kart iskeletleri.
- Hata (yüklenemedi): kart içi hata + "Tekrar dene". Yanıt kaydedilemedi: yanıt kartı eski durumda, hata satırı "Yanıtınız kaydedilemedi" + "Tekrar dene"; yazılmış neden metni KAYBOLMAZ.
- Bulunamadı/kapsam dışı (404): başka çocuğun veya erişimi olmayan toplantıya derin bağlantı → "Bu toplantı bulunamadı." (Ayrıntı sızdırma yok.) Davet düşmüş (veli ilişkisi iptal / bilgi yetkisi kapalı): aynı 404 durumu.
- İzin yok/oturum süresi: mevcut genel kalıp.
- Çevrimdışı: yanıt düğmeleri devre dışı + Note "Bağlantı yok".
- Eşzamanlılık: yanıt gönderilirken tarih değiştiyse (sürüm arttı): Dialog/Sheet "Toplantı saati değişti. Lütfen yeni saate göre yanıt verin." ve sayfa güncellenir, yanıt gönderilmez.
- Başladı/Tamamlandı: kilit; İptal: düğmesiz.
- Sezon kapalı: salt okunur.

## Bu ekranda ÇİZME
Diğer velinin yanıtı/varlığı · fiili katılım · görüşme notu · takvime ekle · yol tarifi · öğretmene mesaj · "kaç veli katılıyor" sayaçları · e-postadan geçişli onay ekranı (e-postada bağlantı yoktur) · çocuğun (öğrencinin) görüşü.
```

## Sipariş sonrası kontrol listesi
- [ ] Çocuk adı + şube en üstte, büyük ve net mi?
- [ ] İki büyük düğme; Katılamayacağım'da neden isteğe bağlı + "yalnız öğretmen ve okul idaresi görür" notu var mı?
- [ ] "Yanıtınız kaydedildi" onayı var mı; "Yanıtı değiştir" yanıt sonrası görünüyor mu?
- [ ] Başladı sonrası yanıt kilitli (düğme yok) mu?
- [ ] Diğer velinin yanıtına/varlığına dair hiçbir ipucu yok mu?
- [ ] Kaydedilemeyen yanıtta yazılan neden metni kaybolmuyor mu?
- [ ] Mobilde zeminde yüzen metin yok, yapışkan eylem çubuğu içerik örtmüyor mu?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- Rehber öğretmen adının veliye gösterilmesi envanterde belirtilmemiş (öneri; kaldırılabilir).
- "İdare veli adına girdi" bilgisinin veliye gösterilmesi envanterde yok (şeffaflık önerisi; BR-PM-20 geçmişi veliye sunmaz).
- Tarih değişirken gönderilmekte olan yanıt çakışması (eşzamanlılık) davranışı analizde ayrıntılı değil; öneri.
- Yapışkan alt eylem çubuğu vs. akış içi yanıt kartı (mobil) — Claude Design'ın kararı; seçimini gerekçesiyle yazması istenir.
- Takvime ekle/yol tarifi gibi yardımcı eylemler envanterde yok; çizilmez.
