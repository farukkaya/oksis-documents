# OKSİS — Veli Toplantısı · Claude Design Prompt · **PM-X-01 · Bildirim tercihleri — e-posta sütunu (E-44) ve Veli Toplantısı grubu**

> Kaynak: İhtiyaç analizi §9.2, §10.2 PM-X-01, K-13, BR-PM-34/36, §16 S-6, §12. Platform: **Web + Mobil (eşit)**. Roller: TÜM roller (veli, öğretmen, idare; öğrenci modülün alıcısı değil ama genel tercih ekranını kullanır). Önce `00-ortak-baglam.md`, sonra bu blok. Emsal (mevcut ekran, yeniden çizme değil GENİŞLET): mobil `apps/mobile/src/features/notifications/components/notification-preferences-screen.tsx` ("Bildirim tercihlerim" — grup başlıklı kartlar, satır başına push anahtarı, telefon izni kartı) ve `packages/core/src/notifications/preferences.ts` (grup ve ad kataloğu).

```
# EKRAN: Bildirim tercihleri — e-posta sütunu (PM-X-01)

**Rol:** Tüm roller. **Amaç:** Mevcut "Bildirim tercihlerim" ekranına, her bildirim türü için **E-posta** tercihini eklemek (E-44) ve yeni "Veli Toplantısı" grubunu göstermek. Bugün ekran yalnız "telefona anlık bildirim" (push) anahtarı gösterir; uygulama içi bildirim her zaman gelir ve kapatılamaz. E-posta yalnız bilgilendirmedir (düğme/bağlantı içermez).

## MEVCUT EKRANIN KORUNAN DİLİ (birebir uy)
Mobil: ScreenHeader başlık "Bildirim tercihlerim" (alt başlık güncellenir: "Telefon ve e-posta ile gelen bildirimler"); telefon bildirim izni kartı (Note); grup başlıkları büyük harf mikro etiket (Ödev, Devamsızlık, Notlar, Sınavlar, Kulüpler, Duyurular, Vekâlet, Diğer + YENİ **Veli Toplantısı**); her grup beyaz kart içinde satırlar (≥ 52 pt); anahtar (Toggle) portal renginde; altta bilgi Note'u. Satırlar rol süzgecinden geçer (yalnız o rolün aldığı türler görünür).

## YENİ: İKİ SÜTUNLU TERCİH (Telefon · E-posta)
- **Mobil:** Her grup kartının başlık satırında sağa yaslı iki mikro sütun etiketi **Telefon · E-posta**. Her satır: solda tür adı (en çok 2 satır), sağda iki anahtar (44×44 hedef, "Telefon" altında push anahtarı, "E-posta" altında e-posta anahtarı). Alternatif (Claude Design karar versin, gerekçesini yazsın): satırı iki satıra bölüp tür adının altında iki etiketli anahtar. 
- **Web:** Ayarlar › "Bildirim tercihlerim" (hesap menüsü/profil altından açılır; mobil ekran yalnız mobilde var, web ekranı YENİ çizilir — bkz. karar notu). Gruplanmış tablo: sütunlar **Bildirim türü · Uygulama içi (kilitli, her zaman açık, kilit ikonu) · Telefon bildirimi · E-posta**. Web'de telefon (push) anahtarı işlevsiz kalacağından web'de "Telefon bildirimi" sütunu, cihazdaki tercihi gösteren ama web'de değiştirilemeyen sönük sütun olarak ÇİZİLMEZ: web'de iki sütun yeterli — **Uygulama içi (sabit)** ve **E-posta**; telefon tercihi için bilgi Note'u "Telefon bildirimlerini mobil uygulamadan yönetebilirsiniz." (karar notunda teyit).

### Hücre durumları (E-posta)
1. **Açık / Kapalı** anahtar (normal).
2. **Bu tür e-postayla gönderilmez** (türün e-posta kanalı yok): anahtar yerine sönük "—" ve erişilebilir ipucu "Bu bildirim e-postayla gönderilmez". (Veli toplantısı türlerinde yalnız N-01 Duyuru, N-03 Değişiklik, N-04 İptal e-posta destekler; hatırlatma türleri desteklemez.)
3. **E-posta adresi yok** (hesapta/kişide adres boş): TÜM e-posta sütunu pasif (anahtarlar devre dışı, sönük) + sütun üstünde ve ekranın tepesinde Note (uyarı tonu): "E-posta adresiniz kayıtlı değil. **Profilinize e-posta ekleyin**" + "Profile git" bağlantısı. Push/uygulama içi etkilenmez.
4. **Varsayılan açık**: kullanıcı hiç dokunmamışsa Veli Toplantısı N-01/N-03/N-04 e-postası AÇIK görünür (katalog varsayılanı); özel bir "varsayılan" rozeti ÇİZME.
Üst bölümde yeni bilgi kartı "E-posta adresiniz": maskeli adres (`n***.y***@ornek.example` gibi tam adres de gösterilebilir) ve açıklama Note'u: "E-postalar yalnız bilgilendirmedir; toplantı yanıtınızı OKSİS uygulamasından verirsiniz." Telefon izni kartı (push) yerinde kalır.

## YENİ GRUP: VELİ TOPLANTISI (rol süzgeçli)
| Tür adı (ekranda) | Veli | Sınıf Rehber Öğr. | İdare | E-posta desteği |
|---|---|---|---|---|
| Veli toplantısı duyuruldu | ✓ | — | — | ✓ |
| Veli toplantısı değişti (saat/yer) | ✓ | — | — | ✓ |
| Veli toplantısı iptal edildi | ✓ | — | — | ✓ |
| Toplantı günü hatırlatması | ✓ | ✓ | — | — |
| Veli toplantısı başlıyor | ✓ | — | — | — |
| Toplantıya yanıtınız bekleniyor | ✓ | — | — | — |
| Şube veli toplantısı (plan, değişiklik, iptal) | — | ✓ | — | — |
İdare bu grupta hiçbir tür görmez (modül bildirimi idareye gitmez). Öğrenci bu grubu görmez. Tür adlarının Türkçe metni tasarım önerisidir; eşleme çekirdek katalogda (`preferences.ts`) tutulacak.

## ROL VARYANTLARI (ayrı çerçeveler)
- **Veli (Nermin Yalçın)** — mobil + web: Veli Toplantısı grubu 6 satır (N-01…N-07 içinden veli türleri), Ödev/Duyurular/Notlar grupları.
- **Sınıf Rehber Öğretmeni (Selma Arslan)** — mobil + web: Veli Toplantısı grubu 2 satır.
- **İdare (Neslihan Çetin)** — mobil + web: grup görünmez; e-posta sütunu diğer türlerde var.

## Örnek veri
Veli: "Veli Toplantısı" grubu: Veli toplantısı duyuruldu (Telefon açık · E-posta açık) · Veli toplantısı değişti (açık · açık) · Veli toplantısı iptal edildi (açık · açık) · Toplantı günü hatırlatması (açık · —) · Veli toplantısı başlıyor (kapalı · —) · Toplantıya yanıtınız bekleniyor (açık · —). "Ödev" grubundan iki satır: Yeni ödev ve değişiklikleri (açık · kapalı) ve Son teslim yaklaşıyor (açık · —) — E-posta desteği kataloğa bağlı olduğundan hangi mevcut türlerin e-posta destekleyeceği bu çerçevede ÖRNEKTİR.
Adres yok varyantı: Hacer Aydın'ın hesabında e-posta boş → e-posta sütunu pasif.
E-posta: nermin.yalcin@ornek.example (kurgu).

## Durumlar
- Yükleniyor: grup kart iskeletleri (5 satır).
- Hata: kart içi hata + "Tekrar dene". Kayıt hatası: anahtar iyimser döner ve geri alınır + Toast hata.
- Boş: "Ayarlanacak bildirim yok" (mevcut kalıp).
- Telefon izni kapalı/reddedildi: mevcut izin kartı kalıpları (Note + "Telefon ayarlarını aç"); e-posta sütunundan bağımsız.
- E-posta adresi yok: yukarıdaki pasif sütun.
- İzin yok/oturum: mevcut genel kalıp. Çevrimdışı: anahtarlar devre dışı + "Bağlantı yok".

## Bu ekranda ÇİZME
E-posta adresi düzenleme formu (profilin işi; yalnız bağlantı) · SMS tercihi (V1'de gönderim yok) · uygulama içi bildirimi kapatma anahtarı (kapatılamaz) · sessiz saat ayarı (ayrı ayar) · "e-postadan yanıt ver" önizlemesi (e-postada düğme yoktur) · toplu "tümünü kapat" (envanterde yok).
```

