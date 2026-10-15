# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-T-03 · Görüşme notu paneli**

> Kaynak: İhtiyaç analizi §6.2 (5), §10.2 PM-T-03, K-11, K-12, BR-PM-10/26/27/28/29/30/31/32/33, §11, §12 (KVKK). Platform: **Web (yan panel) + Mobil (alttan tam yükseklik panel)** — eşit. Roller: Sınıf Rehber Öğretmeni + idare ("Yürüt"). Veli ve öğrenci bu paneli ASLA görmez. Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN: Görüşme notu paneli (PM-T-03)

**Rol:** Sınıf Rehber Öğretmeni (Selma Arslan) — idare "Yürüt" modunda aynı panel. **Amaç:** Veliyle görüşürken, öğrenci başına kısa bir not yazmak; yazdıkça KAYBOLMAMASI (otomatik kayıt, kesintiye dayanıklı); notun veliye sızacağı korkusunu her an görünür bir etiketle gidermek.

## Giriş
PM-T-02'de öğrenci satırına dokununca açılır. Web: PM-T-02'nin sağ bölmesinde yan panel (≈ 40% genişlik, sağdan; sayfa kaymaz). Mobil: alttan açılan TAM YÜKSEKLİK panel (Sheet size="full"; kapatma "Kapat" ya da aşağı çekme; klavye açıkken metin alanı görünür kalır, etiket ve durum şeridi yukarıda sabit).

## Panel anatomisi (yukarıdan aşağı)
1. **Başlık satırı:** öğrenci adı (17/700) · şube + toplantı ("9-A · 20 Ekim 2026 18:30") · kapat (X). Öğrenci avatarı baş harf. 
2. **KALICI ETİKET (başlığa yapışık, kaydırmayla kaybolmaz, kapatılamaz):** göz-çizgili ikon + **"Veliye gösterilmez — yalnız sınıf rehber öğretmeni ve idare görür"** (00 ortak bağlamdaki etiket spesifikasyonu: lacivert %7 zemin, lacivert yazı 12/600, tam genişlik şerit).
3. **Not alanı:** çok satırlı metin alanı; web'de panelin kalan yüksekliğini doldurur, mobilde klavye üstünde kalacak şekilde ≈ 8 satır başlangıç yüksekliği, içerik arttıkça kayar. Yer tutucu: "Görüşme özeti… Listede olmayan bir yakın geldiyse buraya yazabilirsiniz." Karakter sayacı sağ altta "128 / 4.000"; 3.800'den sonra uyarı tonu; 4.000'de giriş durur ve Note "En fazla 4.000 karakter".
4. **Otomatik kayıt durumu (alan altında, canlı bölge/aria-live):** 
   - Yazarken: "Kaydediliyor…" (kısa gecikmeyle)
   - Kaydedildi: "Kaydedildi 18:42" (başarı tonu, tik)
   - Çevrimdışı: "Çevrimdışı — bağlanınca kaydedilecek" (uyarı tonu, bulut-saat ikonu); yazı cihazda tutulur
   - Hata: "Kaydedilemedi" (hata tonu) + "Tekrar dene"; metin KAYBOLMAZ
   Elle "Kaydet" düğmesi YOKTUR (otomatik kayıt); kapatınca bekleyen kayıt tamamlanır ("Kapatmadan önce kaydediliyor…" kısa durumu).
5. **Meta satırı:** "Son düzenleyen: Selma Arslan · 20 Ekim 2026 18:42" (ilk yazan farklıysa "İlk yazan: …" ikinci satır).
6. **Alt bilgi notu (küçük, sönük):** "Bu nota erişimler kayıt altına alınır." (şeffaflık; denetim kaydını yalnız idare görür — öneri, bkz. karar notu).

## Durum varyantları (ayrı çerçeveler)
- **Boş not:** alan boş, yer tutucu görünür, kayıt durumu "Henüz not yok".
- **Dolu not:** metin + "Kaydedildi".
- **Yayınlandı** (toplantı henüz başlamadı): panel açılmaz; PM-T-02'de öğrenci satırı soluk ve not ikonu devre dışı + ipucu "Notlar toplantı başladığında açılır." (Bu panelin çerçevesi yerine bu yönlendirme çizilir.)
- **Tamamlandı, düzenleme penceresi açık:** düzenlenebilir; üstte Note (bilgi) "Tamamlandı — 6 gün daha düzenleyebilirsiniz."
- **Tamamlandı, pencere dolmuş / sezon kapalı / İptal toplantı:** salt okunur — alan düzenlenemez (kilit ikonu), kayıt durumu satırı "Salt okunur", etiket YİNE görünür.
- **Saklama süresi dolmuş (metin silindi):** alan yerine nötr kart: "Bu notun metni saklama süresi dolduğu için silindi. Katılım kaydı korunur." Düzenleme yok. Etiket görünür.
- **Eski rehber öğretmen / yetki:** notu yazan öğretmen, şubenin güncel rehber öğretmeni ve idare görür; yetkisiz kullanıcıya panel hiç açılmaz — 403 durumu "Bu notu görüntüleme yetkiniz yok" (genel hata ekranı ÇİZME, panel içi yetki kartı).
- **İdare "Yürüt":** aynı panel; meta satırında "Neslihan Çetin (Müdür Yardımcısı)".

