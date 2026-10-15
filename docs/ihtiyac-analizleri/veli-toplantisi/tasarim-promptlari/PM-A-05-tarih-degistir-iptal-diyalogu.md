# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-05 · Tarih değiştir / İptal diyaloğu (etki önizlemeli)**

> Kaynak: İhtiyaç analizi §6.4, §7.2, §10.2 PM-A-05, BR-PM-04/13/17/18/23/35, K-9, N-03/N-04, §11. Platform: **Web (Dialog) + Mobil (Sheet)** — eşit. Rol: İdare. Bu bir **diyalog ailesi**dir (tam sayfa değil); PM-A-03 satır/toplu eylemlerinden ve PM-A-04 üst düğmelerinden açılır. Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN AİLESİ: Tarih değiştir ve İptal diyalogları (PM-A-05)

**Rol:** İdare. **Amaç:** Tek bir şube toplantısının (ya da seçili birkaç şubenin) tarih/saat/yerini değiştirmek veya iptal etmek — ve bunu kaydetmeden ÖNCE sonucunu net görmek: kaç yanıt sıfırlanacak, kaç veliye bildirim gidecek, kaç yeni çakışma doğacak. Sıfırlama SESSİZ DEĞİLDİR; idare bunu bilerek onaylar.

Web: orta boy Dialog (toplu modda geniş). Mobil: tam yükseklik Sheet (tarih değiştir) ve ortalanmış ConfirmSheet (iptal). Dört çerçeve: A) Tarih değiştir — tek şube, B) Tarih değiştir — toplu, C) İptal — tek, D) İptal — toplu.

## A) TARİH DEĞİŞTİR — tek şube
Başlık: "Tarih ve saati değiştir" · alt başlık: "6-B · Hasan Yıldırım · şu an Salı, 20 Ekim 2026 17:00–18:00 · 6-B Sınıfı".
Alanlar: **Yeni tarih** · **Yeni başlangıç saati** · **Yeni bitiş saati** (isteğe bağlı) · **Yer** (mevcut yerle dolu gelir). Değişmeyen alan soluk, değişen alan vurgulu ("Eski → Yeni" mini farkı).
Doğrulama: yeni başlangıç anı geçmişse satır hatası "Geçmiş bir an seçilemez"; bitiş başlangıçtan önceyse hata. Hiçbir şey değişmediyse "Kaydet" devre dışı.
Başladı durumundaki şube (erteleme): yalnız hiç fiili katılım ve not girilmemişse mümkün; Note "Toplantı başlamış görünüyor; ertelenirse yeniden Yayınlandı olur." Fiili katılım veya not varsa form yerine engel durumu: hata tint Note "Katılım veya not girilmiş bir toplantı ertelenemez. İptal edebilir ya da yeni bir şube toplantısı planlayabilirsiniz." (alanlar devre dışı).

### ETKİ ÖNİZLEMESİ (kritik bölüm; alanların altında, değişiklik girildikçe canlı)
Kart başlığı "Bu değişiklik ne yapacak?". İki durumlu:
1) **Tarih veya saat değişiyorsa** (Plan sürümü 1 → 2):
   - "**29 yanıt sıfırlanacak**" (vurgulu sayı, uyarı tonu; alt satır: "Katılacağım 24 · Katılamayacağım 5 — eski yanıtlar geçmişte kalır, yeniden yanıt istenecek").
   - "**36 veliye değişiklik bildirimi gidecek**" (alt satır: "Uygulama içi + push; e-posta tercihi açık 22 veliye ayrıca e-posta. Hesabı olmayan 4 veliye ulaşılamaz.").
   - "**2 veliye yeni çakışma**" (uyarı tonu; açılır liste: "Nermin Yalçın ve Kadir Yalçın — 6-B 18:00–19:00 ile 9-A 18:30–19:30 kesişiyor"). Çakışma engellemez.
   - Rehber öğretmen satırı: "Hasan Yıldırım'a değişiklik bilgisi gidecek."
   - Not: "Veliler 'Toplantı saati değişti, lütfen yeniden yanıt verin' bildirimi alacak."
2) **Yalnız yer (ve/veya açıklama) değişiyorsa**: "**0 yanıt sıfırlanacak**" (başarı tonu) + "36 veliye yer değişikliği bildirimi gidecek (yanıt istenmez)". Yalnız başlık/açıklama düzeltmesinde bildirim gitmez (sessiz, denetim kaydına yazılır): önizlemede "Bildirim gitmeyecek" notu.
Önizleme hesaplanırken iskelet; hesaplanamazsa hata + "Tekrar dene" ve "Kaydet" devre dışı. Onaysız değişiklik YAPILAMAZ: "Kaydet" önizleme görünür olmadan etkinleşmez.
Alt düğmeler: "Kaydet · 36 veliye bildir" (birincil) / "Vazgeç".
Başarı sonrası Toast: "6-B toplantısı güncellendi. 36 veliye bildirim gönderildi." Tabloda satır "Sürüm 2" çipiyle güncellenir.

## B) TARİH DEĞİŞTİR — toplu (seçili birkaç şube)
Başlık: "3 şubenin tarihini değiştir" · alt başlık seçili şubeler (6-A, 6-B, 7-A). Alanlar: hangi alanların uygulanacağı (Tarih ☑ · Başlangıç ☑ · Bitiş ☐ · Yer ☐) + yeni değerler.
**Şube kırılımlı önizleme tablosu:** Şube · Eski → Yeni · Sıfırlanacak yanıt · Bildirim gidecek veli · Yeni çakışma. Toplam satırı kalın ("Toplam: 82 yanıt sıfırlanacak · 109 veliye bildirim · 2 veliye yeni çakışma"). Terminal durumdaki (Tamamlandı/İptal) şube seçilmişse tabloda gri satır "Atlanacak — terminal durum" (savunmacı durum). Mobilde tablo, şube kartı listesine döner (her kartta üç sayı).
Düğme: "3 şubeyi kaydet · 109 veliye bildir".

