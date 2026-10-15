# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-T-01 · Toplantılarım (Sınıf Rehber Öğretmeni)**

> Kaynak: İhtiyaç analizi §6.2, §10.2 PM-T-01, §7, BR-PM-07/10, N-02, K-3. Platform: **Web + Mobil (eşit)**. Rol: Sınıf Rehber Öğretmeni (öğretmen portalı, teal `#0E7A5A`). Branş öğretmeni bu ekranı görmez. Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN: Toplantılarım — öğretmen (PM-T-01)

**Rol:** Sınıf Rehber Öğretmeni (Selma Arslan, 9-A). **Amaç:** Kendi şubesinin veli toplantılarını görmek: yaklaşan toplantı ne zaman/nerede, kaç veli geleceğini söyledi, hangisi tamamlanmayı bekliyor; toplantı günü tek dokunuşla katılım takibine geçmek.

## Giriş
Öğretmen portalı: mobil "Daha fazla" menüsünde "Veli Toplantıları"; öğretmen anasayfa kartı/Bekleyen İşler satırı (PM-T-04); N-02 bildirimi ("9-A veli toplantınız planlandı") bu ekrana iner. Web: sol menüde "Veli Toplantıları". Rota /parent-meetings (öğretmen rolünde bu liste açılır; idare rolünde PM-A-01).
Mobil: ALT ekran, STopBar "Toplantılarım".

## Veri alanları (şube toplantısı kartı)
şube ("9-A") · organizasyon başlığı · tarih-saat ("Salı, 20 Ekim 2026 · 18:30–19:30") · yer ("9-A Sınıfı") · **durum rozeti** (Yayınlandı / Başladı / Tamamlandı / İptal; Taslak öğretmene HİÇ görünmez) · beyan sayaçları **Katılacak 18 · Katılamayacak 4 · Yanıt yok 10** (Yanıt yok vurgulu) · Başladı/Tamamlandı'da fiili sayaç "Geldi 15" (ayrı kutucuk dili) · Tamamlandı'da düzenleme penceresi bilgisi "6 gün daha düzenlenebilir" · Sürüm çipi (2+) · İptal kartında "Bu toplantı iptal edildi" bandı (gerekçe GÖSTERİLMEZ — gerekçeyi yalnız idare görür).

## Yerleşim
Bölümler: **Yaklaşan** (Yayınlandı + Başladı, tarihe göre artan) · **Geçmiş** (Tamamlandı + İptal, en yeni üstte; daraltılabilir).
- Web: PageHeader "Toplantılarım" alt başlık "9-A · 2026–2027 · 1. Dönem". Kartlar tek sütun (maks. 880 px). Birincil düğme duruma göre — Yayınlandı: "Yanıtları gör"; Başladı: "Katılım takibine git" (vurgulu); Tamamlandı: "Katılımı gör" (pencere açıksa "Düzenle"); İptal: "Ayrıntı". İkincil bağlantı: "Rapor" (PM-A-06 rehber varyantı).
- Mobil: alt ekran; STopBar; kartlar tam genişlik; "Yaklaşan toplantılar" başlığı bir yüzey şeridi içinde (zeminde yüzen başlık yok). Başladı kartı üstte "canlı" nokta ve tam genişlik "Katılım takibine git" düğmesi.
- Karta dokunma → PM-T-02 (Yayınlandı: salt okunur yanıt listesi; Başladı: işaretleme açık; Tamamlandı: düzenleme penceresi; İptal: salt okunur).

## Rehber öğretmen kapsamı (BR-PM-07)
Sorumlu, işlem anındaki GÜNCEL rehber öğretmendir. Yayından sonra rehber öğretmen değiştiyse yeni rehber öğretmen kartı görür; eski rehber öğretmenin listesinden kalkar (ek çerçeve gerekmez).

## Örnek veri
- Yaklaşan: "1. Dönem Veli Toplantısı · 9-A" — Yayınlandı — Salı, 20 Ekim 2026 · 18:30–19:30 · 9-A Sınıfı — Katılacak 18 · Katılamayacak 4 · Yanıt yok 10.
- Başladı varyantı (toplantı günü 18:35): aynı kart, rozet Başladı, "Geldi 15", "Katılım takibine git".
- Geçmiş: "Sezon Başı Tanışma Toplantısı · 9-A · Salı, 22 Eylül 2026 · Tamamlandı · Katılacak 20 → Geldi 17"; düzenleme penceresi dolmuş, "Katılım ve notlar salt okunur".
- Tamamlandı + pencere açık varyantı: "Tamamlandı · 6 gün daha düzenlenebilir".
- İptal örneği (başka rehber öğretmenin kartı gibi, ayrı çerçeve): "10-C · Salı, 20 Ekim 2026 18:30 · İptal" (rehber: Cemil Aksoy).

## Durumlar
- Yükleniyor: kart iskeletleri (2 adet).
- Boş: toplantı yok → EmptyState "Henüz veli toplantınız yok" + "İdare toplantı planladığında burada göreceksiniz." (Rehber öğretmeni olmayan öğretmende menü öğesi görünmez.) Geçmiş boş: "Geçmiş toplantı yok".
- Hata: kart içi hata + "Tekrar dene". İzin yok: 403.
- Sezon kapalı: kartlar salt okunur, birincil düğme "Ayrıntı"; Note "Bu sezon kapalı."
- Çevrimdışı: son yüklenen liste, üstte "Bağlantı yok — son güncelleme 18:12" Note'u; "Katılım takibine git" çevrimdışı da açılır.

## Bu ekranda ÇİZME
Toplantı planlama/tarih değiştirme/iptal eylemleri (öğretmen yetkisi yok) · veli adına yanıt girme · diğer şubelerin toplantıları · branş öğretmeni görünümü · öğretmenler arası kıyas.
```

## Sipariş sonrası kontrol listesi
- [ ] Taslak toplantı öğretmene görünmüyor mu?
- [ ] Planlama/iptal/veli adına yanıt eylemi sızmamış mı?
- [ ] İptal gerekçesi gösterilmiyor mu?
- [ ] Beyan sayaçları ile "Geldi" kutucuğu ayrı dilde mi? Başladı öncesinde "Geldi" yok mu?
- [ ] Başladı kartının birincil düğmesi "Katılım takibine git" mi?
- [ ] Düzenleme penceresi geri sayımı Tamamlandı kartında mı?
- [ ] Mobilde zeminde yüzen bölüm başlığı yok mu?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- Öğretmen için giriş noktası (Daha fazla mı, anasayfa kısayolu mu?) ve web menü konumu belirsiz.
- Envanter "Katılacak/Yanıt yok sayaçları" der; Katılamayacak sayacı PM-T-02 ile tutarlılık için eklendi.
- Rehber öğretmenin birden fazla şubesi olması (envanter tek şube varsayar gibi) — şube süzgeci gerekir mi?
- Rehber öğretmenin "Rapor" bağlantısı (PM-A-06 varyantı) envanterde ekran olarak yok.