## Kenar durumları
- Bağlantı kopması sırasında metin yerelde tutulur; bağlanınca gönderilir; başka cihazdan farklı sürüm varsa son yazan kazanır, iki sürüm denetimde kalır — arayüzde çözümleme ekranı ÇİZME, yalnız Toast "Not başka bir cihazdan güncellendi".
- Panel açıkken toplantı otomatik kapanırsa (3 gün) not korunur; düzenleme penceresi başlığı güncellenir.
- Çok uzun not (4.000 karakter) kaydırma ve performans: tek alan, kendi içinde kayar.
- Hassas içerik yönlendirmesi (öneri): alanın altında ince ipucu "Sağlık, aile durumu gibi hassas bilgileri yalnız gerektiği kadar yazın." — isteğe bağlı çerçevede göster, karar notunda işaretli.
- Davetli olmayan yakın: not içine serbest metinle yazılır (özel alan YOK).

## Örnek veri
Defne Yalçın · 9-A · Salı, 20 Ekim 2026 18:30. Not metni (zararsız örnek): "Matematikte ek çalışma önerildi; haftada 2 saat etüt planlandı. Veli sınıf WhatsApp grubundan çıkmak istiyor." Karakter 119/4.000. Son düzenleyen: Selma Arslan · 20 Ekim 2026 18:42. Çevrimdışı varyantı: aynı metin + "Çevrimdışı — bağlanınca kaydedilecek". Saklama süresi dolmuş varyantı: Nehir Şahin · 9-A (ileri tarihli tasarım örneği).

## Durumlar (genel)
- Yükleniyor (not çekiliyor): alan yerine iskelet çizgileri; etiket yine görünür.
- Hata (not yüklenemedi): panel içi hata + "Tekrar dene"; etiket görünür.
- Çevrimdışı: yerel taslak varsa gösterilir.
- İzin yok: panel içi yetki kartı.

## Bu ekranda ÇİZME
Notu veliyle/öğrenciyle paylaşma düğmesi · notu bildirimle gönderme · yapay zekâ özeti/öneri/otomatik doldurma · notu dışa aktarma · not geçmişi/diff görünümü (denetim kaydı idareye ayrı ekrandır, PM-A-08) · öğrenciler arası sıralama · emoji/şablon metin ekleme.
```

## Sipariş sonrası kontrol listesi
- [ ] Kalıcı "Veliye gösterilmez — yalnız sınıf rehber öğretmeni ve idare görür" etiketi HER varyantta (salt okunur ve silinmiş dahil) görünüyor, kapatılamıyor mu?
- [ ] Elle "Kaydet" düğmesi yok; kayıt durumu (Kaydediliyor / Kaydedildi hh:mm / Çevrimdışı / Hata) var mı?
- [ ] Çevrimdışıyken yazılan metin kaybolmuyor mu?
- [ ] Metin silinmiş durumu "silindi" kartı olarak ayrı çizilmiş mi (boş alan gibi görünmüyor)?
- [ ] Paylaş / bildirim / dışa aktar / yapay zekâ düğmesi sızmış mı? (olmamalı)
- [ ] Mobilde klavye açıkken etiket ve kayıt durumu görünür mü?
- [ ] Yayınlandı durumunda panel açılmıyor mu?
- [ ] 4.000 karakter sınırı davranışı çizilmiş mi?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- "Bu nota erişimler kayıt altına alınır" bilgi satırı envanterde yok (BR-PM-31 okuma denetimini tanımlar, kullanıcıya bildirimi tanımlamaz); şeffaflık için öneri — onaylanmazsa kaldırılır.
- Hassas içerik ipucu (KVKK özel nitelikli veri) envanterde yok; öneri.
- Eski rehber öğretmenin (yazan) notu görmesi S-4 kararı kabul edildi; paneldeki "İlk yazan" meta satırı bu yüzden eklendi.
- Mobilde panel kalıbı: envanter "alttan tam yükseklik panel" der, harita Sheet size="full" ile uyumlu.
- Önceki/sonraki öğrenciye geçiş kısayolu envanterde yok; çizilmedi.