## C) İPTAL — tek şube
Başlık: "Toplantıyı iptal et" · tehlike tonlu Dialog (tone=danger) · alt başlık "10-C · Cemil Aksoy · Salı, 20 Ekim 2026 18:30".
- **İptal gerekçesi** (isteğe bağlı, çok satırlı) + etiket **"Yalnız idare görür — velilere gönderilen bildirimde yer almaz"**.
- Bilgi satırı: "**32 veliye iptal bildirimi gidecek**" (alt: "Uygulama içi + push; e-posta tercihi açık 20 veliye ayrıca e-posta. Hesabı olmayan 3 veliye ulaşılamaz."). Rehber öğretmene de bilgi gider.
- Not: "Yanıtlar ve geçmiş silinmez; kayıt raporlarda 'iptal' olarak ayrı sayılır. Bu işlem geri alınamaz; yeniden planlama yeni bir şube toplantısıdır."
- Düğmeler: "İptal et · 32 veliye bildir" (tehlike tonu) / "Vazgeç".
- Rehber öğretmeni yoksa ek satır: "Bu şubede rehber öğretmen yok; öğretmen bildirimi gitmeyecek."

## D) İPTAL — toplu
Başlık "3 şubeyi iptal et". Gerekçe (tek alan, seçili tüm şubeler için ortak, isteğe bağlı, "Yalnız idare görür"). Şube kırılımlı liste: şube · bildirim gidecek veli sayısı; toplam satırı "Toplam: 96 veliye iptal bildirimi gidecek". Düğme "3 şubeyi iptal et · 96 veliye bildir". Kısmi başarısızlıkta sonuç diyaloğu: "2 şube iptal edildi, 1 şube güncellenemedi (kayıt değişti)"; başarısız şube işaretli.

## Örnek veri
- 6-B tarih değişikliği: 17:00–18:00 → 18:00–19:00 · 36 davetli veli · 29 yanıt sıfırlanacak (24 Katılacağım + 5 Katılamayacağım) · 22 e-posta tercihli · 4 hesapsız · 2 veliye yeni çakışma (Yalçın ailesi).
- İptal: 10-C, Cemil Aksoy, gerekçe "Rehber öğretmen raporlu", 32 veli, 20 e-posta tercihli, 3 hesapsız.

## Durumlar
- Önizleme hesaplanıyor: iskelet satırları; Kaydet devre dışı.
- Hata: önizleme alınamadı → "Etki hesaplanamadı. Tekrar deneyin." Kayıt hatası → diyalog açık kalır, hata Note'u.
- Eşzamanlılık: "Bu şube toplantısı siz işlem yaparken değişti (Sürüm 3). Kayıt yenilendi; etkiyi yeniden gözden geçirin." — önizleme yenilenir, onay sıfırlanır.
- Çifte işlem: tarih değiştirilirken şube iptal edilmişse "Bu toplantı iptal edilmiş; tarih değiştirilemez."
- İzin yok: eylem düğmeleri rehber öğretmende hiç görünmez.
- Sezon kapalı / Tamamlandı / İptal: eylemler gizli.
- Çevrimdışı: Kaydet devre dışı + "Bağlantı yok".

## Bu ekranda ÇİZME
Tarih değişikliğinde yanıtları SIFIRLAMAMA seçeneği (sıfırlama kuraldır) · iptal bildirimine gerekçe ekleme seçeneği · "bildirim gönderme" kapatma anahtarı · iptali geri alma · başka öğretmen atama · velilere özel mesaj yazma.
```

## Sipariş sonrası kontrol listesi
- [ ] Etki önizlemesi üç sayıyı veriyor mu (sıfırlanacak yanıt · bildirim gidecek veli · yeni çakışma) ve "Kaydet" önizleme olmadan etkinleşmiyor mu?
- [ ] Sıfırlama "sessiz değil": velilere "yeniden yanıt verin" bildirimi gideceği yazılı mı?
- [ ] Yalnız yer değişikliğinde "0 yanıt sıfırlanacak" gösteriliyor mu?
- [ ] İptal gerekçesinde "Yalnız idare görür — bildirimde yer almaz" etiketi var mı?
- [ ] Hesabı olmayan veliye "ulaşılamaz" uyarısı var; SMS gidecek izlenimi yok mu?
- [ ] Toplu modda şube kırılımlı tablo + toplam satırı var mı; mobilde kart listesine dönmüş mü?
- [ ] Başladı + katılım/not varsa erteleme engelleniyor mu?
- [ ] İptal geri alınamaz uyarısı ve "yeniden planlama = yeni şube toplantısı" notu var mı?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- Çakışma sayısının birimi: analiz "yeni çakışma Z" der; prompt "çakışan veli sayısı" varsayar (teyit).
- Rehber öğretmene giden N-02'nin önizlemede gösterilip gösterilmeyeceği analizde yok; prompt gösterir.
- Başladı'dan erteleme koşulunun (hiç katılım/not yok) arayüz anlatımı analizde yok; prompt engel durumunu önerir.
- Toplu tarih değişikliğinde alan bazlı uygulama PM-A-02 "seçilenlere uygula" ile tutarlı varsayıldı.
- Yalnız yer değişikliğinde N-03 yer gövdesi gider (§9.1) — önizleme metni önerisidir.
