# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-E-01 · Bilgilendirme e-postası şablonu (içerik tasarımı)**

> Kaynak: İhtiyaç analizi §9.1 (N-01/N-03/N-04), §9.2, §10.2 PM-E-01, K-13, BR-PM-34/36/40, §12 (KVKK bildirim yüzeyi). Platform: **E-posta (ekran değil)** — gelen kutusunda web (≈ 1100 px pencere) ve mobil (402 px) önizlemesi. Alıcı: Veli (hesabı olan, e-posta adresi kayıtlı, e-posta tercihi açık). Önce `00-ortak-baglam.md`, sonra bu blok. **Bu şablon düğme, bağlantı ve katılım çağrısı İÇERMEZ** (karar K-13).

```
# İÇERİK TASARIMI: Veli toplantısı bilgilendirme e-postası (PM-E-01)

**Rol:** Alıcı Veli (Nermin Yalçın). **Amaç:** Veli e-postayı açtığında toplantının ne zaman, nerede, hangi çocuk için olduğunu net görsün; yanıtı OKSİS uygulamasından vereceğini anlasın. E-posta YALNIZ bilgilendirmedir.

## Üç şablon (aynı iskelet, farklı gövde) + düz metin sürümü
1. **N-01 Duyuru** — konu: "Veli toplantısı · 9-A · 20 Ekim 2026 18:30".
2. **N-03 Değişiklik** — iki gövde: (a) **Tarih/saat değişti** (yanıtlar sıfırlandı): konu "Veli toplantısı saati değişti · 6-B"; (b) **Yer değişti**: konu "Veli toplantısı yeri değişti · 9-B" (yanıt istenmez).
3. **N-04 İptal** — konu "Veli toplantısı iptal edildi · 10-C"; iptal GEREKÇESİ gövdeye YAZILMAZ.
Her şablon için: gelen kutusu satırı (gönderen adı "Altınay Lisesi · OKSİS", konu, ön izleme/preheader metni), açılmış e-posta (web ve mobil önizleme), düz metin (text/plain) sürümü.

## İskelet (yukarıdan aşağı, tek sütun, 600 px)
1. **Başlık bandı:** okul adı "Altınay Lisesi" (okul logosu yer tutucusu) ve "OKSİS" imzası; lacivert `#1B2B5E` bant, beyaz yazı. Gradyan yok. OKSİS logosu yalnız onaylı üç varyanttan biri (mark-alone) ve en az 40 px.
2. **Selamlama:** "Sayın Nermin Yalçın,"
3. **Ana cümle (şablona göre):**
   - N-01: "Defne Yalçın için veli toplantısı planlandı."
   - N-03 tarih: "Emir Yalçın için planlanan veli toplantısının tarih ve saati değişti. Önceki yanıtınız geçersiz oldu; lütfen yeni saate göre yanıtınızı OKSİS uygulamasından yeniden verin."
   - N-03 yer: "Planlanan veli toplantısının yeri değişti. Yanıtınız etkilenmedi."
   - N-04: "Planlanan veli toplantısı iptal edildi."
4. **Bilgi kartı** (açık yüzey `#EEF1FA`, 16 radius, etiket/değer çiftleri): Öğrenci · Şube · Tarih · Saat · Yer · Toplantı. Değişiklikte "Yeni" değerler vurgulu, "Önceki" değerler sönük üstü çizili küçük satır. İptalde tarih/saat "iptal edilen" olarak gösterilir.
5. **Açıklama bölümü** (N-01 ve tarih/yer değişikliğinde, idare yazdıysa): "Okul idaresinin notu:" + açıklama metni.
6. **Yönlendirme cümlesi (düğme DEĞİL, düz metin):** N-01 ve N-03'te "Yanıtınızı OKSİS uygulamasından verebilirsiniz." N-04'te bu cümle yoktur.
7. **Alt bilgi:** "Bu e-posta, OKSİS bildirim tercihlerinize göre gönderildi. Tercihlerinizi uygulamadaki Bildirim tercihlerim bölümünden değiştirebilirsiniz." — bağlantı/düğme YOK. Altında okul adı, okul iletişim bilgisi (kurgu telefon "0212 000 00 00"), "OKSİS" imzası.
Dil: Türkçe, "siz" hitabı, resmi ama sıcak, kısa cümle. Emoji yok.

