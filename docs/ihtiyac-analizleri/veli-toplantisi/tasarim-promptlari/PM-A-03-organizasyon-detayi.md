# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-03 · Organizasyon detayı — şube durum tablosu (İdare)**

> Kaynak: İhtiyaç analizi §6.1 (7, 9), §7, §10.2 PM-A-03, BR-PM-05/06/13/17, K-2, K-9. Platform: **Web + Mobil (eşit)**. Rol: İdare. Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN: Organizasyon detayı — şube durum tablosu (PM-A-03)

**Rol:** İdare. **Amaç:** Bir veli toplantısı organizasyonunun HER ŞUBESİNİ tek tabloda yönetmek: hangi şubede kaç veli geleceğini söylemiş, hangi şube iptal, hangisinin tarihi değişti; tek şubeyi ya da seçili şubeleri iptal etmek / tarihini değiştirmek; yayından sonra şube eklemek.

## Giriş
PM-A-01'de organizasyon kartına tıklanınca açılır. Rota /parent-meetings/[id]. Taslak organizasyon bu ekranı açmaz (doğrudan PM-A-02 düzenleme moduna gider).

## Üst bölüm (organizasyon kimliği)
Başlık "1. Dönem Veli Toplantısı" · organizasyon durumu rozeti (Yayınlandı) · açıklama metni (2 satır, "devamını göster") · özet sayaç çipleri "11 yayında · 1 iptal" · sağda birincil düğme "Şube ekle" (PM-A-02 şube-ekle modu), ikincil "Katılım raporu" (PM-A-06). Sezon kapalıysa "Şube ekle" ve tüm değiştirici eylemler devre dışı + Note "Bu sezon kapalı; kayıtlar salt okunur."
Üstte KpiRow: "Davetli veli 400" · "Katılacağım 214" · "Katılamayacağım 34" · "Yanıt yok 152" (iptal şubeler hariç; toplantılar Başladı/Tamamlandı oldukça 5. kart "Geldi" eklenir). Beyan ve fiili AYRI kartlar.

## ŞUBE DURUM TABLOSU (ekranın ana yapısı)
Sütunlar (web): [seçim kutusu] · Şube · Sınıf Rehber Öğretmeni · Tarih ve saat · Yer · Durum (rozet) · Katılacağım · Katılamayacağım · Yanıt yok · Geldi (şube Başladı/Tamamlandı olana dek "—") · satır eylemleri (⋯).
- Tarih ve saat hücresi: "Sal, 20 Eki · 17:00–18:00"; plan sürümü 2 ve üstündeyse yanında küçük "Sürüm 2" çipi ve ipucu "Tarih/saat değiştirildi; yanıtlar sıfırlandı".
- Rehber hücresi: rehbersiz şubede (11-A) uyarı rozeti "Rehber öğretmen yok — idare yürütür".
- İptal şube (10-C): satır soluk, Durum = İptal rozeti, sayaç sütunları iptal anındaki değerlerle gri. Durum hücresinde ipucu/ikon: "İptal gerekçesi: Rehber öğretmen raporlu" — bu gerekçe yalnız idareye görünür; tabloda "yalnız idare görür" küçük etiketiyle belirtilir (gerekçe veliye gitmez).
- Başladı satırında "canlı" nokta; Tamamlandı satırında düzenleme penceresi bilgisi "5 gün daha düzenlenebilir".
- Sütun başlığına tıklayınca sıralama (şube, tarih, yanıt yok). Sayaçlarda mikro grafik YOK (sade sayı).
- Satır eylemleri (⋯ ve hover kısa düğmeler): **Aç** (PM-A-04) · **Tarih değiştir** (PM-A-05) · **İptal** (PM-A-05). Durum Tamamlandı veya İptal ise Tarih değiştir/İptal GÖSTERİLMEZ (terminal), yalnız "Aç".
- **Çoklu seçim:** satırlar seçilince üstte yapışkan toplu eylem çubuğu: "3 şube seçildi · Tarih değiştir · İptal · Seçimi temizle". Toplu işlem = seçili satırlara AYNI işlem (PM-A-05 toplu mod, şube kırılımlı önizleme). Terminal durumdaki satırlar seçilemez (kutu devre dışı, ipucu "Tamamlanmış ya da iptal edilmiş şube değiştirilemez").
- Üst süzgeç: durum çipleri (Tümü · Yayınlandı · Başladı · Tamamlandı · İptal) + kademe (Ortaokul · Lise). Arama: şube veya öğretmen adı.

