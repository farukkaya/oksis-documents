# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-08 · Görüşme notu denetim kaydı**

> Kaynak: İhtiyaç analizi §10.2 PM-A-08, K-11, K-12, BR-PM-28/31/33, §5, §12 (KVKK). Platform: **Web + Mobil (eşit)**. Rol: **yalnız İdare** (rehber öğretmen denetim kaydını göremez). Önce `00-ortak-baglam.md`, sonra bu blok. Emsal: duyuru "Denetim İzi" ekranı (`audit-drawer` web / `audit-screen` mobil) — kalıbı ve zaman çizgisi dilini ondan al.

```
# EKRAN: Görüşme notu denetim kaydı (PM-A-08)

**Rol:** İdare (Neslihan Çetin; ayrıca okul müdürü Rıza Tuncer). **Amaç:** Veli toplantısı görüşme notlarına KİMİN, NE ZAMAN, hangi öğrencinin notu üzerinde, ne işlem yaptığını (okudu / oluşturdu / değiştirdi / sistem sildi) görmek. Bu liste bir KVKK hesap verebilirlik aracıdır: **metin içermez** — not içeriği hiçbir koşulda burada görünmez.

## Giriş
Web: Veli Toplantıları altında "Not denetim kaydı" sekmesi/bağlantısı (rota /parent-meetings/note-audit); ayrıca PM-A-04'te öğrenci satırından "Bu notun denetim kaydı" kısayolu (aynı ekranı öğrenci süzgeci dolu açar). Mobil: Veli Toplantıları ⋯ menüsü → "Not denetim kaydı" (ALT ekran, STopBar). Rehber öğretmen ve diğer roller için bu giriş HİÇ görünmez.

## Üst açıklama kartı (kalıcı, kapatılamaz)
Kilit/göz ikonlu Note (bilgi tonu): "Bu kayıt görüşme notlarına erişimi ve değişiklikleri gösterir. Not metni burada yer almaz. Saklama süresi dolan notların metni silinse de bu kayıt kalır."

## Süzgeçler (web: üst çubuk; mobil: filtre düğmesi → alt panel)
- **Öğrenci** (arama + seçim; örn. "Defne Yalçın")
- **Toplantı / şube** (organizasyon + şube: "1. Dönem Veli Toplantısı · 9-A")
- **Kullanıcı** (notu işleyen kişi: "Selma Arslan")
- **İşlem** (Okudu · Oluşturdu · Değiştirdi · Sistem sildi)
- **Tarih aralığı**
Etkin süzgeçler çip olarak görünür, "Süzgeçleri temizle".

## Liste
Web tablo, en yeni üstte. Sütunlar: **Tarih ve saat** (mono) · **Kullanıcı** (ad + rol: "Selma Arslan · Sınıf Rehber Öğretmeni" / "Neslihan Çetin · Müdür Yardımcısı" / "Sistem") · **İşlem** (rozet: Okudu = nötr göz ikonu; Oluşturdu = başarı tint artı ikonu; Değiştirdi = bilgi tint kalem ikonu + "Sürüm 3" çipi; Sistem sildi = nötr koyu çöp ikonu) · **Öğrenci** (ad + şube) · **Toplantı** (ör. "1. Dönem · 9-A · 20 Eki 2026"). Satır sonunda ekstra sütun YOK, ayrıntı çekmecesi açılır.
Satıra tıklama → ayrıntı (web: sağ çekmece; mobil: Sheet): işlem tipi, tam zaman damgası, kullanıcı, öğrenci, toplantı, **sürüm numarası** (Oluşturdu: Sürüm 1; Değiştirdi: Sürüm n) ve "İçerik denetim kaydında tutulmaz" ibaresi. Değişiklik kayıtlarında "sürüm kaydı" bilgisi olarak yalnız sürüm no ve zaman gösterilir — İÇERİK / FARK / ÖNCEKİ METİN YOK.
"Sistem sildi" satırı: açıklama "Saklama süresi doldu; not metni kalıcı olarak silindi" + "Not kaydı mevcut, metin yok" göstergesi.
Mobil: kart listesi — üst satır işlem rozeti + saat; ikinci satır "Selma Arslan okudu"; üçüncü satır "Defne Yalçın · 9-A"; dokununca Sheet.
Pager (web) / "Daha fazla yükle" (mobil). Sayfa başına 25 kayıt.

## Örnek veri
| Zaman | Kullanıcı | İşlem | Öğrenci | Toplantı |
|---|---|---|---|---|
| 20 Eki 2026 19:42 | Selma Arslan · Sınıf Rehber Öğretmeni | Değiştirdi · Sürüm 2 | Defne Yalçın · 9-A | 1. Dönem · 9-A |
| 20 Eki 2026 19:38 | Selma Arslan · Sınıf Rehber Öğretmeni | Oluşturdu · Sürüm 1 | Defne Yalçın · 9-A | 1. Dönem · 9-A |
| 20 Eki 2026 19:31 | Selma Arslan · Sınıf Rehber Öğretmeni | Okudu | Ahmet Kerem Polat · 9-A | 1. Dönem · 9-A |
| 21 Eki 2026 09:15 | Neslihan Çetin · Müdür Yardımcısı | Okudu | Defne Yalçın · 9-A | 1. Dönem · 9-A |
| 21 Eki 2026 09:16 | Neslihan Çetin · Müdür Yardımcısı | Okudu | Ahmet Kerem Polat · 9-A | 1. Dönem · 9-A |
Sistem varyantı (ileri tarihli tasarım örneği): "12 Oca 2029 02:00 · Sistem · Sistem sildi · Nehir Şahin · 9-A · 1. Dönem".
Yoğun okuma örneği: aynı kullanıcı aynı öğrenciyi 1 dakikada 3 kez açmış → üç ayrı satır (gruplama/özetleme YOK, her okuma ayrı kayıt).

## Durumlar
- Yükleniyor: tablo iskeleti (10 satır).
- Boş: hiç kayıt yok → EmptyState "Henüz görüşme notu işlemi yok" ("Notlar yazıldığında erişim kayıtları burada listelenir.").
- Süzgeç sonucu boş: "Bu süzgece uyan kayıt yok" + "Süzgeçleri temizle".
- Hata: "Tekrar dene".
- İzin yok: 403 "Not denetim kaydını görüntüleme yetkiniz yok" (rehber öğretmen ve diğerleri).
- Sezon kapalı/arşiv: kayıt görünür kalır (salt okunur zaten).
- Çok büyük sonuç: Pager + "Sonuçlar 25'erli sayfalanır" bilgisi.

## Bu ekranda ÇİZME
Not metni, önceki/sonraki metin farkı, metin özeti · kaydı düzenleme/silme · dışa aktarma (envanterde yok) · kullanıcı bazlı "en çok okuyan" sıralaması/kıyası · uyarı/alarm kuralları · veli ya da öğrenci görünümü.
```