## KESİNLİKLE OLMAYACAKLAR (KVKK + karar K-13)
- Düğme, "Katılacağım/Katılamayacağım" butonu, tıklanabilir bağlantı (hiçbir `href`: uygulama indirme bağlantısı dahil), "güvenli onay ekranı".
- Katılamama nedeni, görüşme notu, diğer velinin adı/yanıtı/varlığı, başka velilerin sayıları ("18 veli katılıyor").
- Gerekçe (iptal sebebi), öğretmenin kişisel iletişim bilgisi.
- Yapay zekâ ile üretilmiş metin; kampanya/pazarlama içeriği.
Çoklu çocuklu veliye her çocuk/şube için AYRI e-posta gider. Hesapsız veli ve e-posta adresi olmayan veliye e-posta gitmez (şablon yok).

## Tasarım kısıtları (e-posta istemcisi gerçeği)
Marka yazı tipi Plus Jakarta Sans e-posta istemcilerinde güvenilir değildir: web-safe yedek (Arial/Helvetica) belirt ve iki halini de göster. Yalnız marka renkleri (lacivert, mavi `#3A4F9C`, yüzey `#EEF1FA`, mürekkep `#0D1117`; durum tint'leri). Tablo tabanlı yerleşim varsayımı. Koyu mod istemcilerinde renk tersleşse de metin kontrastı ≥ 4.5:1 kalmalı. Resim kapalıyken anlam bozulmamalı (logo alt metni).

## Örnek veri
N-01: Defne Yalçın · 9-A · Salı, 20 Ekim 2026 · 18:30–19:30 · 9-A Sınıfı · 1. Dönem Veli Toplantısı. N-03 tarih: Emir Yalçın · 6-B · önceki 17:00–18:00 → yeni 18:00–19:00. N-03 yer: 9-B · önceki 9-B Sınıfı → yeni Spor Salonu. N-04: 10-C · Salı, 20 Ekim 2026 · 18:30 · iptal.

## Durumlar (çerçeveler)
Gelen kutusu satırı (okunmamış/okunmuş) · açılmış e-posta web ve mobil · düz metin sürümü · resim kapalı görünümü · koyu mod tersleşmesi · uzun açıklama metni (800 karakter, sarma) · uzun öğrenci adı (iki satıra sarma) · açıklama yok varyantı (bölüm hiç görünmez).

## Bu içerikte ÇİZME
Düğme, bağlantı, QR kod, takvim eki (.ics), mesaj yazma alanı, abonelikten çık bağlantısı, kampanya blokları, sosyal medya simgeleri, izleme pikseli.
```

## Sipariş sonrası kontrol listesi
- [ ] Hiçbir yerde düğme veya tıklanabilir bağlantı yok mu? (uygulama indirme bağlantısı dahil)
- [ ] Neden, not, diğer veli bilgisi, iptal gerekçesi, "N veli katılıyor" gibi sayı sızmamış mı?
- [ ] Yönlendirme düz cümle: "Yanıtınızı OKSİS uygulamasından verebilirsiniz."
- [ ] Tarih değişikliğinde "yanıtınız geçersiz, yeniden yanıt verin"; yer değişikliğinde "yanıtınız etkilenmedi" ayrımı var mı?
- [ ] Çocuk adı + şube + tarih-saat + yer + okul adı eksiksiz mi?
- [ ] Düz metin sürümü ve resim kapalı/koyu mod çerçeveleri var mı?
- [ ] Yalnız marka renkleri; gradyan/yabancı font yok mu?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi (e-posta için geçerli maddeler).

## Envanterde belirsiz / karar
- **Alt bilgide tercih bağlantısı:** K-13 "düğme/bağlantı yok" der; abonelikten çıkış/tercih bağlantısı hukuki gereklilik mi? Prompt bağlantısız metin kullanır. Teyit/KVKK değerlendirmesi.
- Gönderen adı/adresi ve okul logosunun e-postada kullanımı analizde yok.
- N-03'ün iki gövdesi tek tür; e-posta konu satırları öneridir.
- Çoklu çocuklu veliye çocuk başına e-posta (öneri; analiz bildirim başına şube toplantısı der).