## MOBİL yerleşimi (402×874)
STopBar "Veli Toplantıları" (geri) + ⋯ (Şube ekle · Katılım raporu). Organizasyon kimlik kartı (başlık, durum rozeti, sayaç çipleri), altında 2×2 özet kartı (Davetli · Katılacağım · Katılamayacağım · Yanıt yok). Şube tablosu **kart listesi**: her kart — üst satırda şube + durum rozeti; ikinci satırda "Sal, 20 Eki · 17:00–18:00 · 6-B Sınıfı"; üçüncü satırda rehber adı; dördüncü satırda üç mini sayaç (Katılacağım / Katılamayacağım / Yanıt yok); sağda ">". Karta dokununca PM-A-04. Kart ⋯ → alt panel: Aç · Tarih değiştir · İptal. Çoklu seçim: "Seç" modu (üstte "Seç" düğmesi) → kartlarda onay kutusu, alt sabit çubuk "3 seçili · Tarih değiştir · İptal". Durum süzgeci yatay çip şeridi.

## Örnek veri (12 şube)
5-A Aylin Demirtaş 17:00–18:00 · 5-B Tolga Kılıç · 6-A Meral Özkan · 6-B Hasan Yıldırım (Sürüm 2, 17:00 → 18:00 taşındı) · 7-A Pınar Sevim · 8-A Okan Bulut · 9-A Selma Arslan 18:30–19:30 (Katılacağım 18 · Katılamayacağım 4 · Yanıt yok 10) · 9-B Burak Tekin · 10-A Gülay Erdem · 10-C Cemil Aksoy (İPTAL) · 11-A rehber yok · 12-A Serpil Koç (Konferans Salonu). Sayılar tutarlı: toplam davetli 400, iptal şube dahil değil (214 + 34 + 152 = 400).
Toplantı sonrası varyant: aynı tablo Tamamlandı (9-A: Katılacağım 18 → Geldi 15) — "Geldi" sütunu dolu ve ayrı rozet dilinde.

## Durumlar
- Yükleniyor: kimlik kartı + tablo iskeleti (8 satır).
- Süzgeç sonucu boş: "Bu süzgece uyan şube yok" + "Süzgeçleri temizle". (Organizasyonun hiç şubesi olamaz — BR-PM-02; bu durum çizilmez.)
- Hata: kart içi hata + "Tekrar dene".
- İzin yok: 403 "Bu toplantıyı görüntüleme yetkiniz yok". Bulunamadı/silinmiş: 404 "Bu toplantı bulunamadı".
- Sezon kapalı: salt okunur (yukarıda).
- Toplu işlem kısmen başarısız: Toast + dialog "3 şubeden 2'si güncellendi, 1'i güncellenemedi (kayıt değişti)"; başarısız satır işaretli.
- Eşzamanlılık: başka idareci bir şubeyi değiştirdi → satır üstünde Note "Bu şube değişti, yenileyin".
- Tüm şubeler iptal: üstte hata tint bant "Tüm şubeler iptal edildi".

## Bu ekranda ÇİZME
Organizasyonu "tümüyle sil" (yayında silinmez; toplu iptal ile karşılanır) · şubeyi organizasyondan çıkar (yayınlanmış şube çıkarılamaz, iptal edilir) · öğretmen atama/değiştirme · öğretmen bazlı kıyas.
```

## Sipariş sonrası kontrol listesi
- [ ] Beyan sayaçları ile "Geldi" sütunu görsel olarak ayrışıyor mu? "Geldi" toplantı öncesinde "—" mi?
- [ ] Tamamlandı/İptal satırlarında Tarih değiştir/İptal gizli, çoklu seçim devre dışı mı?
- [ ] İptal gerekçesi yalnız idareye görünür etiketli mi?
- [ ] Rehbersiz şube uyarısı var mı? "Atama" eylemi sızmış mı?
- [ ] Toplu eylem çubuğu seçili satır sayısını gösteriyor; mobilde alt çubuk sekme çubuğunu örtmüyor mu?
- [ ] "Şube ekle" yayında görünür; sezon kapalıyken devre dışı mı?
- [ ] Mobilde şube kartları 44pt hedefli, sayaçlar ikon+metinle mi?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- Organizasyon düzeyinde "tümünü iptal" ayrı eylem değil; "tüm satırları seç + toplu İptal" olarak ele alındı.
- İptal gerekçesinin tabloda ipucu olarak gösterilmesi (analiz: gerekçeyi yalnız idare görür; gösterim yeri belirtilmemiş).
- Tamamlandı/İptal şubelerin sayaçlarının tabloda gösterimi (iptal anı değerleri) analizde net değil.
- Düzenleme penceresi geri sayımının tabloda gösterilmesi (yalnız PM-T-02'de tanımlı) — burada öneri.
