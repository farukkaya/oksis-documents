---
tags: [teknik/i18n, kural, sozluk]
date: 2026-10-23
status: approved
sahibi: oksis-education-expert
onay: Faruk (2026-10-23 — S-1…S-20 öneriler aynen kabul; S-21 → yabancı dil adları çevrilmez)
---

# OKSİS i18n Terim Sözlüğü — Türkçe → İngilizce (taslak)

Bu sözlük [[0029-cok-dil-altyapisi-ingilizce]] kararının bağlayıcı terim kaynağıdır. Mevcut sistem metinleri (arayüz, hata, bildirim, e-posta, Excel, referans verisi) Claude tarafından toplu çevrilirken **bu tablolardaki karşılıklar dışına çıkılmaz**. Onaylandıktan sonra Tolgee glossary'sine aktarılır. Bu sözlükte olmayan yeni bir alan kavramı çıkarsa önce buraya önerilir, sonra çevrilir.

**Kapsam:** Sistem metinleri. Kullanıcı içeriği (duyuru, ödev, not açıklaması, okul/şube/kişi adları) kapsam dışıdır. Ders adları, ders kataloğu ve branş adları **çevrilmez** (bkz. §16).

**Kaynaklar:** `oksis-ui/CLAUDE.md` MEB sözlüğü (UI identifier'ları), `docs/domain/` kavram notları ve `aliases`, `teknik/kurallar/ortak/naming-conventions.md`, `oksis-api/src/Oksis.Infrastructure/Persistence/Seed/MasterData/*` (2026-10-23 okundu), `packages/core` etiket haritaları.

**Güven:** *yüksek* = tartışmasız ya da kodla/sektörle yerleşik; *orta* = makul ama başka seçenek de savunulabilir; *düşük* = Faruk kararı gerekir (§17'de seçenekleriyle).

---

## 1. Genel ilkeler

### 1.1 Varyant: Amerikan İngilizcesi (en-US) — öneri

Sözlük **tek varyantta, Amerikan İngilizcesinde** tutulur (yazım: *color, canceled, center, enroll, counselor, advisor, labor*). Gerekçe:

1. **Sınıf numarası birebir eşleşir.** MEB "9. sınıf" = US "Grade 9". İngiliz sisteminde aynı yaş "Year 10"dur; "Year 9" yazmak yabancı veliye çocuğunun bir yıl geride olduğunu söyler. Bu, varyant seçiminin en ağır gerekçesidir.
2. **Türk sisteminin yapısı ABD'ye yakın:** iki dönemli yıl (= *semester*; İngiltere'de yıl üç *term*'dür), müdür (= *principal*), sınıf rehber öğretmeni (= *homeroom teacher*), karne (= *report card*), not (= *grade*).
3. **Kod zaten bu yönde:** `homeroomTeacher`, `VicePrincipal`, `gradeLevel`, `reportCard`, `Proctor` yerine `Invigilator` gibi birkaç istisna dışında (bkz. §1.6).
4. MEB'in kendi İngilizce yayınları ve Türk üniversiteleri *semester*, *grade*, *principal* kullanır; hedef kitle (Türkiye'de yaşayan yabancı veli/öğretmen) bu dille karşılaşır.

**Bilinçli istisnalar** (ayrı ayrı gerekçeli): resmî tatillerin Türk devletinin kullandığı resmî İngilizce adları (*Labour and Solidarity Day* — özel isim, §14), *Canteen* (Türk okulundaki kantin ABD'de karşılığı olmayan bir yerdir, *Cafeteria* yemekhaneye ayrılmıştır), alerjen adları (AB 1169/2011 listesi; yine de US yazımı *Sulfites* kullanılır).

**Biçim (varyanttan ayrı karar, §17 S-1):** dil `en`, ama tarih **gün önce** (`23 Oct 2026` / `23/10/2026`) ve saat **24 saat** (`14:30`). Türkiye'de yaşayan kullanıcı için `10/23/2026` ve `2:30 PM` yanıltıcıdır. Ondalık ayırıcı İngilizcede noktadır: "0,5 gün" → "0.5 days".

### 1.2 Türk eğitim sistemine özgü kavramlar — üç strateji

| Strateji | Ne zaman | Örnek |
|---|---|---|
| **A. Doğal karşılık** | İngilizcede aynı işlevi gören yerleşik kavram varsa | Karne → *Report Card*, Yoklama → *Attendance*, Teneffüs → *Break* |
| **B. Açıklamalı karşılık** | Karşılık var ama Türk kuralı farklı işliyorsa; terim çevrilir, ilk geçtiği yerde ya da yardım metninde kısa açıklama verilir | Özürsüz devamsızlık → *Unexcused absence* (+ "10-day limit"), Nöbet → *Duty* |
| **C. Türkçeyi koru (+ parantez açıklaması)** | Kurum/sistem adı, kısaltma ya da İngilizcede karşılığı olmayan resmî ad | MEB, e-Okul, OYSS, KVKK, T.C. Kimlik No |

Varsayılan A'dır. B'yi yalnız kuralın farkı kullanıcıyı yanıltacaksa kullan. C'yi yalnız özel ad ve kısaltmada kullan — bir alan kavramını Türkçe bırakmak (ör. "Nöbet") yabancı öğretmene hiçbir şey söylemez.

### 1.3 Kısaltmalar

- MEB, e-Okul, MEBBİS, OYSS, KVKK, LGS, YKS, BİLSEM, RAM **korunur**, Türkçe harfleriyle (İ, Ş dahil) yazılır.
- Kullanıcının ilk karşılaştığı yerde (yardım metni, ayar açıklaması, bilgi şeridi) bir kez açılımı verilir: "MEB (Ministry of National Education)". Düğme ve sütun başlığında açılım verilmez.
- Kısaltma çoğulu/iyelik eki İngilizcede yoktur: "e-Okul'a aktar" → "Export to e-Okul".

### 1.4 Büyük harf kullanımı

- **Title Case:** menü/gezinme öğeleri, sayfa başlıkları, sekme adları, modal başlıkları. ("Report Cards", "Duty Roster")
- **Sentence case:** düğmeler, form etiketleri, sütun başlıkları, durum rozetleri, hata/bilgi mesajları, bildirim ve e-posta metni. ("Save draft", "Due date", "Not taken")
- Cümle içinde alan terimleri küçük harfle yazılır ("your homeroom teacher"); özel adlar (MEB, resmî tatiller, rol adları menüde) büyük.
- Türkçe kaynaktaki BÜYÜK HARF vurgusu İngilizceye taşınmaz.
- **Türkçede de aynı kural (2026-10-26):** sayfa/sekme/bölüm/modal başlıkları Title Case ("Veli Toplantısı Ayarları"); düğmeler ve soru başlıkları sentence case ("Değişiklikleri kaydet", "Save changes"). **Başlık anahtarı düğmede yeniden kullanılmaz;** düğme için ayrı `.confirm`/`.action` anahtarı açılır (başlık Title Case, düğme sentence case).

### 1.5 Ton ve dil bilgisi

- Ton: sade, doğrudan, kibar; "you" ile hitap. Türkçedeki "sen/siz" ayrımı İngilizcede yoktur, her ikisi "you".
- Düğmeler emir kipinde fiil: "Kaydet" → "Save", "Yayınla" → "Publish". "Vazgeç" → "Cancel".
- Durumlar sıfat/geçmiş zaman: "Yayınlandı" → "Published", "İptal" (durum) → "Canceled".
- **Çoğul:** İngilizce tekil/çoğul ayrımı ICU plural ile kurulur ("1 day / 2 days", "1 absence / 3 absences"); Türkçedeki "3 gün" kalıbı doğrudan kopyalanmaz.
- **Sıra sayıları:** "1. Sınav" → "Exam 1"; "3. ders" → "Period 3"; "9. Sınıf" → "Grade 9"; "1. Dönem" → "Semester 1"; "1. tercih" → "1st choice". Türkçedeki "N." biçimi İngilizceye taşınmaz.
- **Homework sayılamaz:** "bir ödev" → "a homework assignment", "3 ödev" → "3 assignments", "ödevler" (modül) → "Homework". "a homework" / "homeworks" yasak.
- Tek harfli kısa kodlar (yoklama ızgarası "G", "İ", "R") İngilizce harflerle yeniden kurulur (§4.1).

### 1.6 Kod adı ile arayüz terimi

Arayüz terimi kod adıyla aynı olmak zorunda değil; anahtar (`<modül>.<ekran>.<öğe>`) kod adını, değer arayüz terimini taşır. Fark bulunan satırlar tablolarda "Kod:" notuyla işaretlidir. Önemli farklar:

| Kavram | Backend | UI identifier | Arayüz (en) |
|---|---|---|---|
| Not | `Mark` | `grade` | Grade |
| Sınıf seviyesi | `GradeLevel` | `gradeLevel` | Grade (sayıyla), Grade level (tek başına) |
| Şube | `ClassRoom` | `section` | Class (öneri, §17 S-2) |
| Derslik | `Room` | `classroom` | Room |
| Sezon | `AcademicSession` | `academicSession` | Academic year |
| Dönem | `AcademicTerm` | — | Semester |
| Ders programı | `Timetable` / `ScheduleProgram` | `schedule` | Class schedule |
| Gözetmen | `Invigilator` | — | Proctor |
| Mazeret | `AttendanceExcuse` | `excuse` | Excuse |

---

## 2. Okul yapısı ve akademik takvim

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Okul | School | Kod `School` (tenant). | yüksek |
| Kurum | Institution | "Okul" ile eşanlamlı kullanılıyorsa "School" tercih edilir. | orta |
| Kademe | School level | UI `schoolStage`. Değerleri aşağıda. **Çelişki:** domain notu [[Sınıf Seviyesi]] "Kademe"yi sınıf seviyesinin eşanlamlısı sayıyor; UI sözlüğü kademeyi Anaokulu/İlkokul/Ortaokul/Lise sayıyor. Çeviri UI sözlüğünü izler. | orta |
| Anaokulu / Okul öncesi (kademe) | Preschool | `EducationLevel.Preschool`. Bkz. §17 S-7. | orta |
| İlkokul | Primary school | US'de "elementary school" daha yaygın; uluslararası programlarda (IB PYP, Cambridge Primary) "primary" yerleşik. Bkz. §17 S-7. | orta |
| Ortaokul | Middle school | | yüksek |
| Lise | High school | | yüksek |
| Sınıf (seviye) | Grade (sayıyla: "Grade 9"); tek başına: Grade level | Kod `GradeLevel`, UI `gradeLevel`. "9th grade" değil "Grade 9" (sıralanabilir, kısa). UK "Year 10" KULLANILMAZ (bkz. §1.1). | yüksek |
| Anaokulu (sınıf seviyesi, kod `0`) | Preschool | Kademe ile aynı ad; seed'de tek satır. | orta |
| 1. Sınıf … 12. Sınıf | Grade 1 … Grade 12 | Seed `GradeLevelSeedData`. | yüksek |
| Hazırlık Sınıfı | Preparatory year | Lisedeki dil hazırlığı; "Prep year" kısa biçim (rozet). | yüksek |
| Şube | Class | Kod `ClassRoom`, UI `section`. "Class 9-A". Bkz. §17 S-2 (Section / Homeroom seçenekleri). | düşük |
| Şube adı (9-A) | Class name | Değer (9-A) çevrilmez. | orta |
| Sınıflar ve Şubeler (modül) | Grades & Classes | Şube = Class kararına bağlı. | orta |
| Derslik | Room | Kod `Room`, UI `classroom`. "Classroom" yerine "Room": şube = Class iken "Classroom" karışır. | yüksek |
| Kapasite (şube/derslik) | Capacity | | yüksek |
| Sezon | Academic year | Ürün dilindeki "sezon" = MEB "eğitim-öğretim yılı". **"Season" yasak** (İngilizcede spor sezonu). Kod `AcademicSession`. Bkz. §17 S-5. | orta |
| Eğitim-öğretim yılı | Academic year | "Sezon" ile aynı karşılık. | yüksek |
| Aktif sezon | Current academic year | | yüksek |
| Arşiv sezonu | Archived academic year / Past year | | orta |
| Sezon geçişi | Year-end rollover | Yeni yıla geçiş işlemi. | orta |
| Yenileme dönemi | Re-enrollment period | | orta |
| Hazırlık (sezon durumu `Setup`) | Setup | | yüksek |
| Dönem | Semester | MEB'de yılda iki dönem; US "semester". "Term" UK'de üç dönemi çağrıştırır. Kod `AcademicTerm`. Bkz. §17 S-4. | orta |
| 1. Dönem / 2. Dönem | Semester 1 / Semester 2 | Seed `AcademicTermTypeSeedData`. | orta |
| Dönem (genel "süre" anlamında: tercih dönemi, yenileme dönemi) | Period | Akademik dönemle karıştırılmaz. | yüksek |
| Akademik takvim | Academic calendar | UI `academicCalendar`. | yüksek |
| Okul ayarları | School settings | | yüksek |
| Yapı (ayar) | Structure | | yüksek |
| Akademik politika | Academic policy | | yüksek |
| Kurum bilgileri | School information | | yüksek |
| MEB kurum kodu | MEB school code | Kısaltma korunur. | yüksek |
| Modül | Module | | yüksek |

### 2.1 Tatil türleri

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Tatil | Holiday | Kod `Holiday`. | yüksek |
| Resmî tatil | Public holiday | Kod `OfficialHoliday`. US "federal holiday" Türkiye için yanlış. | yüksek |
| Ulusal bayram | National holiday | | yüksek |
| Dini bayram | Religious holiday | | yüksek |
| Okul etkinliği (tatil türü) | School event | | yüksek |
| Eğitime kapalı gün | No-school day | | orta |
| Yarıyıl tatili | Midyear break | İki dönem arası (Ocak–Şubat). "Semester break" da anlaşılır. | orta |
| Ara tatil | Mid-semester break | MEB'in Kasım ve Nisan ara tatilleri. | orta |
| Acil tatil | Emergency closure | Kod `HolidayType.EmergencyClosure`. | yüksek |
| Arife | Eve | "Ramadan Feast Eve". Yarım gün olduğu ayrıca belirtilir. | yüksek |
| Yarım gün (tatil) | Half day | | yüksek |
| Okul günü | School day | | yüksek |

---

## 3. Roller ve kişiler

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Süper Admin | Super Admin | Platform rolü (`SUPER_ADMIN`). | yüksek |
| Okul Yöneticisi (rol) | School Administrator | `SCHOOL_ADMIN`; seed açıklaması "Müdür". Rol adı olarak "Principal" değil "School Administrator": rolü müdür dışında biri de taşıyabilir. Bkz. §17 S-19. | orta |
| Müdür | Principal | | yüksek |
| Müdür Yardımcısı | Vice Principal | `VICE_PRINCIPAL`. US'de "Assistant Principal" de yaygın; kod adıyla aynı olan seçildi. | orta |
| İdare | Administration | "İdare onayı" → "administration approval". | yüksek |
| Okul Müdürlüğü (duyuru imzası) | Principal's Office | | orta |
| Öğretmen | Teacher | | yüksek |
| Sınıf öğretmeni (ilkokul) | Classroom teacher | İlkokulda bütün dersleri okutan öğretmen; branş öğretmeninden ayrılır. | orta |
| Branş öğretmeni | Subject teacher | | yüksek |
| Sınıf rehber öğretmeni | Homeroom teacher | UI `homeroomTeacher`. Tek başına "Rehber" denmez (UI kuralı). | yüksek |
| Rehber öğretmen / Psikolojik danışman | School counselor | Rol henüz yok (`Counselor` ertelendi). "Guidance counselor" eski; bkz. §17 S-16. | orta |
| Rehberlik servisi | Counseling office | | orta |
| Rehberlik (alan) | Guidance and counseling | | yüksek |
| Rehber (kişi rehberi) | Directory | UI `contact`. "Guide" değil. | yüksek |
| Nöbetçi öğretmen | Duty teacher | "Teacher on duty" cümle içinde. | yüksek |
| Vekil öğretmen | Substitute teacher | Kısa: "Substitute". | yüksek |
| Danışman (kulüp) | Advisor | US yazımı (UK "adviser"). | yüksek |
| Gözetmen | Proctor | Kod `Invigilator` (UK). US varyantında "Proctor". Bkz. §17 S-11. | orta |
| Veli | Parent | Kod `Parent`. Veli hukuken vasi olabilir; yasal bağlamda (KVKK, rıza) "parent or guardian". Bkz. §17 S-17. | orta |
| Öğrenci | Student | | yüksek |
| Sekreter | Secretary | Okul sekreterliği → "School office". | orta |
| Sekreterlik | School office | | orta |
| Muhasebeci | Accountant | | yüksek |
| Personel | Staff | | yüksek |
| Kişi | Person | Kod `Person`. Arayüzde çoğunlukla "user" ya da bağlama göre rol adı yeterli. | orta |
| Kullanıcı | User | | yüksek |
| Hesap | Account | | yüksek |
| Profil (Profilim) | Profile (My Profile) | | yüksek |
| Rol | Role | | yüksek |
| Rol ataması | Role assignment | | yüksek |
| İzin (yetki) | Permission | Öğretmen izniyle (leave) karışmaz. | yüksek |
| Davet | Invitation | | yüksek |
| Davetli | Invitee | | yüksek |
| Çocuklarım (veli) | My Children | | yüksek |
| Portal | Portal | | yüksek |

---

## 4. Yoklama ve devamsızlık

> Mevzuat bağlamı: ortaöğretimde devamsızlık sınırı Ortaöğretim Kurumları Yönetmeliği **md. 36**'dadır (özürsüz 10 gün, toplam 30 gün). Madde numarası ikincil kaynaklardan; **birincil metinden doğrulanmadı** — bağlantı: <https://www.mevzuat.gov.tr/> (Ortaöğretim Kurumları Yönetmeliği, RG 07.09.2013). İlkokul/ortaokul için Okul Öncesi Eğitim ve İlköğretim Kurumları Yönetmeliği — madde numarası **doğrulanmadı**. Çeviri bu sınırları adıyla değil sayı ve ayarla verir ("10-day unexcused absence limit"), çünkü yabancı kullanıcı yönetmeliği tanımaz.

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Yoklama | Attendance | UI `rollCall`. Fiil: "Yoklama al" → "Take attendance". "Roll call" yalnız etkinlik sayımında. | yüksek |
| Yoklama al / gönder | Take attendance / Submit attendance | | yüksek |
| Devamsızlık (modül) | Attendance | Menü adı. | yüksek |
| Devamsızlık (sayılan) | Absence(s) | "12 days of absence" / "Absences: 12". | yüksek |
| Ders oturumu | Lesson | UI `attendanceSession`. Yoklamanın bağlandığı tarihli ders. Teknik bağlamda "class session". | orta |
| Özürlü devamsızlık | Excused absence | MEB: izinli + raporlu. | yüksek |
| Özürsüz devamsızlık | Unexcused absence | | yüksek |
| Özürsüz devamsızlık hakkı/sınırı | Unexcused absence limit | Açıklamalı (B): yardım metninde "exceeding it means repeating the grade". | yüksek |
| Toplam devamsızlık sınırı | Total absence limit | | yüksek |
| Devamsızlık eşiği (uyarı) | Absence warning threshold | Seed `ABSENCE_THRESHOLD_PERCENT`. | yüksek |
| Uyarı eşiği aşıldı | Absence warning threshold reached | Bildirim türü `ATT_THRESHOLD`. | yüksek |
| Risk altındaki öğrenciler | Students at risk | | yüksek |
| Gün eşdeğeri | Day equivalent | Ders sayısının güne çevrilmesi. | orta |
| Yarım gün (devamsızlık) | Half day | "0.5 days". | yüksek |
| Tam gün | Full day | | yüksek |
| Geç kalma birikimi | Late accumulation | "Every 3 lates count as half a day." | orta |
| Geç sayılma sınırı | Late threshold | Seed `ATTENDANCE_LATE_THRESHOLD_MINUTES`: "Late threshold (minutes)". | yüksek |
| Devreden devamsızlık | Carried-over absences | Nakil öğrencinin önceki okul toplamı. UI `absenceCarryOver`. | orta |
| Devamsızlık özeti | Absence summary | | yüksek |
| Mazeret | Excuse | UI `excuse`. "Absence excuse" bağlam gerekirse. | yüksek |
| Mazeret bildir | Submit an excuse | | yüksek |
| Yeni mazeret bildirimi | New absence excuse | Bildirim `ATT_EXCUSE_SUBMITTED`. | yüksek |
| Sağlık raporu | Medical certificate | "Doctor's note" daha gündelik; resmî belge olduğu için "medical certificate". | orta |
| Dilekçe | Written request | Bağlama göre "parent's letter". | orta |
| Destekleyici belge | Supporting document | | yüksek |
| Gün içi izin | Early dismissal | UI `dailyLeave`. Öğrencinin gün ortasında ayrılması. | orta |
| Erken ayrıldı | Left early | | yüksek |
| Düzeltme talebi | Correction request | UI `amendmentRequest`. "Amendment" İngilizcede resmî/hukuki; arayüzde "correction". | orta |
| Düzeltme penceresi | Correction window | | yüksek |
| Yeniden açma talebi | Reopen request | | yüksek |
| Yoklama size açıldı | Attendance reopened for you | Bildirim `ATT_SESSION_REOPENED`. | yüksek |
| Geriye dönük giriş (retro) | Retroactive entry | Şerh: "Entered retroactively". | yüksek |
| Alınmayanlar | Not taken | Liste başlığı: "Attendance not taken". | yüksek |
| Yoklama hatırlatması | Attendance reminder | | yüksek |
| Günlük yoklama özeti | Daily attendance summary | | yüksek |
| Öğrenci ilk derse gelmedi | Student absent from first period | Bildirim `ATT_ABSENT`. | yüksek |
| Canlı pano | Live dashboard | | yüksek |
| Devam sorgusu (Şu an okulda mı?) | Attendance lookup | | orta |
| Öğrenci takibi (menü grubu) | Student Tracking | UI `studentTracking`. | orta |
| Etkinlik yoklaması | Activity roll call | UI `activityRollCall`. Gezi/tören güvenlik sayımı; devamsızlığa girmez. | orta |
| Sayım anı | Head count | "Count round" anlaşılmaz; "head count" yerleşik. | orta |
| Sorumlu devri | Handover | UI `activityHandover`. | yüksek |
| Sonradan eklenen (katılımcı) | Added later | | yüksek |
| Kulüp saati yoklaması | Club period attendance | | yüksek |

### 4.1 Yoklama durumları (enum etiketleri)

| Türkçe | İngilizce | Kısa kod | Not / gerekçe | Güven |
|---|---|---|---|---|
| Geldi (Var) | Present | ✓ | `present` | yüksek |
| Gelmedi (Yok) | Absent | ✕ | `absent` | yüksek |
| Geç Geldi | Late | L | `late`. US "tardy" okul jargonudur, yabancı veli için "late" daha açık. | yüksek |
| İzinli | Excused | E | `excused`. Bkz. §17 S-10: Raporlu da özürlüdür; "Excused" etiketi bunu gizler. | orta |
| Raporlu | Medical | M | `medicalReport`. Bkz. §17 S-10. | düşük |
| Bekliyor (oturum) | Pending | | | yüksek |
| Süre Devam Ediyor | In progress | | | yüksek |
| Tamamlandı | Completed | | | yüksek |
| Alınmadı | Not taken | | | yüksek |
| İptal | Canceled | | US yazımı tek "l". | yüksek |
| Tümü sayıldı / Kısmen sayıldı / Sayım yapılmadı / Sayım bekliyor | All counted / Partially counted / Not counted / Awaiting count | | | yüksek |
| Rapor / izin (devir gerekçesi) | Sick leave / leave | | `medicalLeave` | orta |
| Görev değişikliği | Role change | | | yüksek |
| Nöbet devri | Duty handover | | | yüksek |

---

## 5. Notlar, sınavlar, karne

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Not (modül/menü) | Grades | UI `grade`, kod `Mark`. Bkz. §17 S-3 (sınıf seviyesiyle çakışma). | orta |
| Not (tek değer) | Grade | Sayısal değerden söz edilirken "score" kabul edilir ("Score: 85"). | orta |
| Puan | Score | | yüksek |
| Not defteri | Gradebook | Kod `GradeBook`. | yüksek |
| Not girişi | Grade entry | | yüksek |
| Eksik not girişi hatırlatması | Missing grade entry reminder | | yüksek |
| Sınav notu girildi | New exam grade published | Bildirim `GRADE_PUBLISHED`. | yüksek |
| Sınav (not defteri sütunu) | Exam | UI `assessment`. "1. Sınav" → "Exam 1". Numaralandırma dönem içinde. | yüksek |
| Değerlendirme | Assessment | Genel kavram (sütun); sözlü/performans/proje dahil. | yüksek |
| Yazılı | — (kullanılmaz) | UI kuralı: "Yazılı" deme. Yalnız tarif gerekirse "written exam". OYSS bağlamında MEB metni "yazılı" diyor — orada da "Exam" (bkz. §13). | yüksek |
| 1. Sınav / 2. Sınav | Exam 1 / Exam 2 | Seed `VZ1–VZ4`. Açıklama "2. Dönem 1. Sınavı" → "Semester 2, Exam 1". | yüksek |
| Sözlü | Oral | Seed `SZ`; açıklama "Sözlü Değerlendirme" → "Oral assessment". | yüksek |
| Performans | Performance task | Seed `PER`; "Performans Görevi". Sütun başlığında kısa "Performance". | yüksek |
| Proje | Project | Seed `PRJ`; "Proje Görevi" → "Project assignment". | yüksek |
| Sınav türü | Exam type | Kod `ExamType`. | yüksek |
| Girmedi | Did not take | Ortalamaya katılmaz. Rozet kısa: "DNT" kullanılmaz, tam yazılır. | orta |
| Muaf | Exempt | | yüksek |
| Ortalama | Average | | yüksek |
| Dönem puanı / dönem ortalaması | Semester average | | yüksek |
| Yıl sonu başarı puanı | Year-end average | | orta |
| Ağırlık (sınav ağırlığı) | Weight | | yüksek |
| Not ölçeği | Grading scale | Kod `GradeScale`. | yüksek |
| 100'lük sistem | 100-point scale | Seed `TR_100`; açıklama "Türkiye standart" → "Turkish standard". | yüksek |
| 5'lik sistem | 5-point scale | Seed `TR_5`; "Eski MEB sistemi" → "Former MEB scale". | yüksek |
| Harf sistemi | Letter grades | Seed `HARFLI`. | yüksek |
| Geçme notu | Passing grade | | yüksek |
| Kilitli / Kilit aç | Locked / Unlock | | yüksek |
| Yayından geri al | Unpublish | | yüksek |
| Denetim izi | Audit trail | | yüksek |
| Karne | Report card | UI `reportCard`. | yüksek |
| Karne yayınlandı | Report card published | | yüksek |
| Takdir belgesi | High Honors certificate | Bkz. §17 S-8. MEB: dönem ağırlıklı ortalaması 85 ve üzeri — eşik ve madde numarası **doğrulanmadı** (ikincil kaynak: OÖKY "md. 157/160" çelişkili). | düşük |
| Teşekkür belgesi | Honors certificate | Bkz. §17 S-8. 70–84,99 (doğrulanmadı). | düşük |
| Takdir / Teşekkür alt sınırı | High Honors / Honors threshold | Okul ayarı. | orta |
| Üstün başarı belgesi | Certificate of Outstanding Achievement | | orta |
| Onur belgesi | Certificate of Merit | Davranış/örnek tutum ödülü; akademik değil. Bkz. §17 S-9. | düşük |
| Sınıf tekrarı | Repeating the grade | US "grade retention" teknik; arayüzde "repeat the grade". | orta |
| Sınıf geçme | Promotion to the next grade | | orta |
| Mezuniyet | Graduation | | yüksek |

### 5.1 Sınav takvimi

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Sınav takvimi | Exam schedule | UI `examSchedule`; akademik takvimle karışmaz. | yüksek |
| Sınav haftası / Sınav penceresi | Exam week / Exam window | Kod `ExamWindow`. Arayüzde "Exam week". | orta |
| Sınav haftası duyuruldu | Exam week announced | | yüksek |
| Planlanmış sınav | Scheduled exam | | yüksek |
| Sınav saati | Exam period | Ders saatine yerleşen sınav. | orta |
| Saat isteği (ödünç ders saati) | Period request | "Hour request" anlaşılmaz; okul dili "period". | orta |
| Sınav saati seçilmedi hatırlatması | Exam period not selected reminder | | yüksek |
| Sınav tarihi değişti | Exam date changed | | yüksek |
| Yarın sınav var | Exam tomorrow | | yüksek |
| Sınav planı görüşe açıldı | Exam plan open for review | | yüksek |
| Görüş | Comment | "Review comment". | orta |
| Sınav gözetmenliği değişti | Proctoring assignment changed | | orta |
| Sınav oturumu | Exam session | | yüksek |
| Sınav dersliği | Exam room | | yüksek |
| Oturma planı | Seating plan | | yüksek |
| Kelebek sınav sistemi | Mixed seating | Farklı şubelerin karışık oturduğu düzen. Bkz. §17 S-12. | düşük |
| Ortak sınav | Common exam | | yüksek |
| Merkezi sınav takvimi | Central exam calendar | | yüksek |
| Ülke geneli sınav | Nationwide exam | | yüksek |
| Okul geneli sınav | School-wide exam | | yüksek |

---

## 6. Ders programı, zil, görevlendirme, müfredat

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Ders (müfredat dersi, genel kavram) | Subject | Ders **adları** çevrilmez (§16). "Ders seç" → "Select subject". UI `course`, kod `Subject`. | orta |
| Ders (programdaki tek ders, zaman dilimi) | Lesson | "Bugün 6 dersin var" → "You have 6 lessons today". | yüksek |
| Ders saati (zil periyodu) | Period | "3. ders" → "Period 3". | yüksek |
| Ders programı | Class schedule | UI `schedule`, kod `Timetable`. Bkz. §17 S-6. | orta |
| Haftalık ders programı | Weekly schedule | | yüksek |
| Programım | My Schedule | | yüksek |
| Program sürümü | Schedule version | | yüksek |
| Yayınla (program) | Publish | | yüksek |
| Program istisnası | Schedule exception | | yüksek |
| Ders iptali | Lesson canceled | | yüksek |
| Zil çizelgesi / Zil programı | Bell schedule | UI `bellSchedule`. | yüksek |
| Teneffüs | Break | US ilkokul "recess" dar; "break" tüm kademelerde doğru. | yüksek |
| Teneffüs süresi (dk) | Break length (min) | | yüksek |
| Öğle arası | Lunch break | | yüksek |
| Etüt | Study hall | Vekil atanmayan ders: "Study hall / free study". | orta |
| Kulüp saati | Club period | | yüksek |
| Haftalık ders saati | Weekly hours | "Weekly lesson hours" uzun biçim. | yüksek |
| Müfredat | Curriculum | | yüksek |
| Haftalık ders çizelgesi (MEB) | MEB weekly hours chart | | orta |
| Müfredat sürümü | Curriculum version | | yüksek |
| MEB kaynak belgesi | MEB source document | | yüksek |
| Seçmeli ders | Elective | | yüksek |
| Zorunlu ders | Required subject | | yüksek |
| Branş | Subject area | UI `subjectArea`; "branch" yasak. Branş **adları** çevrilmez. | yüksek |
| Görevlendirme | Teaching assignment | UI `teacherAssignment`. Tek başına "Assignment" yasak (ödevle karışır). | yüksek |
| Yetkinlik ataması | Qualification | Öğretmenin okutabileceği ders. | orta |
| Öğretmen müsaitliği | Teacher availability | | yüksek |
| Kapasite (öğretmen, saat/hafta) | Teaching load (hours/week) | | orta |
| Aylık karşılık | Base hours | Maaş karşılığı ders saati. | orta |
| Ek ders | Additional paid hours | Türk ücret sistemine özgü; açıklamalı (B). | orta |
| Öğretmen izni | Teacher leave | UI `teacherLeave`. | yüksek |
| Raporlu / İzinli / Görevli (öğretmen izni türü) | Sick leave / On leave / On official duty | `medical` / `permitted` / `onDuty`. | orta |

---

## 7. Nöbet ve vekâlet

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Nöbet | Duty | UI `duty`. Açıklamalı (B): yardım metninde "supervising an area during breaks". | yüksek |
| Nöbet & Vekâlet (modül) | Duty & Substitution | | orta |
| Nöbet çizelgesi | Duty roster | | yüksek |
| Nöbet bölgesi / Nöbet yeri | Duty area | Kod `DutyLocation`. | yüksek |
| Nöbetçi | On duty | "Nöbetçi öğretmen" için §3. | yüksek |
| Nöbetlerim | My Duties | | yüksek |
| Nöbet defteri | Duty log | | yüksek |
| Nöbet muafiyeti | Duty exemption | | yüksek |
| Sürekli / Geçici (muafiyet) | Permanent / Temporary | | yüksek |
| Nöbet politikası | Duty policy | | yüksek |
| Haftalık nöbet sıklığı | Weekly duty frequency | | yüksek |
| Yancı | Backup duty | Nöbetçi öğle yemeğindeyken bölgeye bakan öğretmen (2026-10-26 kararı: "Lunch relief" bırakıldı). Bkz. §17 S-14. | orta |
| Yancılık max süresi | Maximum backup duty time | Seed `DUTY_MAX_ASSISTANT_MINUTES`. | orta |
| Adil otomatik dağıt | Fair auto-assign | | orta |
| Yük & Adalet | Workload & Fairness | | orta |
| Vekâlet | Substitution | Kod `Substitution`. UK "cover". Bkz. §17 S-13. | orta |
| Vekâlet görevi (bildirim) | Substitution assignment | Bildirim `SUBSTITUTION_ASSIGNED`. | yüksek |
| Vekil atandı | Substitute assigned | | yüksek |
| Boş ders | Uncovered lesson | | orta |
| Yürürlük başlangıcı | Effective from | | yüksek |
| Yürürlükte | In effect | | yüksek |

### 7.1 Nöbet yeri şablonları (seed `DutyLocationTemplateSeedData`)

| Türkçe | İngilizce | Açıklama (seed) → en | Güven |
|---|---|---|---|
| Kapı | Entrance | "Okul giriş kapısı" → "School entrance" | yüksek |
| Koridor | Hallway | "Kat koridoru" → "Floor hallway". US "hallway" (UK "corridor"). | yüksek |
| Kantin | Canteen | "Kantin önü" → "In front of the canteen". Bilinçli istisna (§1.1); bkz. §17 S-15. | orta |
| Spor Salonu | Gym | "Spor salonu girişi" → "Gym entrance" | yüksek |
| Bahçe | Schoolyard | "Dış bahçe" → "Outdoor schoolyard". "Garden" yanlış (bahçe = okul avlusu). | yüksek |
| Merdiven | Stairwell | "Merdiven boşluğu" → "Stairwell" | yüksek |

---

## 8. Ödevler

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Ödev (modül/kavram) | Homework | UI `homework`; "assignment" tek başına yasak. Sayılamaz — bkz. §1.5. | yüksek |
| Bir ödev (tekil kayıt) | Homework assignment | "Assignment" yalnız "homework assignment" içinde. | yüksek |
| Ödev ver | Assign homework | | yüksek |
| Yeni ödev verildi | New homework assigned | Bildirim `HOMEWORK_PUBLISHED`. | yüksek |
| Son teslim (tarihi) | Due date | | yüksek |
| Son teslim yaklaşıyor | Homework due soon | | yüksek |
| Teslim / Ödev teslimi | Submission | | yüksek |
| Ödev gönder / Ödev gönderildi | Submit homework / Homework submitted | | yüksek |
| Eksik ödev | Missing homework | | yüksek |
| Eksik ödev bildirimi | Missing homework notice | | yüksek |
| Ödev kapandı ve eksik notu | Homework closed: missing items noted | Bildirim `HOMEWORK_REVIEWED`. | orta |
| Ödev kontrolü / Takip ızgarası | Homework check / Tracking grid | | orta |
| Yapıldı / Yapılmadı / Eksik | Done / Not done / Incomplete | | orta |
| Ödev tamamlama oranı | Homework completion rate | | yüksek |
| Ödev panosu | Homework dashboard | | yüksek |
| Ödev politikası | Homework policy | | yüksek |

---

## 9. Kulüpler

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Kulüp | Club | | yüksek |
| Kulüp danışmanı | Club advisor | | yüksek |
| Başkan / Üye | President / Member | | yüksek |
| Kulüp başvurusu | Club application | | yüksek |
| Kulüp başvuru sonucu | Club application result | | yüksek |
| Kulüp etkinliği | Club activity | | yüksek |
| Kulüp duyurusu | Club announcement | | yüksek |
| Katıldı / Katılmadı / İşaretlenmedi | Attended / Did not attend / Not marked | | yüksek |
| Kulüp tercihi | Club preference | Öğrencinin sıralı seçimi. "1. tercih" → "1st choice". | orta |
| Tercih dönemi | Preference period | | orta |
| Tercihe açık / kapalı | Open for preferences / Closed for preferences | | orta |
| Yerleştirme | Placement | | yüksek |
| Yerleşmedi | Not placed | | yüksek |
| Tercih dışı | Outside preferences | | orta |
| Kontenjan | Capacity | | yüksek |
| Kontenjanı aşan | Over capacity | | yüksek |
| Mevcut üye | Existing member | | yüksek |
| Elle düzenlenen | Manually adjusted | | yüksek |
| Kulüp Üye Kabul Sorumlusu (ayar) | Club membership approval by | Değerler "Teacher" / "Administration". Bkz. §17 S-20. | düşük |
| Askıda (kulüp) | Paused | | orta |
| Kulübe yerleştirme özeti | Club placement summary | | yüksek |
| Kulüp kategorileri: Bilim, Teknoloji, Sanat, Müzik, Spor, Kültür, Sosyal Sorumluluk, Girişimcilik, Yabancı Dil, Diğer | Science, Technology, Art, Music, Sports, Culture, Social Responsibility, Entrepreneurship, Foreign Languages, Other | Enum etiketleri (`packages/core/src/club/constants.ts`). | yüksek |

---

## 10. Duyuru, bildirim, veli toplantısı, iletişim

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Duyuru | Announcement | | yüksek |
| Acil duyuru | Urgent announcement | | yüksek |
| Duyuru şablonu | Announcement template | | yüksek |
| Geri çek | Withdraw | "Delete" değil. | yüksek |
| Sessiz düzeltme | Silent edit | | orta |
| Hedef kitle | Audience | | yüksek |
| Okul geneli / Şube kapsamı | School-wide / Class-specific | | orta |
| Moderasyon (serbest / eşikli) | Moderation (open / approval required) | | orta |
| Gelen kutusu | Inbox | | yüksek |
| Gönderim kanalı | Delivery channel | Uygulama içi / Push / E-posta → In-app / Push / Email | yüksek |
| Gönderim raporu | Delivery report | | yüksek |
| Bildirim | Notification | | yüksek |
| Bildirim ayarları | Notification settings | | yüksek |
| Sessiz saat | Quiet hours | | yüksek |
| Mesaj | Message | | yüksek |
| Veli toplantısı | Parent-teacher meeting | UI `parentMeeting`. "Conference" bire bir görüşmeyi çağrıştırır; OKSİS'teki toplantı şube toplu. | orta |
| Şube toplantısı | Class meeting | Şube = Class kararına bağlı. | orta |
| Katılacağım / Katılamayacağım / Yanıt bekleniyor | I'll attend / I can't attend / Awaiting response | | yüksek |
| Katılamama nedeni | Reason for not attending | | yüksek |
| Fiili katılım | Actual attendance | Devamsızlık değil. | orta |
| Veli adına yanıt | Response on behalf of parent | | yüksek |
| Görüşme notu | Meeting note | | yüksek |
| Hesapsız veli | Parent without an account | | yüksek |
| Bugün veli toplantısı / Veli toplantısı başlıyor | Parent-teacher meeting today / starting now | | yüksek |
| Toplantı yanıtı bekleniyor | Meeting response pending | | yüksek |
| Veli toplantısı rehber öğretmen bilgisi | Parent-teacher meeting: homeroom teacher assigned | Bildirim `PARENT_MEETING_ASSIGNED`. | orta |
| Bekleyen işler | To-do | UI `pendingTask`. | orta |
| Onay bekleyen | Awaiting approval | | yüksek |
| Anasayfa | Home | | yüksek |

---

## 11. Öğrenci kaydı, belgeler, kimlik

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Kayıt (öğrenci) | Enrollment | US yazımı (tek "l"). | yüksek |
| Kayıt yenileme | Re-enrollment | | yüksek |
| Terfi (sezon geçişinde) | Promotion | | yüksek |
| Nakil gelen / Nakil giden | Transfer in / Transfer out | | yüksek |
| Nakil | Transfer | | yüksek |
| Mezun et / Mezun | Graduate / Graduated | | yüksek |
| Askıya al | Suspend (account) | Disiplindeki "uzaklaştırma" ile karışmaması için ekranda "Suspend account". | orta |
| Arşivle / Anonimleştir | Archive / Anonymize | | yüksek |
| Öğrenci numarası / Okul numarası | Student number | | yüksek |
| Sicil no | Staff ID | | orta |
| T.C. Kimlik No | Turkish ID number (T.C. Kimlik No) | Strateji C. | yüksek |
| Yabancı uyruklu | Foreign national | | yüksek |
| Kayıt evrakı | Enrollment document | | yüksek |
| Öğrenci belgesi | Student document | | orta |
| Veli-öğrenci ilişkisi / Yakınlık | Relationship to student | Anne/Baba/Vasi → Mother/Father/Legal guardian | yüksek |
| Rıza / Açık rıza | Consent / Explicit consent | | yüksek |
| Aydınlatma metni | Privacy notice | KVKK bağlamında. | yüksek |
| Giriş / Çıkış | Sign in / Sign out | | yüksek |
| Şifre / Parola | Password | | yüksek |
| Şifre sıfırlama | Reset password | | yüksek |
| Davet bağlantısı | Invitation link | | yüksek |

---

## 12. Disiplin ve ödül (modül henüz yok — ileri tarihli)

> OKSİS'te disiplin modülü kodda yok; terimler gelecekteki metinler ve karne/ayar ekranlarında geçen göndermeler için verilmiştir. Yaptırım adlarının yönetmelik maddeleri **doğrulanmadı**.

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Disiplin | Discipline | | yüksek |
| Okul öğrenci ödül ve disiplin kurulu | Student Awards and Discipline Board | | orta |
| Disiplin cezası | Disciplinary action | | yüksek |
| Kınama | Reprimand | | orta |
| Okuldan kısa süreli uzaklaştırma | Short-term suspension | | orta |
| Okul değiştirme | Transfer to another school (disciplinary) | | orta |
| Örgün eğitim dışına çıkarma | Expulsion from formal education | | orta |
| Davranış notu / Davranış puanı | Conduct grade | | orta |
| Ödül | Award | | yüksek |

---

## 13. Kurumlar, sistemler ve kısaltmalar

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| MEB | MEB (Ministry of National Education) | Strateji C; açılım ilk geçişte. | yüksek |
| Millî Eğitim Bakanlığı | Ministry of National Education | Uzun ad geçerse. | yüksek |
| İl / İlçe Millî Eğitim Müdürlüğü | Provincial / District Directorate of National Education | | yüksek |
| e-Okul | e-Okul | MEB öğrenci bilgi sistemi; açılım "(MEB student information system)". OKSİS e-Okul'a bağlanmaz. | yüksek |
| MEBBİS | MEBBİS | Açılım "(MEB staff information system)". | yüksek |
| OYSS (Ortak Yazılı Sınav Sistemi) | OYSS (MEB Common Exam System) | ADR 0028. Açılımda "written" yerine "exam" (UI kuralı); "puanlar e-Okul'da" → "Scores are in e-Okul". | orta |
| KVKK | KVKK (Turkish Personal Data Protection Law) | | yüksek |
| LGS / YKS | LGS / YKS | Merkezi lise ve üniversite giriş sınavları; açılım "(high school / university entrance exam)". | yüksek |
| RAM (Rehberlik ve Araştırma Merkezi) | RAM (Guidance and Research Center) | | orta |
| BİLSEM | BİLSEM (Science and Art Center for gifted students) | | orta |
| T.C. | — | Kimlik no içinde korunur; tek başına "Republic of Türkiye". Ülke adı "Türkiye" (resmî İngilizce ad, 2022). | yüksek |

---

## 14. Resmî tatil adları (seed `OfficialHolidaySeedData`)

Türk devletinin kullandığı resmî İngilizce adlar esas alınır; bunlar özel isim sayılır, US yazım kuralı uygulanmaz (*Labour*). Dini bayram tarihleri seed'de taslaktır (K-12 §A5) — çeviriyi etkilemez.

| Türkçe | İngilizce | Not / gerekçe | Güven |
|---|---|---|---|
| Yılbaşı | New Year's Day | | yüksek |
| Ulusal Egemenlik ve Çocuk Bayramı | National Sovereignty and Children's Day | | yüksek |
| Emek ve Dayanışma Günü | Labour and Solidarity Day | Resmî ad; US "Labor" kullanılmaz. | orta |
| Atatürk'ü Anma, Gençlik ve Spor Bayramı | Commemoration of Atatürk, Youth and Sports Day | | yüksek |
| Demokrasi ve Millî Birlik Günü | Democracy and National Unity Day | | yüksek |
| Zafer Bayramı | Victory Day | | yüksek |
| Cumhuriyet Bayramı | Republic Day | | yüksek |
| Ramazan Bayramı | Ramadan Feast (Eid al-Fitr) | Bkz. §17 S-18. | orta |
| Kurban Bayramı | Feast of the Sacrifice (Eid al-Adha) | Bkz. §17 S-18. | orta |
| Ramazan Bayramı Arifesi | Ramadan Feast Eve | Yarım gün. | orta |
| Kurban Bayramı Arifesi | Feast of the Sacrifice Eve | Yarım gün. | orta |

---

## 15. Referans verisi ve ortak arayüz sözcükleri

### 15.1 Yabancı dil adları (seed `ForeignLanguageSeedData`)

**Karar (Faruk, 2026-10-23): yabancı dil adları ÇEVRİLMEZ**, aşağıdaki tablo uygulanmaz (bkz. §16). Kayıt için eski not: ADR 0029 yabancı dil adlarını çevrilir listesine koyuyordu. **Çelişki:** seed açıklaması "ad, dilin MEB öğretmenlik branşının adıyla birebir aynıdır" diyor; branş adları ise çevrilmez. Aynı ekranda branş "İngilizce", dil "English" görünebilir. Bkz. §17 S-21.

| Türkçe | İngilizce | Güven |
|---|---|---|
| Almanca | German | yüksek |
| Arapça | Arabic | yüksek |
| Bulgarca | Bulgarian | yüksek |
| Çince | Chinese | yüksek |
| Farsça | Persian | yüksek |
| Fransızca | French | yüksek |
| İngilizce | English | yüksek |
| İspanyolca | Spanish | yüksek |
| İtalyanca | Italian | yüksek |
| Japonca | Japanese | yüksek |
| Korece | Korean | yüksek |
| Rusça | Russian | yüksek |
| Ukraynaca | Ukrainian | yüksek |
| Urduca | Urdu | yüksek |

Not: sıralama Türkçe alfabetiktir (`DisplayOrder`); İngilizce arayüzde ada göre sıralanırsa sıra değişir — çeviri değil, gösterim kararı.

### 15.2 Bildirim türleri (seed `NotificationEventTypeSeedData`) — kalan satırlar

Yukarıdaki bölümlerde geçmeyenler:

| Türkçe | İngilizce | Güven |
|---|---|---|
| Yoklama düzeltme talebi | Attendance correction request | yüksek |
| Yoklama yeniden açma talebi | Attendance reopen request | yüksek |
| Sınav takvimi yayınlandı | Exam schedule published | yüksek |
| Sınav için ders saati isteği | Period request for an exam | orta |
| Saat isteği sonuçlandı | Period request answered | orta |
| Sınav planına görüş bırakıldı | Comment added to exam plan | yüksek |
| Sınav planı görüşü ele alındı | Exam plan comment resolved | yüksek |
| Taksit hatırlatması | Installment reminder | yüksek |
| Ödeme alındı | Payment received | yüksek |
| Yeni duyuru | New announcement | yüksek |
| Kulüp etkinliği yayınlandı / iptal edildi | Club activity published / canceled | yüksek |
| Kulüp duyurusu yayınlandı | Club announcement published | yüksek |
| Kulüp tercih dönemi açıldı / kapandı | Club preference period opened / closed | yüksek |
| Kulüp tercihi hatırlatması | Club preference reminder | yüksek |
| Kulüp yerleştirme sonucu | Club placement result | yüksek |
| Kulüp yerleştirmesi tamamlanmadı | Club placement: not placed | orta |
| Yemek menüsü yayınlandı / değişti | Meal menu published / changed | yüksek |
| Veli toplantısı yayınlandı / değişti / iptal edildi | Parent-teacher meeting published / changed / canceled | yüksek |
| Bildirim grupları: Devamsızlık, Akademik, Ödeme, Duyuru, Ödev, Kulüp, Yemekhane, Veli Toplantısı | Attendance, Academics, Payments, Announcements, Homework, Clubs, Cafeteria, Parent-Teacher Meetings | yüksek |

### 15.3 Sistem ayarı etiketleri (seed `SystemSettingSeedData`)

| Türkçe | İngilizce | Güven |
|---|---|---|
| Platform varsayılan dili | Platform default language | yüksek |
| Varsayılan zaman dilimi | Default time zone | yüksek |
| Şube başına max öğrenci | Maximum students per class | orta |
| Min şifre uzunluğu | Minimum password length | yüksek |
| Access / Refresh token süresi | Access / Refresh token lifetime | yüksek |
| Davet linki geçerlilik süresi | Invitation link validity | yüksek |
| Max dosya yükleme boyutu | Maximum upload size | yüksek |
| Okul başına varsayılan depolama kotası | Default storage quota per school | yüksek |

### 15.4 Yemekhane

| Türkçe | İngilizce | Not | Güven |
|---|---|---|---|
| Yemekhane | Cafeteria | Kantinden ayrı (§7.1). | yüksek |
| Yemek menüsü | Meal menu | | yüksek |
| Öğün | Meal | | yüksek |
| Kahvaltı / Öğle Yemeği / İkindi Kahvaltısı | Breakfast / Lunch / Afternoon snack | | yüksek |
| Yemek / Yemek kataloğu | Dish / Dish catalog | US "catalog". | yüksek |
| Menü grubu | Menu group | | yüksek |
| Alerjen | Allergen | 14 kod AB listesi: Gluten, Crustaceans, Eggs, Fish, Peanuts, Soybeans, Milk, Tree nuts, Celery, Mustard, Sesame, Sulfites, Lupin, Mollusks (US yazımı). | yüksek |

### 15.5 İzin (yetki) açıklamaları — fiil sözlüğü

İzin seed'indeki ~120 açıklama bu fiil eşlemesiyle çevrilir: görüntüle → View; düzenle → Edit; güncelle → Update; ekle / oluştur → Add / Create; sil → Delete; yönet → Manage; yayınla → Publish; geri çek → Withdraw; onayla / reddet → Approve / Reject; ata → Assign; dışa aktar → Export; Excel'e aktar → Export to Excel; Excel'den aktar → Import from Excel; arşivle → Archive; kilitle → Lock; yükle (dosya) → Upload; indir → Download. Teknik parantezler ("ABAC broad scope", "legacy", "SuperAdmin-only") ya kaldırılır ya da olduğu gibi kalır; çevrilmez.

### 15.6 Ortak durumlar ve eylemler

| Türkçe | İngilizce | Güven |
|---|---|---|
| Taslak | Draft | yüksek |
| Yayında / Yayınlandı | Published | yüksek |
| Aktif / Pasif | Active / Inactive | yüksek |
| Arşiv / Arşivlendi | Archived | yüksek |
| Bekliyor / Onaylandı / Reddedildi | Pending / Approved / Rejected | yüksek |
| İptal / İptal edildi | Canceled | yüksek |
| Kaydet / Vazgeç / Sil / Düzenle | Save / Cancel / Delete / Edit | yüksek |
| Geri al | Undo | yüksek |
| Dışa aktar / İçe aktar | Export / Import | yüksek |
| Yeniden dene | Try again | yüksek |
| Yakında | Coming soon | yüksek |
| Tümü / Diğer | All / Other | yüksek |
| Bugün / Yarın / Bu hafta | Today / Tomorrow / This week | yüksek |
| Hafta sonu | Weekend | yüksek |
| Başlangıç / Bitiş | Start / End | yüksek |
| İsteğe bağlı (opsiyonel) | Optional | yüksek |
| Gerekçe / Sebep | Reason | yüksek |
| Dosya / Ek | File / Attachment | yüksek |
| Depolama kotası | Storage quota | yüksek |
| Rapor (modül) | Reports | yüksek |
| Finans / Cari hesap / Ücretlendirme / Taksit / İndirim / Burs | Finance / Account statement / Tuition / Installment / Discount / Scholarship | orta |

---

## 16. Çevrilmez

Bu kalemler İngilizce arayüzde de Türkçe kaynak biçimiyle görünür; sözlüğe çeviri olarak eklenmez, Tolgee glossary'sinde "do not translate" işaretiyle tutulur.

| Kalem | Gerekçe / kaynak |
|---|---|
| **Ders adları** (Matematik, Türk Dili ve Edebiyatı, Fizik, Din Kültürü ve Ahlak Bilgisi, İngilizce …) | ADR 0029: özel isim; "Matematik" İngilizce arayüzde de "Matematik". `NameKey` almaz. |
| **Ders kataloğu** (MEB ders listesi, müfredat dersleri, okulun özel dersleri) | ADR 0029. |
| **Branş adları** (`SubjectBranch`: Matematik, Sınıf Öğretmenliği, Rehberlik ve Psikolojik Danışmanlık …) | ADR 0029: dersle aynı ad uzayı. "Branş" sözcüğünün kendisi çevrilir (Subject area). |
| **Yabancı dil adları** (`ForeignLanguage`: İngilizce, Almanca, Arapça …) | Faruk, 2026-10-23 (S-21): branş/ders adlarıyla aynı ad uzayı. `NameKey` almaz. |
| **Eğitim programı adları** (`EducationProgram`: Anadolu Lisesi …) | Faruk, 2026-10-23: ders kataloğu niteliğinde. |
| İl, ilçe, mahalle adları | ADR 0029. |
| Okul adları, şube adları (9-A), kişi adları | Kullanıcı içeriği / özel isim. |
| Okulun eklediği ya da adını değiştirdiği katalog kayıtları (`NameKey` boş) | ADR 0029. |
| Kullanıcının yazdığı her içerik (duyuru, ödev, not, açıklama, görüşme notu, duyuru şablonu) | ADR 0029. |
| Kalıcı kodlar: sınav türü `VZ1`, `SZ`, `PER`, `PRJ`; rol kodları; izin kodları; olay anahtarları (`ATT_THRESHOLD` …) | Anahtar, görünen metin değil. |
| Kısaltmalar: MEB, e-Okul, MEBBİS, OYSS, KVKK, LGS, YKS, RAM, BİLSEM, T.C. | Korunur; açılım §13. |
| Atatürk | Özel isim. |
| OKSİS (ürün adı) | Marka. |

---

## 17. Faruk'un onayına sunulan belirsiz terimler

**Karar (Faruk, 2026-10-23):** S-1…S-20'de kalın yazılan öneriler aynen kabul edildi; bu terimler bağlayıcıdır ve tablolardaki güven derecelerinden bağımsız olarak "yüksek" sayılır. S-21: **yabancı dil adları çevrilmez** (dersler/branşlarla tutarlılık; §15.1 tablosu yalnız bilgi amaçlıdır, uygulanmaz).

| # | Türkçe | Seçenekler | Öneri ve gerekçe |
|---|---|---|---|
| S-1 | Varyant ve biçim | (a) **en-US yazım + gün-önce tarih, 24 saat** · (b) tam en-US (`10/23/2026`, `2:30 PM`) · (c) en-GB | (a): sınıf numarası eşleşmesi US'yi zorunlu kılıyor (§1.1); Türkiye'de yaşayan kullanıcı ay-önce tarihi yanlış okur. |
| S-2 | Şube | **Class** ("Class 9-A") · Section · Homeroom | "Class" yabancı veli için en doğal. Risk: "class" gündelik dilde "ders" de demek — bu yüzden ders = Lesson/Subject, derslik = Room sabitlendi. "Section" kod adına yakın ama okul dışı kullanıcıya yabancı; "Homeroom" ABD'ye özgü ve ilkokulda tuhaf. |
| S-3 | Not | **Grade** (menü "Grades") · Mark · Score | US'de doğal olan "Grade"; ama sınıf seviyesi de "Grade 9". Ayrım kuralı: sınıf seviyesi **her zaman sayıyla** ("Grade 9") ya da "grade level"; not tek başına "grade(s)". Alternatif "Mark" kodla (`Mark`) uyumlu ama US'de İngiliz kokar. |
| S-4 | Dönem | **Semester** · Term | Yılda iki dönem = semester. "Term" İngiltere'de üç dönemi, ABD'de belirsiz süreyi çağrıştırır. |
| S-5 | Sezon | **Academic year** · School year · (Season — red) | İkisi de doğru; "Academic year" kodun eski adıyla (`AcademicYear`) ve uluslararası okullarla uyumlu. "Season" kesin reddedilmeli. |
| S-6 | Ders programı | **Class schedule** · Timetable | US tutarlılığı "schedule"; ama "Timetable" kodda (`Timetable`) ve uluslararası okullarda yaygın, "exam schedule" ile de karışmaz. Faruk "Timetable" derse tek istisna olarak yazılır. |
| S-7 | Kademe adları | İlkokul: **Primary school** · Elementary school. Anaokulu: **Preschool** · Kindergarten | "Primary" uluslararası programlarda yerleşik, US okuru da anlar. "Kindergarten" ABD'de yalnız 5–6 yaş sınıfıdır; MEB anaokulu 3–5 yaşı kapsar → "Preschool". |
| S-8 | Takdir / Teşekkür belgesi | **High Honors / Honors certificate** · Certificate of Excellence / Certificate of Achievement · Türkçeyi koru ("Takdir (High Honors)") | US "honor roll" mantığıyla birebir; iki kademeli sıra korunur. Türkçeyi korumak yabancı veliye hiçbir şey söylemez. |
| S-9 | Onur belgesi | **Certificate of Merit** · Good Conduct Certificate · Certificate of Honor | "Certificate of Honor" Takdir/Teşekkür'le karışır. Belgenin dayanağı (davranış, sınıf arkadaşlarının oyu) MEB metninden **doğrulanmadı**; doğrulanınca "Good Conduct" daha isabetli olabilir. |
| S-10 | İzinli / Raporlu (yoklama) | (a) **Excused / Medical** · (b) Excused – Leave / Excused – Medical · (c) Authorized / Sick | MEB'de ikisi de özürlü devamsızlık. (a) kısa ve rozete sığar; (b) bu gerçeği gösterir ama uzun. Kısa kodlar L / E / M. |
| S-11 | Gözetmen | **Proctor** · Invigilator | US varyantı "Proctor"; kod `Invigilator`. UK okullu öğretmen "invigilator"a alışık olabilir. |
| S-12 | Kelebek sınav sistemi | **Mixed seating** · Butterfly seating · "Kelebek" korunur | "Butterfly" Türkçe deyimin harfi harfine çevirisi, yabancıya anlam taşımaz. "Mixed seating" işlevi söyler. |
| S-13 | Vekâlet | **Substitution** · Cover | Kod ve US kullanımı "substitution/substitute". "Cover" UK okul jargonu. |
| S-14 | Yancı | **Backup duty** (karar 2026-10-26) · Lunch relief · Relief duty · Duty assistant | Kod `reliever`. Türk okuluna özgü; işlev "nöbetçi yemekteyken bölgeyi tutmak". |
| S-15 | Kantin | **Canteen** (duty `canteen` anahtarı tr "Kantin", 2026-10-26) · Snack bar · School store | "Cafeteria" yemekhaneye ayrıldı. "Canteen" İngiliz kullanımı ama Türkiye'deki İngilizcede yerleşik. |
| S-16 | Rehber öğretmen | **School counselor** · Guidance counselor | Güncel US terimi "school counselor". Rol henüz yok. |
| S-17 | Veli | **Parent** (yasal metinde "parent or guardian") · Parent/Guardian (her yerde) · Guardian | Her yerde "Parent/Guardian" arayüzü kalabalıklaştırır; rol adı kodla aynı "Parent". |
| S-18 | Ramazan / Kurban Bayramı | **Ramadan Feast (Eid al-Fitr) / Feast of the Sacrifice (Eid al-Adha)** · yalnız "Eid al-Fitr / Eid al-Adha" · "Ramazan Bayramı" korunur | Türk resmî İngilizcesi "Ramadan Feast / Feast of the Sacrifice"; parantez Müslüman olmayan ya da Arap ülkelerinden gelen kullanıcıya köprü. |
| S-19 | Okul Yöneticisi (rol) | **School Administrator** · Principal · School Admin | Rol müdür dışında birine de verilebiliyor; "Principal" unvan, rol değil. |
| S-20 | Kulüp Üye Kabul Sorumlusu | **"Club membership approval by: Teacher / Administration"** · Club Admissions Officer · Membership approver | Ayar bir kişi değil, bir yetki modu (öğretmen ya da idare); ad bunu söylemeli. |
| S-21 | "İngilizce" — dil adı mı, branş/ders adı mı? | (a) **Dil listesi çevrilir (English), ders/branş "İngilizce" kalır** · (b) yabancı dil listesi de çevrilmez | ADR 0029 (a)'yı söylüyor; aynı ekranda iki farklı ad görünecek. Faruk'un bilmesi gereken bir tutarsızlık, çeviri hatası değil. |

### 17.1 Göç 3 terim denetimi eklemeleri (2026-10-26)

Göç 3 en denetimi (oksis-education-expert) sonrası eklenen/kesinleşen terimler. Faruk kararları: S2 salon = Assembly hall, S3 Konferans Salonu = Auditorium, S4 Kantin = Canteen, S5 "Devamsızlık Sınırı Aşımı Resmî Yazısı" = Absence Limit Official Letter (tr'de "Resmi" yerine "Resmî"), S7 kulüp yoklaması = Club Activity Attendance.

| Türkçe | İngilizce |
|---|---|
| Yancı | Backup duty |
| Sorumlu öğretmen | Teacher in charge |
| Sayım (etkinlik güvenlik sayımı; "Tur" kullanılmaz, tr "Sayım anı") | Head count (üst başlık "Activity head count") |
| Etkinlik yoklaması | Activity roll call |
| Devam karnesi | Attendance record |
| Yoklama dökümü | Attendance roster |
| Hızlı sorgu | Quick lookup |
| Beklenen öğretmen | Scheduled teacher |
| Düzeltilmiş | Corrected |
| Mazeret bildirimi | Excuse request |
| Mazeret kaydı | Excuse record |
| Geçersiz | Voided |
| Eşiğe uzaklık | Distance to limit |
| Katılacak / Katılmayacak / Yanıt yok | Attending / Not attending / No response |
| Gelmedi (veli toplantısı) | Absent |
| Davet düştü | No longer invited |
| Not denetim kaydı | Note audit log |
| Katılım ızgarası | Attendance grid |
| Başvuru + onay | Application + approval |
| Derslik türleri (Spor salonu / Konferans salonu) | Gym / Auditorium |
| Açık alan | Outdoor area |
| Ödev işlemleri | Homework actions |
| Ödev tamamlama raporu | Homework completion report |
| Okul ortalaması | School average |
| Geç | Late |
| Görsel / Belge / Bağlantı | Image / Document / Link |
| Nöbet yönetimi | Duty management |
| Koridor / Kapı (nöbet yeri türü) | Hallway / Entrance |
| Hastalık izni / Görev değişikliği (devir nedeni) | Sick leave / Role change |

---

## 18. Belge–kod çelişkileri (bu çalışmada bulundu)

1. `teknik/kurallar/ortak/naming-conventions.md` §1 sınıf seviyesi için `Grade`, sezon için `AcademicYear`, dönem için "Güz / Bahar" diyor; kod `GradeLevel`, `AcademicSession` ve "1. Dönem / 2. Dönem" (T1/T2) kullanıyor. Kod kazanır; naming-conventions tablosu güncellenmeli.
2. [[Sınıf Seviyesi]] notunun `aliases` alanında "Kademe" var; `oksis-ui/CLAUDE.md` sözlüğü "Kademe"yi `schoolStage` (Anaokulu/İlkokul/Ortaokul/Lise) olarak tanımlıyor. Bu sözlük UI tanımını izledi; domain notu düzeltilmeli.
3. `ForeignLanguageSeedData` açıklaması dil adını branş adıyla "birebir aynı" sayıyor; ADR 0029 biri çevrilir öteki çevrilmez diyor (S-21).
4. `oksis-ui/CLAUDE.md`: "UI-facing strings are **Turkish**" ve web `CLAUDE.md`: `<html lang="tr">` — ADR 0029 zemin aşamasında güncellenmesi gereken kurallar (bu belgenin işi değil, not olarak).
