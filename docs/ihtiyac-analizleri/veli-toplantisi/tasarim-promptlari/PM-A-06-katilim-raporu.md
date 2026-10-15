# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-A-06 · Katılım raporu (beyan × fiili)**

> Kaynak: İhtiyaç analizi §13, §10.2 PM-A-06, K-10, K-16, BR-PM-13/25/28/40, §11, §12. Platform: **Web + Mobil (eşit)**. Rol: İdare (tüm okul); sınıf rehber öğretmeni aynı ekranı **yalnız kendi şubesi** kapsamıyla görür (varyant). Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN: Katılım raporu (PM-A-06)

**Rol:** İdare (Neslihan Çetin) — varyant: Sınıf Rehber Öğretmeni (Selma Arslan, yalnız 9-A). **Amaç:** Toplantı akşamı ve sonrasında, kağıt liste olmadan: hangi şubede kaç veli gelmeyi söylemişti, kaçı gerçekten geldi; beyan ile fiili katılım tutuyor mu; kime ulaşılamadı.

## Giriş
PM-A-03'teki "Katılım raporu" düğmesi ya da Raporlar altındaki Veli Toplantısı girişi. Rota /parent-meetings/[id]/report. Rehber öğretmen: PM-T-01 kartındaki "Rapor" bağlantısı → aynı ekran, kapsam yalnız kendi şubesi (okul toplamı yok, "Şube toplantısı: 9-A" başlığı).
Toplantı henüz yapılmadıysa rapor yalnız BEYAN tarafını gösterir; "Geldi/Gelmedi" sütunları soluk "—" ve "Toplantı sonrası" notu.
Rapor öznesi ŞUBE ve TOPLANTIDIR; öğretmen bazlı kıyas, sıralama ya da "en iyi/en kötü" etiketi YOKTUR.

## Bölüm 1 · Organizasyon özeti (şube tablosu)
KpiRow (iki ayrı eksen, ayrı kartlar): BEYAN → "Yanıt oranı %62" · "Katılacağım 214" · "Katılamayacağım 34" · "Yanıt yok 152"; FİİLİ → "Fiili katılım %58" · "Geldi 231" · "Gelmedi 169". İki grup arasında görsel ayırıcı; fiili kartlar ayrı kutucuk diliyle.
Web tablo — satır = şube; sütunlar: Şube · Davetli veli · Katılacağım · Katılamayacağım · Yanıt yok · **Geldi** · **Gelmedi** · Yanıt oranı · Fiili katılım oranı. Alt toplam satırı. **İptal edilen şubeler ayrı satır grubu** ("İptal edilen şubeler (1)"), sayılar toplama DAHİL DEĞİL; satır soluk, "İptal" rozeti.
Mobil: şube başına kart (şube, iki oran çipi: "Yanıt %68 · Fiili %66"; altta altı mini sayaç iki satırda); okul toplamı üstte özet kartta.

| Şube | Davetli | Katılacağım | Katılamayacağım | Yanıt yok | Geldi | Gelmedi |
|---|---|---|---|---|---|---|
| 5-A | 34 | 20 | 3 | 11 | 22 | 12 |
| 5-B | 35 | 19 | 4 | 12 | 21 | 14 |
| 6-A | 36 | 21 | 3 | 12 | 23 | 13 |
| 6-B | 36 | 18 | 2 | 16 | 20 | 16 |
| 7-A | 37 | 22 | 3 | 12 | 24 | 13 |
| 8-A | 36 | 20 | 4 | 12 | 22 | 14 |
| 9-A | 32 | 18 | 4 | 10 | 15 | 17 |
| 9-B | 38 | 23 | 3 | 12 | 25 | 13 |
| 10-A | 40 | 24 | 3 | 13 | 26 | 14 |
| 11-A | 36 | 12 | 2 | 22 | 14 | 22 |
| 12-A | 40 | 17 | 3 | 20 | 19 | 21 |
| **Toplam** | **400** | **214** | **34** | **152** | **231** | **169** |
İptal grubu: 10-C — davetli 32; yanıt/fiili sayıları iptal anındakiyle gri.

## Bölüm 2 · Beyan × fiili çapraz tablosu
3 satır (beyan: Katılacağım / Katılamayacağım / Yanıt yok) × 2 sütun (fiili: Geldi / Gelmedi) = ALTI bağımsız hücre ızgarası; her hücre sayı + satırın yüzdesi. Satır/sütun başlıkları iki farklı rozet dilinde (beyan hap rozet, fiili kutucuk). Okul toplamı: Katılacağım→Geldi 190 · Katılacağım→Gelmedi 24 · Katılamayacağım→Geldi 6 · Katılamayacağım→Gelmedi 28 · Yanıt yok→Geldi 35 · Yanıt yok→Gelmedi 117. Hücreye dokunma/tıklama Bölüm 4'teki veli listesini o kombinasyona süzer. Yorum/öneri cümlesi YOK; yalnız sayı.

## Bölüm 3 · Kanal ulaşımı
Üç sayaç kartı: "Hesabı olmayan veli 37" · "E-posta adresi olmayan veli 41" · "İdare tarafından veli adına işaretlenen yanıt 18". (Öneri, bilgi amaçlı dördüncü kart: "Son 30 günde giriş yapmamış veli 63" — envanterde 'öneri' işaretli, ayrı çerçevede.) Hesapsız kartının altında "SMS bu sürümde gönderilmiyor" notu.

