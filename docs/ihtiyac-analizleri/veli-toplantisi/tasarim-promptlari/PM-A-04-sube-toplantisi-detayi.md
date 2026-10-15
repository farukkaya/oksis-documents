# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-04 · Şube toplantısı detayı (İdare): yanıtlar, yanıtsızlar, hesapsızlar, yürüt**

> Kaynak: İhtiyaç analizi §6.1 (8), §6.4, §10.2 PM-A-04, BR-PM-14/16/19/20/21/24, K-6, K-7, K-8, §11. Platform: **Web + Mobil (eşit)**. Rol: İdare (rehber öğretmen "Yanıt yok" ve "Hesapsız" listelerini görür/arar, veli adına işaretleyemez — bkz. PM-T-02). Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN: Şube toplantısı detayı — idare (PM-A-04)

**Rol:** İdare (Neslihan Çetin). **Amaç:** Tek bir şube toplantısının (ör. 9-A, Salı 20 Ekim 18:30) velilerini yönetmek: kim ne yanıt verdi, kim hiç yanıt vermedi, kime telefonla ulaşıp onun adına yanıt girmeli, hesabı olmayan kim; toplantı başladıktan sonra öğretmen yoksa toplantıyı idarenin yürütmesi.

## Giriş
PM-A-03 şube satırı → "Aç". Rota /parent-meetings/[id]/sections/[sectionId]. Üstte geri oku → organizasyon.

## Üst bölüm (şube toplantısı kimliği)
"9-A · Veli Toplantısı" · durum rozeti · "Salı, 20 Ekim 2026 · 18:30–19:30 · 9-A Sınıfı" · sorumlu: "Sınıf Rehber Öğretmeni: Selma Arslan" (rehbersizse uyarı rozeti "Rehber öğretmen yok — toplantıyı siz yürütürsünüz") · Sürüm çipi (2 ve üstü) · eylem düğmeleri: **Tarih değiştir** · **İptal** (PM-A-05; terminal durumda gizli).
Özet sayaç satırı (beyan): Katılacağım 18 · Katılamayacağım 4 · Yanıt yok 10 (32 davetli veli). Başladı sonrası ikinci satır (fiili, ayrı rozet dili): Geldi 15.
Durum notları: Başladı → Note "Toplantı başladı. Veli yanıtları kilitlendi; yanıtı yalnız siz güncelleyebilirsiniz." · İptal → hata tint "Bu toplantı iptal edildi. Kayıtlar korunur." · Tamamlandı → "Toplantı tamamlandı. Katılım ve notlar 5 gün daha düzenlenebilir."

## Sekmeler
**Tümü (32)** · **Yanıt yok (7)** · **Hesapsız veliler (3)** · **Yürüt** (yalnız Başladı ve sonrasında görünür). Not: üstteki "Yanıt yok 10" sayacı = Yanıt yok sekmesindeki 7 hesaplı veli + 3 hesapsız veli; sekme başlığı altında bu toplamı açıklayan küçük ipucu ("10 veliden 3'ü hesapsız").

