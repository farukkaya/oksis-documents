# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-P-01 · Toplantılarım (Veli — çocuk başına kartlar)**

> Kaynak: İhtiyaç analizi §6.3, §10.2 PM-P-01, K-7, BR-PM-14/16/19/21/22/23, N-01/N-03/N-04, §11. Platform: **Web + Mobil (eşit)** — mobil birincil kullanım. Rol: Veli (portal rengi Bronz `#B05A0A`). Öğrenci bu ekranı görmez. Önce `00-ortak-baglam.md`, sonra bu blok. Emsal biçim: `docs/ihtiyac-analizleri/odev/EKRAN 6 Çocuğumun Ödevleri (veli, HW-P-01).md` (kart anatomisi, "zemin üzerine metin yok" kuralı, durum çerçeveleri).

```
# EKRAN: Toplantılarım — veli (PM-P-01)

**Rol:** Veli (Nermin Yalçın; çocukları Defne Yalçın 9-A ve Emir Yalçın 6-B). Teknoloji meraklısı değil, telefondan hızlı yanıt vermek ister. **Amaç:** "Ne zaman, nerede, ben gelebilecek miyim?" sorusuna 10 saniyede yanıt vermek; iki çocuğun toplantısını KARIŞTIRMAMAK; plan değişirse yeniden yanıt verebilmek.

## Giriş
Bildirim (N-01/N-03/N-04) → PM-P-02 detayı ya da bu liste; veli anasayfası kartı (PM-P-03); "Daha fazla" → "Veli Toplantıları" (konumu Faruk'la netleşecek). Mobil: ALT ekran, STopBar "Toplantılarım". Web: sol menüde "Veli Toplantıları" (veli web paneli).

## Yerleşim ve kart anatomisi
**Çocuk başına AYRI kart.** Aynı çocuğun aynı toplantısı tek kart; çoklu çocuk = çoklu kart. Her kartın üstünde çocuk avatarı (baş harf) + **çocuk adı ve şube HER ZAMAN yazar** ("Defne Yalçın · 9-A"). Kart içeriği:
1. Çocuk satırı: avatar + "Defne Yalçın · 9-A".
2. Başlık: organizasyon başlığı ("1. Dönem Veli Toplantısı").
3. Tarih-saat: "Salı, 20 Ekim 2026 · 18:30–19:30" (okunur, büyük) · yer: "9-A Sınıfı" (konum ikonu).
4. **Yanıt durumu** (veli yüzeyinde dil): "Yanıt bekleniyor" (uyarı tint) / "Katılacağım" (başarı tint, tik) / "Katılamayacağım" (NÖTR, çarpı).
5. **Eylem alanı:**
   - Yanıtsızsa: iki BÜYÜK düğme kartın içinde yan yana — "Katılacağım" (birincil, dolu) ve "Katılamayacağım" (ikincil, çerçeveli). Her biri ≥ 48 pt yüksek. "Katılacağım"a dokunmak yanıtı hemen kaydeder (kart anında "Katılacağım" durumuna döner + Toast "Yanıtınız kaydedildi"). "Katılamayacağım"a dokunmak kartı genişletir (web: satır içi açılır, mobil: alt panel): isteğe bağlı **Neden** alanı + not "Bu açıklamayı yalnız öğretmen ve okul idaresi görür" + "Yanıtı gönder" (neden yazmadan da gönderilebilir).
   - Yanıtlıysa: durum + **"Yanıtı değiştir"** bağlantı/düğmesi (kartta her zaman; toplantı Başladı olana dek). Katılamayacağım yanıtında veli kendi nedenini görür (kısa kırpık, "Nedeniniz: Vardiyam çakışıyor…").
6. **Bantlar** (kart içinde üst şerit, birden fazlası olabilir):
   - **Değişiklik** (bilgi tint, saat ikonu): "Saat değişti — lütfen yeniden yanıt verin" (plan sürümü arttı, yanıt sıfırlandı; kart "Yanıt bekleniyor"a döner; eski tarih/saat üstü çizili küçük "Eski: 17:00" bilgisi OPSİYONEL).
   - **Çakışma** (uyarı tint, çift-takvim ikonu): "Aynı saatte başka toplantınız var" + altında hangi çocuk/şube ("Emir Yalçın · 6-B · 18:00–19:00"). Engellemez; kart yine yanıtlanabilir.
   - **İptal** (hata tint, iptal ikonu): "Bu toplantı iptal edildi"; düğmeler YOK; gerekçe GÖSTERİLMEZ; kart "Geçmiş" bölümüne düşer (iptal bandıyla görünür kalır).
   - **Yanıt kilitli** (nötr, kilit ikonu): toplantı Başladı → "Toplantı başladı; yanıtınız kilitlendi" ("Yanıtı değiştir" yok).
7. Ayrıntı için tüm kart dokunulabilir → PM-P-02.

**Bölümler:** Yaklaşan (tarihe göre artan) · Geçmiş (Tamamlandı + İptal; daraltılmış, "Geçmiş toplantılar" başlığı kendi yüzey şeridinde). Mobilde bölüm başlıkları zeminde yüzmez: grubun kart yığınının başlık satırında ya da ince yüzey şeridinde durur.
Veli YALNIZ kendi yanıtını görür; aynı çocuğun diğer velisinin (Kadir Yalçın) yanıtı/nedeni/varlığı HİÇBİR yerde gösterilmez. Fiili katılım ("geldi/gelmedi") veliye gösterilmez; not hiçbir biçimde görünmez.

## WEB yerleşimi (1440, veli paneli)
Ortada tek sütun (maks. 720 px), PageHeader "Toplantılarım" alt başlık "2026–2027 · 1. Dönem". Kartlar dikey; büyük ekranda iki çocuk kartı yan yana değil alt alta (kıyas/karışma önlenir). Katılamayacağım genişlemesi satır içi.

## MOBİL yerleşimi (402×874)
Alt (stack) ekran STopBar. Kartlar tam genişlik, iç boşluk 16; düğmeler tam genişliğin yarısı (her biri ≥ 48 pt). Katılamayacağım → alt panel (Sheet, içerik kadar). Pull-to-refresh.

## Örnek veri
- Kart 1 — Defne Yalçın · 9-A · "1. Dönem Veli Toplantısı" · Salı, 20 Ekim 2026 · 18:30–19:30 · 9-A Sınıfı · **Katılacağım** (başarı) · "Yanıtı değiştir" · çakışma bandı: "Aynı saatte başka toplantınız var — Emir Yalçın · 6-B · 18:00–19:00".
- Kart 2 — Emir Yalçın · 6-B · aynı başlık · Salı, 20 Ekim 2026 · 18:00–19:00 · 6-B Sınıfı · **Yanıt bekleniyor** · değişiklik bandı "Saat değişti — lütfen yeniden yanıt verin" (17:00 → 18:00) · çakışma bandı · iki büyük düğme.
- Geçmiş: Defne Yalçın · 9-A · "Sezon Başı Tanışma Toplantısı" · Salı, 22 Eylül 2026 · Tamamlandı · Yanıtınız: Katılacağım.
- İptal örneği (başka veli, ayrı çerçeve): "Bu toplantı iptal edildi" bandı, düğmesiz.
- Katılamayacağım örneği (ayrı veli: Sevgi Polat): "Katılamayacağım" nötr rozet; "Nedeniniz: Vardiyam çakışıyor, öğretmenle telefonla görüşebilir miyiz?".

## Durumlar
- Yükleniyor: kart iskeletleri (2 adet).
- Boş: çocuğa yönelik hiç toplantı yok → boş ekran kalıbı bir kart içinde: "Şu an veli toplantısı yok" + "Okul toplantı planladığında burada göreceksiniz." (çocuk adı geçmeden genel).
- Çocuksuz/bağlantısız veli: mevcut "NoLinkedChildNotice" kalıbı (çocuk kapısı) — yeniden çizme, yalnız bu ekranda gösterim yerini işaretle.
- Bilgi yetkisi kapalı çocuk (ayrı yaşayan ebeveyn, `CanViewInfo` kapalı): o çocuğun kartı HİÇ görünmez; bu durum için "çocuğunuz için toplantı yok" gibi bir metin de yazma.
- Hata: kart içinde hata + "Tekrar dene". Yanıt kaydı hatası: kart eski durumda kalır + Toast "Yanıtınız kaydedilemedi. Tekrar deneyin." (düğmeler yeniden etkin).
- İzin yok/oturum: mevcut genel kalıp.
- Çevrimdışı: yanıt düğmeleri devre dışı + Note "Bağlantı yok — yanıtınızı iletmek için bağlantı gerekir" (yerel kuyruğa alma YOK).
- Sezon kapalı: yaklaşan yok; geçmiş görünür.
- Başladı sonrası kilit: yukarıdaki kilit bandı.
- Hesapsız veli: bu ekranı hiç göremez (hesabı yok) — çizilmez.

## Bu ekranda ÇİZME
Diğer velinin yanıtı/varlığı · fiili katılım (geldi/gelmedi) · görüşme notu · öğretmenin yanıt sayaçları ("sınıfın %70'i katılacak" gibi kıyas) · e-posta yanıt bağlantısı · takvime ekle (envanterde yok) · yol tarifi/harita · mesaj gönderme.
```

