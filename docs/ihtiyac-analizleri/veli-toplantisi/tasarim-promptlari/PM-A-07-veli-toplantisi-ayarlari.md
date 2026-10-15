# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-07 · Veli Toplantısı ayarları (okul ayarı eki)**

> Kaynak: İhtiyaç analizi §10.2 PM-A-07, K-12, K-15, BR-PM-12/30/33/38/39, §9.1 (N-05/N-06/N-07), §16 S-1/S-3. Platform: **Web + Mobil (eşit)**. Rol: İdare (yalnız ayarı değiştirir). Önce `00-ortak-baglam.md`, sonra bu blok. Emsal: mevcut "Okul Ayarları" ekranlarının politika kartları (Ödev Politikası vb.) — aynı kabuğu ve kart kalıbını fork et.

```
# EKRAN: Veli Toplantısı ayarları (PM-A-07)

**Rol:** İdare (okul yöneticisi + müdür yardımcısı). **Amaç:** Modülün okul genelindeki üç ayarını (hatırlatma saati ve bildirimleri, başlangıç bildirimi, görüşme notu saklama süresi) tek yerde, sonuçlarını açıkça yazarak yönetmek; sistem sabitlerini (otomatik kapanış, düzenleme penceresi) bilgi olarak görmek.

## Giriş
Web: Okul Ayarları içinde "Veli Toplantısı" kartı/sayfası (rota /settings/parent-meetings; Ayarlar yan menüsünde "Politika" grubunun yanında). Mobil: Daha fazla → Okul Ayarları → Veli Toplantısı (ALT ekran, STopBar "Veli Toplantısı"). Değiştiren yalnız idare; diğer roller bu ekranı görmez.

## Ayar kartları (yukarıdan aşağı; her kart bir yüzey)
1. **Toplantı günü hatırlatması** (N-05)
   - Alan: **Sabah hatırlatma saati** (saat seçici; varsayılan 08:00). Açıklama: "Toplantı günü bu saatte 'Katılacağım' diyen velilere ve sınıf rehber öğretmenine hatırlatma gider. Yayın, o günün hatırlatma saatinden sonra yapılırsa hatırlatma gönderilmez."
   - **Sessiz saat uyarısı:** seçilen saat okulun sessiz saat aralığına (örnek 22:00–07:00) düşerse Note (uyarı tonu): "Bu saat sessiz saat aralığında. Push bildirimi sessiz saat bitişine ertelenir; uygulama içi bildirim hemen gelir." Saat dışarıdaysa uyarı yok.
2. **Yanıt bekleniyor hatırlatması** (N-07)
   - Alan: **Aç/kapa anahtarı** (varsayılan AÇIK). Açıklama: "Toplantıdan 2 gün önce, o şube toplantısına henüz yanıt vermemiş velilere bir kez hatırlatma gider." Gün sayısı (2) sabit bilgi çipi olarak gösterilir; değiştirilemez (teyit gerekir — bkz. karar notu).
3. **Toplantı başlıyor bildirimi** (N-06)
   - Alan: **Aç/kapa anahtarı** (varsayılan KAPALI). Açıklama: "Başlangıç anında 'Katılacağım' diyen velilere 'Veli toplantısı başlıyor' bildirimi gider." Altında bilgi satırı: "Başlangıç anı sessiz saatteyse push bildirimi gönderilmez (ertelenmez); uygulama içi kayıt yazılır."
4. **Görüşme notu saklama süresi** (K-12)
   - Alan: **Saklama süresi (yıl)** — sayı adım seçici 0–10, varsayılan 2. Açıklama: "Sezon kapatıldıktan (arşivlendikten) sonra bu süre dolunca görüşme notlarının METNİ kalıcı olarak silinir. Katılım kayıtları ve denetim kaydı kalır; denetim kaydı not metni içermez."
   - Kural satırları: 0 seçilirse Note (uyarı): "0 yıl: sezon kapatıldığında notlar hemen silinir." En çok 10.
   - Okuldan ayrılan öğrencinin notları için aynı süre işler (bilgi satırı).
   - **Süre kısaltma onayı:** mevcut değerden KÜÇÜK bir değer kaydedilirken ConfirmDialog (tone=warn): "Saklama süresi 2 yıldan 1 yıla düşürülüyor. Süresi dolan notlar gece çalışan işlemle KALICI olarak silinir ve geri getirilemez." Düğmeler: "Süreyi değiştir" / "Vazgeç".
5. **Sistem sabitleri (salt okunur bilgi kartı, düzenlenemez)** — kilit ikonu:
   - "Otomatik kapanış: Toplantı başladıktan 3 gün sonra tamamlanmamışsa sistem kapatır. Girilmiş notlar ve katılım korunur."
   - "Düzenleme penceresi: Toplantı tamamlandıktan sonra 7 gün boyunca katılım ve notlar düzeltilebilir; sonrası salt okunur."
   - Not: "Bu değerler sistem sabitidir; okul ayarı değildir."

Üstte kısa tanıtım kartı: "Bu ayarlar tüm şube toplantıları için geçerlidir." Alt çubuk: "Kaydet" (değişiklik yokken devre dışı) / "Değişiklikleri geri al". Kaydedilmemiş çıkış uyarısı. Kayıt sonrası Toast "Veli Toplantısı ayarları kaydedildi".

## WEB yerleşimi (1440)
Ayarlar kabuğu (sol ayar menüsü + içerik). PageHeader "Veli Toplantısı" alt başlık "Okul ayarları". İçerik tek sütun (maks. 760 px); beş kart dikey. Sağda sticky kısa "Bu ayar neyi etkiler?" özet paneli (isteğe bağlı): bildirim türü adları (N-05/N-06/N-07 yerine kullanıcı dilinde: "Toplantı günü hatırlatması", "Yanıt bekleniyor", "Toplantı başlıyor").

## MOBİL yerleşimi (402×874)
STopBar + beş kart dikey. Saat ve yıl seçiciler yerel picker alt paneli (Sheet). Toggle bileşeni (44pt). Kaydet sabit alt çubukta. Uyarı Note'ları kartın içinde.

## Örnek veri
Sabah saati 08:00 (varsayılan) · N-07 açık · N-06 kapalı · saklama 2 yıl. Uyarı varyantı: saat 06:30 seçildi (sessiz saat 22:00–07:00 içinde). Kısaltma onayı varyantı: 2 → 1 yıl. 0 yıl uyarı varyantı.

## Durumlar
- Yükleniyor: kart iskeletleri.
- Hata: yüklenemedi → kart içi hata + "Tekrar dene"; kaydetme hatası → Toast hata, değerler korunur.
- İzin yok: 403 "Bu ayarları yönetme yetkiniz yok" (öğretmen/veli bu ekranı görmez).
- Eşzamanlılık: "Bu ayarlar başka bir yönetici tarafından değiştirildi. Yenileyin." Note + "Yenile".
- Sezon kapalı: ayarlar okul genelidir, kapalı sezondan etkilenmez (bu durum çizilmez).
- Çevrimdışı: Kaydet devre dışı.

## Bu ekranda ÇİZME
Otomatik kapanış gününü / düzenleme penceresini düzenleme alanı (sistem sabiti) · e-posta kanalı ayarı (e-posta tercihi alıcıdadır, PM-X-01) · SMS ayarları (gönderim yok) · şube bazlı ayar · hatırlatma metni düzenleme · yapay zekâ.
```