## Bölüm 4 · Veli listesi (süzgeçli)
Satır: öğrenci · şube · veli · beyan rozeti (+ "İdare girdi" çipi) · fiili kutucuk (Geldi/Gelmedi) · **"Not var" işareti** (ikon; NOT METNİ ASLA YOK).
Hazır süzgeç çipleri: "Katılacağım dedi ama gelmedi" · "Hiç yanıt vermedi" · "Veli adına işaretlenen" · "Hesapsız". Ek: şube seçici, arama. Web tablo + Pager; mobil kart listesi + "Daha fazla yükle".

## Dışa aktarım
Web: "Excel'e aktar" düğmesi (sağ üst); dosya şube özeti + çapraz tablo + veli listesi sayfalarını içerir; tıklamadan önce küçük Note: "Dışa aktarımda görüşme notu metni yoktur; yalnız 'not var/yok' bilgisi bulunur." Mobil: ⋯ menüsünde "Dışa aktar" → paylaşım paneli, aynı Note.

## ROL VARYANTI — Sınıf Rehber Öğretmeni
Yalnız 9-A; Bölüm 1 tek satır; okul toplamı ve diğer şubeler görünmez; Bölüm 3 yalnız şubenin hesapsız/e-postasız sayıları; Bölüm 4 aynı. Dışa aktarım kapsamı "yalnız 9-A" (envanterde belirtilmemiş, bkz. karar notu).

## Örnek veri
Yukarıdaki tablo (1. Dönem Veli Toplantısı, toplantı sonrası görünümü: Salı, 20 Ekim 2026'dan sonra). Önceki-toplantı varyantı (yalnız beyan): aynı tablo, Geldi/Gelmedi sütunları "—". 9-A çaprazı: Katılacağım→Geldi 14 · Katılacağım→Gelmedi 4 · Katılamayacağım→Geldi 0 · Katılamayacağım→Gelmedi 4 · Yanıt yok→Geldi 1 · Yanıt yok→Gelmedi 9.
Süzgeçli liste örneği "Katılacağım dedi ama gelmedi" (9-A): Nehir Şahin → Gülsüm Şahin · Ozan Taşkın → Selçuk Taşkın · Hira Nur Bozkurt → Zehra Bozkurt; her satırda "Not var" ikonu farklı (biri var, ikisi yok).

## Durumlar
- Yükleniyor: KPI + tablo iskeleti; çapraz tablo hücreleri iskelet.
- Boş: raporlanacak veri yok (hiçbir şube yayınlanmamış ya da tüm şubeler iptal) → EmptyState "Raporlanacak veri yok" + nedeni ("Tüm şubeler iptal edildi" / "Toplantı henüz yayınlanmadı").
- Süzgeç sonucu boş: "Bu süzgece uyan veli yok" + "Süzgeçleri temizle".
- Hata: "Tekrar dene". Excel hazırlanamadı: Toast hata.
- İzin yok: 403; rehber öğretmen başka şubenin raporunu açarsa 404 "Bu rapor bulunamadı".
- Sezon kapalı: rapor açılır (salt okunur); ek bant çizme.
- Toplantı Başladı ama Tamamlanmadı: fiili sütunlar "devam ediyor" etiketiyle; "Gelmedi" HESAPLANMAZ ("İşaretlenmedi" gösterilir) — Tamamlandı olunca Gelmedi'ye döner.

## Bu ekranda ÇİZME
Görüşme notu metni veya özeti · öğretmen bazlı kıyas/sıralama/performans · trend grafiği · yapay zekâ içgörüsü · dışa aktarımda not · SMS ulaşım istatistiği (SMS gönderilmez).
```

## Sipariş sonrası kontrol listesi
- [ ] Beyan ve fiili iki AYRI eksen/kart grubu; "Gelmedi" yalnız Tamamlandı sonrası hesaplanıyor mu?
- [ ] Çapraz tabloda altı hücre bağımsız, sayılar toplamla tutarlı mı?
- [ ] İptal şubeler ayrı grup, toplama dahil değil mi?
- [ ] "Not var" işareti var, not metni YOK mu? Excel Note'u var mı?
- [ ] Öğretmen kıyası/sıralaması sızmış mı?
- [ ] Rehber öğretmen varyantı yalnız kendi şubesi mi?
- [ ] Hesapsız/e-postasız/veli adına işaretlenen sayıları var; SMS "gönderildi" izlenimi yok mu?
- [ ] Mobilde zeminde yüzen metin yok, tablo kart listesine dönmüş mü?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **Rehber öğretmen için rapor ekranı:** envanterde PM-A-06 yalnız İdare; §13 "rehber öğretmen: kendi şubesi" der. Prompt aynı ekranın kapsamlı varyantını çizer (giriş PM-T-01). Teyit; rehberin Excel dışa aktarımı da belirsiz.
- **"Dört kutu" ifadesi (§13):** çapraz tablo "dört kutu bağımsız" der ama sayılan kombinasyonlar altıdır; prompt 3×2 ızgara çizer. Teyit.
- Mobilde Excel dışa aktarım analizde belirtilmemiş; web ile eşitlik için eklendi.
- "Son 30 günde giriş yapmamış veli" analizde "öneri, bilgi amaçlı" (§11); kart isteğe bağlı.
- Raporlar menüsünde ayrı bir giriş olup olmayacağı belirsiz.