### Sekme 1 · Tümü
Öğrenci → veliler yapısı. Web: tablo — öğrenci adı (satır başlığı, altında numarası) · veli adı + yakınlık (anne/baba/vasi) · **yanıt rozeti** (Katılacağım / Katılamayacağım / Yanıt yok) + "İdare girdi" çipi (veli adına girildiyse) · neden ikonu (Katılamayacağım'da neden varsa yorum-balonu ikonu; üzerine gelince/dokununca neden metni) · yanıt zamanı · "Geçmiş" bağlantısı. Bir öğrencinin birden fazla velisi varsa öğrenci satırı grup başlığı, veliler alt satırlar (Defne Yalçın → Nermin Yalçın: Katılacağım; Kadir Yalçın: Yanıt yok). Aynı şubede iki kardeş varsa veli tek yanıt verir ve yanıt iki öğrenci satırında görünür (küçük "ortak yanıt" notu).
Toplantı Başladı olduktan sonra her satırın sonunda küçük fiili katılım kutucuğu (Geldi / işaretlenmedi / Gelmedi) görünür — beyan rozetinden ayrı dilde. Görüşme notu bu ekranda GÖSTERİLMEZ (not PM-T-02/03'tedir).
Süzgeç: yanıt (Tümü · Katılacağım · Katılamayacağım · Yanıt yok) · arama (öğrenci/veli adı).

### Sekme 2 · Yanıt yok
Yanıtı olmayan, hesabı olan davetli veliler. Her satır: veli adı · öğrenci · yakınlık · telefon (maskeli "0555 010 ** **", dokununca/hover'da tam) · **Ara** düğmesi (telefon ikonlu; web'de `tel:` bağlantısı, mobilde arama başlatır) · **Veli adına işaretle** düğmesi. Telefonu kayıtlı olmayan velide Ara devre dışı + "Telefon kayıtlı değil".
Üstte Note: "Bu veliler toplantı bildirimini almış olabilir. Telefonla ulaştıysanız yanıtı onlar adına işleyin."

### Sekme 3 · Hesapsız veliler
OKSİS hesabı olmayan davetli veliler. Satır: veli adı · öğrenci · telefon · Ara · Veli adına işaretle. Üstte Note (uyarı tonu): "Bu velilere uygulama içi bildirim, push ya da e-posta ulaşmaz; SMS bu sürümde gönderilmiyor. Telefonla arayıp yanıtlarını siz işleyin." Aynı kişi Yanıt yok sekmesinde TEKRAR görünmez.

### Sekme 4 · Yürüt (Başladı'dan itibaren)
PM-T-02 (Katılım takibi) ile AYNI işlev ve görünüm — sayaç, öğrenci/veli satırları, "Geldi" tek dokunuş, not paneli girişi (PM-T-03), "Toplantıyı tamamla". Üstte etiket: "Toplantıyı idare olarak yürütüyorsunuz". Bu sekme PM-T-02'nin bileşenini yeniden kullanır; yeniden çizme, çerçeveyi göster ve "bkz. PM-T-02" notu ekle.

## VELİ ADINA İŞARETLE (diyalog / panel)
Web: Dialog (orta boy). Mobil: alt panel (Sheet).
- Başlık: "Nermin Yalçın adına yanıt" · alt başlık "Defne Yalçın · 9-A · Salı, 20 Ekim 2026 18:30".
- Yanıt: iki büyük seçenek kartı "Katılacağım" / "Katılamayacağım" (segment, biri seçili).
- Katılamayacağım seçilirse: isteğe bağlı **Neden** alanı (çok satırlı, 300 karakter). Altında Note: "Neden yalnız sınıf rehber öğretmeni ve okul idaresi tarafından görülür."
- İsteğe bağlı **Kanal notu** (tek satır; örn. "Telefonla görüşüldü").
- Düğmeler: "Kaydet" / "Vazgeç". Kayıt sonrası Toast "Yanıt Nermin Yalçın adına kaydedildi"; satır rozeti güncellenir, "İdare girdi" çipi eklenir.
- Mevcut yanıtı güncelleme de aynı panelden (önceki değer gösterilir: "Önceki: Katılacağım").
- Başladı sonrasında da açılır (idare güncelleyebilir); veli kendi yanıtını artık değiştiremez.

## YANIT GEÇMİŞİ (yan panel / alt panel)
Satırdaki "Geçmiş" bağlantısı → web'de sağdan çekmece, mobilde Sheet. İçerik: dikey zaman çizgisi, en yeni üstte; her kayıt: tarih-saat · "Yanıt yok → Katılacağım" (önceki → yeni) · **giren** ("Nermin Yalçın (veli)" ya da "Neslihan Çetin (idare) · Sevinç Çınar adına") · kanal notu · **plan sürümü** çipi ("Sürüm 1"). Sürüm artınca "Plan değişti — yanıtlar sıfırlandı (Sürüm 2)" ayırıcı satırı. Silinemez, düzenlenemez (düzenle/sil düğmesi YOK).

## Örnek veri
- 9-A, 32 davetli veli. Tümü sekmesi: Defne Yalçın → Nermin Yalçın (Katılacağım, 15 Eki 09:12) ve Kadir Yalçın (Yanıt yok); Ahmet Kerem Polat → Sevgi Polat (Katılamayacağım — neden: "Vardiyam çakışıyor, öğretmenle telefonla görüşebilir miyiz?", 14 Eki 21:40); Bilge Naz Aydın → Hacer Aydın (Yanıt yok); Can Efe Sönmez → Murat Sönmez (Yanıt yok); Ece Su Korkmaz → Dilek Korkmaz (hesapsız); Furkan Aras → Ramazan Aras (hesapsız); Gökçe Tan → Ayhan Tan (hesapsız); Lale Sevgi Çınar → Sevinç Çınar (veli adına işaretlenmiş Katılacağım, "İdare girdi", kanal notu "Telefonla görüşüldü").
- Geçmiş örneği (Sevinç Çınar): "13 Eki 10:05 · Yanıt yok → Katılacağım · Neslihan Çetin (idare), Sevinç Çınar adına · Telefonla görüşüldü · Sürüm 1".

## Durumlar
- Yükleniyor: kimlik kartı + tablo iskeleti; sekme sayıları "…".
- Boş: "Yanıt yok" sekmesi boş → EmptyState "Tüm veliler yanıt verdi" (olumlu ton, tik ikonu). "Hesapsız veliler" boş → "Bu şubede hesabı olmayan veli yok". "Tümü" boş (şubede davetli veli yok) → EmptyState "Bu şubede davetli veli yok. Öğrenci–veli bağlantıları okul kayıtlarından gelir."
- Hata: "Tekrar dene". Veli adına kayıt hatası: panel içinde hata, panel kapanmaz.
- İzin yok: 403. Kapsam dışı/bulunamadı: 404.
- Eşzamanlılık: kayıt sırasında yanıt değişmişse Note "Bu yanıt az önce değişti. Güncel değeri görün." + yenile.
- İptal ve sezon kapalı: "Veli adına işaretle" devre dışı. Tamamlandı: yanıt kilitli, işaretleme devre dışı + ipucu (analizde Tamamlandı sonrası veli adına yanıt tanımlı değil).
- Davet düşmüş satır (veli ilişkisi iptal edildi / bilgi görme yetkisi kapandı): satır "Davet düştü" notuyla soluk, listede kalır, geçmişi açılabilir, işaretle devre dışı.

## Bu ekranda ÇİZME
Velilere mesaj/SMS gönderme düğmesi · "hatırlat" toplu bildirim düğmesi (yanıt çağrısı sistemce zamanlanır — N-07; elle tetik YOK) · diğer velinin yanıtını veliye gösteren görünüm · görüşme notu metnini bu ekranda gösterme · yanıt geçmişini düzenleme/silme.
```

## Sipariş sonrası kontrol listesi
- [ ] "Hesapsız veliler" sekmesinde SMS/bildirim gönderilebiliyormuş gibi bir düğme yok mu? Not net mi?
- [ ] Veli adına işaretle diyaloğunda yanıt + isteğe bağlı neden + isteğe bağlı kanal notu var mı?
- [ ] Yanıt geçmişi salt okunur mu, "giren" ve "kimin adına" ayrı mı, plan sürümü görünüyor mu?
- [ ] Katılamama nedeni yalnız idare/rehber yüzeyinde; "yalnız öğretmen ve idare görür" notu var mı?
- [ ] Yanıt rozeti ile fiili katılım kutucuğu ayrışıyor mu? "Gelmedi" toplantı Tamamlandı olmadan görünmüyor mu?
- [ ] "Yürüt" sekmesi yalnız Başladı'dan sonra, PM-T-02 bileşeniyle aynı mı?
- [ ] Telefon kısayolu (tel:) var; telefonsız veli durumu devre dışı mı?
- [ ] Sekme sayıları tutarlı mı (7 + 3 = 10)?
- [ ] Mobilde sekmeler yatay çip/segment, satırlar kart; zeminde yüzen metin yok mu?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- Tamamlandı sonrası "veli adına işaretle" analizde tanımsız; prompt devre dışı bırakır (teyit: düzenleme penceresinde izin verilsin mi?).
- Rehber öğretmenin "Yanıt yok / Hesapsız" listelerini görme/arama yüzeyi envanterde ayrı ekran değil; PM-T-02 içinde sekme olarak çizilir.
- Telefon maskesi analizde yok (öneri; Rehber modülü gösterimiyle uyum teyidi).
- "Hesapsız" ve "Yanıt yok" sayım mantığı: üst sayaçta Yanıt yok hesapsızları da içerir (raporla tutarlı mı? teyit).