## Sipariş sonrası kontrol listesi
- [ ] Çocuk adı + şube her kartta yazılı mı; iki çocuk kartı karışmıyor mu?
- [ ] Yanıtsız kartta iki büyük düğme doğrudan kartta mı (≥ 48pt)?
- [ ] "Katılamayacağım" nötr (kırmızı/uyarı değil) mi; "Yanıt yok" yerine "Yanıt bekleniyor" mu?
- [ ] Katılamayacağım'da neden isteğe bağlı ve "yalnız öğretmen ve okul idaresi görür" notu var mı?
- [ ] Başladı sonrası yanıt kilidi, değişiklik/çakışma/iptal bantları ayrı çizilmiş mi?
- [ ] Diğer velinin yanıtı, fiili katılım, not sızmamış mı?
- [ ] Mobilde zeminde yüzen metin / bölüm başlığı yok mu?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **Aktif çocuk bağlamıyla ilişki:** header'daki çocuk seçici (switch-child) seçili çocukla süzer mi, yoksa tüm çocuklar mı listelenir? Envanter "çocuk başına kartlar" der; prompt TÜM çocukları gösterir. Teyit.
- Katılamayacağım'ın kartta tek dokunuşla mı genişleme/panel ile mi alındığı envanterde belirtilmemiş (öneri: genişleme).
- "Katılacağım" tek dokunuş geri alma/onay ihtiyacı (yanlışlıkla dokunma) — prompt anında kaydeder; "Yanıtı değiştir" her zaman var; ek onay yok.
- Fiili katılımın veliye gösterilip gösterilmeyeceği envanterde yok; prompt göstermez.
- Menü/giriş noktası ve "takvime ekle" ihtiyacı belirsiz.
- Veli web panelinin tasarım dili (web prototipinde veli ekranı yok) — duyurularda da eksik bırakılmış; yönü Faruk'la netleşmeli.
