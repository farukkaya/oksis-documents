# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-T-02 · Katılım takibi (Sınıf Rehber Öğretmeni; idare "Yürüt")**

> Kaynak: İhtiyaç analizi §6.2, §6.4, §10.2 PM-T-02, K-6, K-10, BR-PM-10/11/12/24/25/26/30, §11. Platform: **Web + Mobil (eşit)** — mobil ekranın ASIL kullanım yeridir (öğretmen toplantı sırasında telefonla işaretler). Roller: Sınıf Rehber Öğretmeni ve (Yürüt sekmesinde) idare; aynı bileşen. Önce `00-ortak-baglam.md`, sonra bu blok.

```
# EKRAN: Katılım takibi (PM-T-02)

**Rol:** Sınıf Rehber Öğretmeni (Selma Arslan, 9-A). İdare "Yürüt" sekmesinden (PM-A-04) AYNI ekranı kullanır. **Amaç:** Toplantı sürerken, kalabalık bir koridorda tek elle: gelen her veliyi tek dokunuşla "Geldi" işaretlemek; her öğrenci için görüşme notuna geçmek; bitince "Toplantıyı tamamla". Beyan (veli ne dedi) ile fiili (gerçekte geldi mi) iki ayrı şeydir ve ekranda ayrı görünür — fiili ASLA beyandan ön doldurulmaz.

## Üst bölüm (yapışkan)
- Kimlik şeridi: "9-A · Salı, 20 Ekim 2026 · 18:30–19:30 · 9-A Sınıfı" + durum rozeti (Yayınlandı / Başladı / Tamamlandı / İptal).
- **Sayaç şeridi** (tek satır, mobilde yatay kaydırılabilir ya da 2×2): "Katılacak 18 · Katılamayacak 4 · Yanıt yok 10 · **Geldi 15**". İlk üçü beyan (hap rozet dili, sönük), **Geldi** vurgulu, ayrı kutucuk dilinde ve büyük sayı (örn. "Geldi 15 / 32"). Geldi dokunuşla anında güncellenir (iyimser güncelleme). Sayaçlar süzgeç görevi görür (dokununca liste o gruba süzülür; tekrar dokununca kalkar).
- Durum notları: Yayınlandı → Note "Katılım işaretleme toplantı başladığında açılır. Şimdilik veli yanıtlarını görebilirsiniz." · Başladı → "Toplantı başladı." (canlı nokta) · Tamamlandı → "Tamamlandı. **6 gün daha** düzenleyebilirsiniz." (süre dolunca "Düzenleme süresi doldu; kayıtlar salt okunur.") · İptal → hata tint "Bu toplantı iptal edildi."
- Arama kutusu (öğrenci/veli adı) + süzgeç çipleri: Tümü · Katılacak · Katılamayacak · Yanıt yok · Geldi · İşaretlenmedi.

## Liste (öğrenci başına, altında veli satırları)
**Öğrenci satırı:** ad soyad (15 semibold) · okul no (küçük, mono-olmayan) · sağda **not durumu ikonu**: kalem-boş (not yok) / dolu not ikonu (not var). Satıra (öğrenci adı alanına) dokunma → **görüşme notu paneli (PM-T-03)**.
**Veli satırı (öğrenci satırının altında girintili):** veli adı + yakınlık (anne/baba/vasi) · **beyan rozeti** (Katılacağım / Katılamayacağım / Yanıt yok; salt okunur, "İdare girdi" çipi varsa gösterilir) · Katılamayacağım'da küçük neden ikonu (dokununca/hover'da neden metni açılır; bu metin yalnız rehber ve idareye görünür) · **"Geldi" düğmesi** (aşağıda).
Bir öğrencinin iki velisi varsa iki veli satırı (Defne Yalçın → Nermin Yalçın: Katılacağım; Kadir Yalçın: Yanıt yok). Aynı veli şubede iki öğrencinin velisiyse (kardeşler) veli iki öğrenci satırında görünür; Geldi işareti veli başınadır, iki satırda senkron gösterilir (küçük "ortak" ipucu).

### "Geldi" düğmesi — durumlar (min. 44×44; mobilde sağda başparmak bölgesinde)
1. **İşaretlenmedi** (varsayılan, toplantı Başladı iken): çerçeveli nötr düğme "Geldi" + boş kutucuk. Burada "Gelmedi" YAZMAZ.
2. **Geldi** (dokunulmuş): dolu yeşil düğme + tik; tekrar dokunmak işareti geri alır (kısa Toast "Geri alındı" ile). 
3. **Kilitli** (Yayınlandı/İptal/pencere dolmuş): düğme yerine soluk salt okunur kutucuk; Yayınlandı'da hiç düğme yok.
4. **Gelmedi** (yalnız toplantı Tamamlandı olduktan sonra, işaretsizler için): nötr koyu çarpı kutucuğu "Gelmedi" — kırmızı değil. Düzenleme penceresi açıkken dokununca Geldi'ye çevrilebilir.
5. **Eşitlenmedi**: çevrimdışı işaretlenen satırda küçük "bulut-saat" ikonu ("Bağlanınca gönderilecek"); gönderilince kalkar.
Beyanı "Katılacağım" olan veli satırı ile "Yanıt yok" olan veli satırının Geldi düğmesi AYNIDIR — beyandan ön doldurma ve renk ipucu yok. Davetli olmayan biri (kayıtlı olmayan yakın) işaretlenemez: listenin altında sabit bilgi satırı "Listede olmayan bir yakın geldiyse bunu öğrencinin notuna yazın."

## Alt sabit çubuk
**"Toplantıyı tamamla"** (birincil, tam genişlik). Yalnız Başladı'da etkin. Basınca onay (web ConfirmDialog / mobil ConfirmSheet): "Toplantıyı tamamla? **17 veli işaretlenmedi ve 'Gelmedi' sayılacak.** Tamamladıktan sonra 7 gün içinde katılım ve notları düzeltebilirsiniz." Düğmeler "Tamamla" / "Vazgeç". Tamamlandı'da çubuk "Tamamlandı · 6 gün daha düzenlenebilir" bilgi çubuğuna döner (düğme yok). Yayınlandı'da çubuk gizli.

## Sekmeler / ek listeler (rehber öğretmen için)
Üstte segment: **Katılım** (ana) · **Yanıt yok (7)** · **Hesapsız veliler (3)**. Son ikisi PM-A-04 ile aynı satır düzeni: veli adı · öğrenci · telefon · **Ara** (tel:). Rehber öğretmen "Veli adına işaretle" GÖRMEZ (yalnız idare). İdare "Yürüt" sekmesinde bu iki segment yoktur (PM-A-04 sekmeleri zaten var).

## WEB yerleşimi (1440)
İki bölmeli: solda öğrenci/veli listesi (≈60%), sağda PM-T-03 not paneli (≈40%, açık değilse "Bir öğrenci seçin" boş bölmesi). Sayaç şeridi üstte, Toplantıyı tamamla alt sabit. Klavye: ↑↓ satır gezinme, Boşluk = Geldi aç/kapa, Enter = not paneli (odak halkası görünür).

## MOBİL yerleşimi (402×874)
Alt (stack) ekran: STopBar "9-A · Katılım". Sticky sayaç şeridi + arama + çip süzgeci; satırlar tam genişlik; veli satırı yüksekliği ≥ 56 pt; Geldi düğmesi sağ kenarda 88×44. Öğrenci satırına dokunma → not paneli alttan tam yükseklik Sheet (PM-T-03). Alt sabit "Toplantıyı tamamla" (sekme çubuğunun üstünde değil, ekranın en altında; klavye açıkken gizlenir). Uzun listede (40 öğrenci) hızlı harf çubuğu YOK, arama yeterli.

## Örnek veri
9-A, 28 öğrenci, 32 davetli veli. Sayaç: Katılacak 18 · Katılamayacak 4 · Yanıt yok 10 · Geldi 15. Satırlar: 
- Ahmet Kerem Polat — Sevgi Polat · Katılamayacağım (neden ikonu) · Geldi: işaretlenmedi · not ikonu: dolu
- Bilge Naz Aydın — Hacer Aydın · Yanıt yok · **Geldi** ✓ (yanıt vermemiş ama gelmiş: beyandan bağımsız!) · not boş
- Can Efe Sönmez — Murat Sönmez · Yanıt yok · İşaretlenmedi · not boş
- Defne Yalçın — Nermin Yalçın · Katılacağım · **Geldi** ✓ · not dolu; Kadir Yalçın · Yanıt yok · İşaretlenmedi
- Ece Su Korkmaz — Dilek Korkmaz · Hesapsız veli (rozet: "Hesapsız" + Yanıt yok) · İşaretlenmedi
Çevrimdışı örneği: Defne Yalçın satırında "bulut-saat" ikonu; üstte Note "Çevrimdışı — işaretleriniz cihazda, bağlanınca gönderilecek."
Tamamlandı varyantı: işaretsizler "Gelmedi" kutucuğu; üst "Tamamlandı · 6 gün daha düzenlenebilir"; sayaç "Geldi 15 · Gelmedi 17".
Tamamla onayı: "17 veli işaretlenmedi ve 'Gelmedi' sayılacak."

## Durumlar
- Yükleniyor: sayaç + satır iskeletleri.
- Boş: şubede davetli veli yok → EmptyState "Bu şubede davetli veli yok" (+ "İdareye başvurun" bilgisi). Süzgeç sonucu boş: "Bu süzgece uyan kayıt yok".
- Hata: "Tekrar dene". "Geldi" kaydı hatası: satır geri alınır + Toast hata "İşaret kaydedilemedi" + "Tekrar dene".
- İzin yok: 403 (rehber öğretmen olmayan öğretmen). Başka şubenin toplantısı: 404.
- Çevrimdışı: yerel işaretleme + not devam eder (bkz. Eşitlenmedi), üstte kalıcı Note; bağlanınca "N işaret gönderildi" Toast. Çakışma (iki cihaz): son yazan kazanır; ekranda ek bir çözüm arayüzü ÇİZME, yalnız "Başka bir cihazdan güncellendi" Toast.
- Sezon kapalı: tüm düğmeler kilitli, salt okunur.
- Rehber öğretmeni yok (idare yürütüyor): üst Note "Bu şubenin rehber öğretmeni yok; toplantıyı siz yürütüyorsunuz."
- Davet düşmüş satır (veli ilişkisi iptal): soluk "Davet düştü", Geldi düğmesi yok.
- İki idareci/öğretmen eşzamanlı: "Kayıt değişti, yenileyin" Note.

## Bu ekranda ÇİZME
"Gelmedi" düğmesi (işaretlenmeyen = Tamamlandı sonrası Gelmedi sayılır; açık "Gelmedi" işareti yok) · beyana göre otomatik "Geldi" ön doldurma · veli adına yanıt girme (rehberde yok) · davetli olmayan kişi ekleme · yoklama/devamsızlık dili · velilere mesaj gönderme · kıyas/sıralama.
```