## Sipariş sonrası kontrol listesi
- [ ] Mevcut ekranın dili korunmuş, "Telefon · E-posta" iki sütun net mi; 44pt anahtar hedefleri?
- [ ] E-posta desteklemeyen türde "—" + ipucu var; anahtar yok mu?
- [ ] E-posta adresi yokken sütun pasif + "Profilinize e-posta ekleyin" bilgisi var mı?
- [ ] Uygulama içi bildirim kapatılabiliyormuş gibi çizilmemiş mi (kilitli/sabit)?
- [ ] İdare ve öğrenci için Veli Toplantısı grubu görünmüyor mu; rol süzgeci çerçeveleri var mı?
- [ ] E-posta yalnız bilgilendirme notu var mı?
- [ ] Mobilde zeminde yüzen metin yok (bölüm etiketi kuralı mevcut ekranla çelişmiyor mu) — mevcut ekran büyük harf bölüm etiketini zeminde kullanıyor; yüzey kuralı için etiketi kart başlık satırına almayı öner.
- [ ] `00-ortak-baglam.md` ortak kontrol listesi.

## Envanterde belirsiz / karar
- **Web'de bu ekran yok:** mevcut "Bildirim tercihlerim" yalnız mobilde (web push kapsam dışı). Envanter PM-X-01'i web ✓ işaretler; web sayfası ve menü konumu yeni çizilir (öneri: hesap menüsü). Web'de telefon (push) sütununun gösterimi belirsiz — prompt iki sütun (Uygulama içi + E-posta) + bilgi notu önerir.
- **Hangi mevcut türlerin e-posta desteklediği:** analiz "tüm türler için alıcı seçimi açılır" der; her türün `DefaultEmailEnabled` ve `RecipientCanChooseEmail` değerleri katalogdan gelir. Hücre durumu (2) bu yüzden tasarımda.
- Veli toplantısı tür adlarının metni (analizde yalnız kod adı ve içerik var) öneridir.
- E-posta adresi ekleme yeri (profil) envanterde tanımlı değil; yalnız bağlantı çizilir.
- Mevcut ekranın bölüm etiketleri zemine basılıyor; "zemin üzerine metin yok" kuralı bu ekranda nasıl uygulanacak (etiketi kart içine al) — Faruk teyidi.