## Sipariş sonrası kontrol listesi
- [ ] Saklama süresi 0–10 aralığında; 0 uyarısı ve kısaltma onayı var mı?
- [ ] Otomatik kapanış (3 gün) ve düzenleme penceresi (7 gün) SALT OKUNUR bilgi olarak mı?
- [ ] Sessiz saat uyarısı yalnız push'a atıf yapıyor mu (uygulama içi/e-posta ertelenmez)?
- [ ] N-06 varsayılan KAPALI, N-07 varsayılan AÇIK mı?
- [ ] Sistem sabit kartında kilit ikonu var, düzenle düğmesi yok mu?
- [ ] Mobilde zeminde yüzen metin yok, toggle/picker 44pt mi?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **N-07 ayarı:** envanterdeki PM-A-07 alan listesi (hatırlatma saati, başlangıç bildirimi, saklama) N-07 aç/kapa alanını içermez; §16 S-1 "okul ayarıyla kapatılabilir" der. Prompt aç/kapa ekler, gün sayısını (2) sabit gösterir. Teyit: gün sayısı ayar mı?
- **Saklama süresi değiştirme etkisi:** yeni süre mevcut notlara da (geriye dönük) uygulanır mı? Prompt "uygulanır" varsayıp kısaltmada onay ister. Teyit.
- **Sessiz saat aralığı** kaynağı (okul ayarı mı, kullanıcı mı?) bu ekranda salt okunur örnek gösterilir; kaynak netleşmeli.
- Ayarın menüdeki yeri (Politika grubu mu, ayrı kart mı?) belirsiz.