## Sipariş sonrası kontrol listesi
- [ ] Hiçbir yerde not metni, özeti veya diff'i var mı? (olmamalı)
- [ ] Dört işlem tipi (Okudu / Oluşturdu / Değiştirdi / Sistem sildi) ayrı rozetle ve ikon+metinle mi?
- [ ] "Okudu" kayıtları da listede mi, gruplanmamış mı?
- [ ] Süzgeçler: öğrenci, toplantı, kullanıcı, işlem, tarih var mı?
- [ ] Düzenle/sil/dışa aktar düğmesi sızmış mı? (olmamalı)
- [ ] Giriş yalnız idareye mi (rehber öğretmen menüsünde yok)?
- [ ] Mobilde kart listesi, zeminde yüzen metin yok mu?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- BR-PM-31 "değişiklikte sürüm kaydı" der; envanter "metin içermez" der. Prompt yalnız sürüm no + zaman gösterir (metin/fark yok). Teyit: idare sürüm içeriğine erişebilir mi? (varsayılan: hayır).
- Giriş noktası belirsiz (Veli Toplantıları içinde sekme mi, Ayarlar/Denetim altında mı?).
- Dışa aktarım yok varsayıldı.
- Okuma kayıtlarının hacmi (her panel açılışı bir kayıt) için gruplama/saklama kuralı analizde yok; prompt gruplamaz.
- "Sistem sildi" satırı yalnızca saklama süresi dolunca görünür; modül yeni olduğundan prompt ileri tarihli örnek kullanır.
