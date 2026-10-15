# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-01 · Veli Toplantıları listesi (İdare)**

> Kaynak: İhtiyaç analizi §10.2 PM-A-01, §7, §8 (BR-PM-05/06/08/42), §5. Platform: **Web + Mobil (eşit)**. Rol: İdare (okul yöneticisi + müdür yardımcısı). Önce `00-ortak-baglam.md` bloğunu yapıştır, ardından aşağıdaki bloğu.

```
# EKRAN: Veli Toplantıları listesi (PM-A-01)

**Rol:** İdare (Neslihan Çetin, Müdür Yardımcısı). **Amaç:** Sezonun veli toplantılarını tek bakışta görmek: hangi organizasyon yayında, kaç şubesi iptal/tamamlandı, yanıt oranı nasıl; yeni toplantı başlatmak; taslağa dönmek.

## Giriş noktaları
- Web: sol menüde "Veli Toplantıları" (konumu Faruk'la netleşecek — menüde seçili göster). Rota /parent-meetings.
- Mobil: "Daha fazla" menüsünde "Veli Toplantıları" satırı → bu ekran ALT ekran (STopBar "Veli Toplantıları", sağda "+" yeni toplantı).

## Veri alanları (organizasyon başına)
başlık · tarih aralığı (ilk–son şube toplantısı tarihi; tek günse tek tarih) · şube sayısı · durum (Taslak/Yayınlandı) · şube durum özeti sayaçları ("11 yayında · 1 iptal") · yanıt oranı (güncel sürümde yanıt veren davetli veli / toplam davetli veli; iptal şubeler hariç; yüzde + ince ilerleme çubuğu) · son güncelleme. Sezon = üst bardaki seçili sezon. Kapalı (arşivlenmiş) sezonda liste salt okunur, "Yeni toplantı" devre dışı + Note: "Bu sezon kapalı; yeni toplantı oluşturulamaz."

## WEB yerleşimi (1440)
PageHeader: "Veli Toplantıları" · alt başlık "2026–2027 · 1. Dönem" · sağda birincil düğme "Yeni toplantı".
Üstte KpiRow (3 kart, listeden türetilen sayılar): "Yayında olan toplantı" · "Yanıt bekleyen veli" · "Bu hafta toplantısı olan şube".
Sekmeler: **Yayında ve geçmiş** (varsayılan) · **Taslaklar (n)**.
Süzgeç çubuğu: durum (Tümü · Yayınlandı · Başladı · Tamamlandı · İptal içeren) · tarih aralığı. Taslaklar sekmesinde süzgeç yok.
Liste: organizasyon KARTLARI (tablo değil; her kart tek satır genişliğinde): sol blokta başlık + tarih aralığı + "12 şube"; orta blokta şube durum sayaç çipleri (durum rozet dilinde); sağ blokta yanıt oranı çubuğu "%62 · 248/400 veli" ve "Aç" bağlantısı. Kart tıklanınca PM-A-03. Kart menüsü (⋯): Aç · (Taslaksa) Düzenle · (Taslaksa) Sil. Yayınlanmış organizasyonda SİL YOKTUR.
Taslak kartı: Taslak rozeti, "Son kaydedilen: 14 Ekim 14:20", "8 şube seçili", yanıt oranı YOK; eylemler "Düzenle" (PM-A-02'yi açar) ve "Sil" (ConfirmDialog tone=danger: "Taslak silinsin mi? Bu işlem geri alınamaz. Hiçbir veliye bildirim gitmedi.").
Sayfalama: Pager (10/sayfa).

## MOBİL yerleşimi (402×874)
STopBar + sağda "+". Altında yatay çip şeridi: Yayında ve geçmiş · Taslaklar (n). Süzgeç tek düğme (filtre ikonu) → alt panel: durum + tarih aralığı. KPI satırı mobilde YOK; yerine ilk kartın üstünde tek satır özet kartı "1 yayında · 152 veli yanıt bekliyor". Organizasyon kartı: başlık, tarih, "12 şube", sayaç çipleri (iki satıra sarar), yanıt çubuğu. Karta dokununca PM-A-03. Kart ⋯ → alt panel (Aç/Düzenle/Sil — Sil yalnız taslakta). Sticky metinli hap Fab "Yeni toplantı".

## Örnek veri
1. "1. Dönem Veli Toplantısı" · Salı, 20 Ekim 2026 · 12 şube · "11 yayında · 1 iptal" (iptal: 10-C; 15 Ekim itibarıyla hiçbir şube henüz başlamadı). Yanıt oranı %62 (248/400 davetli veli).
2. "Sezon Başı Tanışma Toplantısı" · Salı, 22 Eylül 2026 · 4 şube (9-A, 9-B, 10-A, 12-A) · "4 tamamlandı" · yanıt oranı %81 (Geçmiş).
3. Taslak: "2. Dönem Başı Veli Toplantısı" · 8 şube · son kaydedilen 14 Ekim 14:20.

## Durumlar
- Yükleniyor: kart iskeletleri (3 adet), KpiRow iskeleti.
- Boş (sezonda hiç toplantı yok): EmptyState "Bu sezon için veli toplantısı planlanmadı" + "Yeni toplantı" düğmesi. Taslak sekmesi boş: "Kayıtlı taslak yok".
- Süzgeç sonucu boş: EmptyState filtered "Bu süzgece uyan toplantı yok" + "Süzgeçleri temizle".
- Hata: kart içinde hata + "Tekrar dene".
- İzin yok: "Bu sayfayı görüntüleme yetkiniz yok".
- Sezon kapalı: salt okunur; Note "Sezon kapatıldığında yayındaki toplantılar iptal edilir, başlamış olanlar tamamlanır." (BR-PM-42).

## Kenar durumları (ayrı çerçeve)
- Tüm şubeleri iptal edilmiş organizasyon: kartta "Tüm şubeler iptal" bandı (hata tint), yanıt çubuğu gizli.
- Çok uzun başlık (60+ karakter): iki satıra kırpılır, tam başlık tooltip / uzun basış.
- 40+ organizasyon: Pager; mobilde "Daha fazla yükle".
- Eşzamanlılık: başka idareci taslağı sildi → karta girince 404 durumu "Bu taslak silinmiş."

## Bu ekranda ÇİZME
Organizasyon "iptal et" düğmesi (iptal şube düzeyindedir; PM-A-03'te toplu yapılır) · yanıt oranı grafiği/trend · öğretmen bazlı kıyas.
```

## Sipariş sonrası kontrol listesi
- [ ] Yayınlanmış organizasyon kartında "Sil" var mı? (olmamalı; yalnız taslakta)
- [ ] Yanıt oranı ile fiili katılım karıştırılmış mı? (listede yalnız beyan oranı)
- [ ] Taslak kartında yanıt oranı/şube durum sayaçları uydurulmuş mu? (olmamalı)
- [ ] KpiRow sayıları listeden türetilebilir mi; analizde olmayan metrik var mı?
- [ ] Mobilde zeminde yüzen metin / sekme kökü gibi ikinci büyük başlık var mı?
- [ ] Sezon kapalı varyantında "Yeni toplantı" devre dışı mı?
- [ ] `00-ortak-baglam.md` sonundaki ortak kontrol listesi.

## Envanterde belirsiz / karar
- Menü konumu ve mobil giriş ("Daha fazla" içinde hangi grup).
- KpiRow envanterde tanımlı değil; prompt türetilmiş 3 sayı önerir — reddedilirse kaldırılır.
- Yanıt oranı paydası: iptal şubeler hariç varsayıldı (teyit).