## Sipariş sonrası kontrol listesi
- [ ] Beyan rozeti ile "Geldi" düğmesi/kutucuğu görsel olarak ayrı; "Geldi" beyandan ön doldurulmamış mı? (yanıt vermemiş ama gelmiş örneği var mı)
- [ ] Toplantı Tamamlandı olmadan "Gelmedi" yazıyor mu? (yazmamalı)
- [ ] Geldi tek dokunuş, geri alınabilir, ≥ 44pt mi?
- [ ] Tamamla onayı işaretsiz veli sayısını ve "Gelmedi sayılacak" bilgisini yazıyor mu?
- [ ] Düzenleme penceresi geri sayımı ("N gün daha") var mı; süre dolunca salt okunur mu?
- [ ] Çevrimdışı/"Eşitlenmedi" durumu çizilmiş mi?
- [ ] Katılamama nedeni yalnız bu (yetkili) yüzeyde; "Veli adına işaretle" rehberde yok mu?
- [ ] Davetli olmayan yakın için işaretleme yok, nota yazma ipucu var mı?
- [ ] Web iki bölme, mobil Sheet ile not paneli mi?
- [ ] "Yoklama/devamsızlık" dili sızmamış mı?
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **Açık "Gelmedi" işareti:** analiz yalnız "Geldi" işareti tanımlar (işaretsiz = Gelmedi). Prompt "Geldi geri alma" ile yetinir; ayrı "Gelmedi" düğmesi yok. Teyit.
- **Kardeşler / aynı veli iki satırda:** fiili katılım "veli başına" mı "veli × öğrenci" mi? Prompt veli başına, iki satırda senkron varsayar.
- **Rehber öğretmenin "Yanıt yok/Hesapsız" segmentleri:** §5/BR-PM-24 yetki verir; envanterde yeri belirtilmemiş — bu ekrana segment olarak eklendi.
- Sayaçların süzgeç olarak kullanılması envanterde yok (öneri).
- İdare "Yürüt" görünümünde "Veli adına işaretle"nin bulunup bulunmayacağı (PM-A-04 sekmeleri zaten var) — burada çizilmez.
