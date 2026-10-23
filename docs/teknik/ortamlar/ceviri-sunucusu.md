# Çeviri sunucusu (Tolgee)

> [!info] Belge bilgisi
> **Amaç:** OKSİS çeviri yönetim aracının (Tolgee, self-hosted, ücretsiz sürüm) Oracle test VM'indeki kurulumunu, işletimini, yedeğini ve güncellemesini kayda geçirmek.
> **Kurulum:** 2026-10-23. Karar: [[0029-cok-dil-altyapisi-ingilizce]]; plan: Dilim 0.3 ve D (sunucu kısmı).
> **Gizli bilgi:** Yalnız değişken **adları** yazılır; değer yok.

## Yer ve yapı

| Öğe | Değer |
|---|---|
| Sunucu | Oracle A1 test VM'i (test yığınıyla aynı makine) |
| Dizin | `/opt/oksis/ceviri/` (sahip: root) |
| Compose | `/opt/oksis/ceviri/docker-compose.yml`, proje adı `oksis-ceviri` (test yığınından **bağımsız**; `/opt/oksis/test` dosyalarına dokunulmaz) |
| Tolgee | `tolgee/tolgee:v3.238.1` (sabit etiket), `mem_limit` 1,5 GiB, `JAVA_OPTS=-Xms256m -Xmx512m` (Metaspace sınırı yok: 768 MiB'da OOM) |
| Veritabanı | `postgres:17.11-alpine`, `mem_limit` 256 MiB, yalnız iç ağda (`oksis-ceviri_ic`, `internal`), dışarı port yok; gömülü PG kapalı |
| Port | `127.0.0.1:8085` → Tolgee 8080 (yalnız VM içi) |
| Tünel erişimi | Tolgee, `oksis-test_default` ağına **external** olarak bağlıdır (diğer ad: `tolgee-ceviri`); `cloudflared` hedefi `http://tolgee-ceviri:8080` |
| Veri | Docker volume'ları `oksis-ceviri_pgdata`, `oksis-ceviri_tolgeedata` |
| Yeniden başlatma | `restart: unless-stopped`; sağlık kontrolü `GET /api/public/configuration` (açılış toleransı 180 sn; açılış ~50-60 sn) |
| Adres | `https://ceviri.oksis.net` (Cloudflare Tunnel `oksis-test` + Cloudflare Access `test-ekibi`) |

## Yapılandırma

Gizli değerler `/opt/oksis/ceviri/.env` dosyasındadır (root:root, `0600`), VM'de rastgele üretilmiştir. Değişken adları:

- `POSTGRES_PASSWORD`, `SPRING_DATASOURCE_PASSWORD` (aynı değer)
- `TOLGEE_AUTHENTICATION_INITIAL_PASSWORD` (yönetici parolası; kullanıcı adı `admin`)
- `TOLGEE_AUTHENTICATION_JWT_SECRET`

Gizli olmayanlar compose içinde: `TOLGEE_AUTHENTICATION_ENABLED=true`, `TOLGEE_AUTHENTICATION_REGISTRATIONS_ALLOWED=false` (kayıt kapalı), `TOLGEE_AUTHENTICATION_CREATE_DEMO_FOR_INITIAL_USER=false` (demo kullanıcı/proje yok), `TOLGEE_AUTHENTICATION_INITIAL_USERNAME`, `TOLGEE_POSTGRES_AUTOSTART_ENABLED=false`, `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `TOLGEE_FRONTEND_URL`, `TOLGEE_TELEMETRY_ENABLED=false`.

Not: `TOLGEE_AUTHENTICATION_INITIAL_PASSWORD` yalnız ilk kullanıcı oluşurken uygulanır. Parolayı sonradan Tolgee arayüzünden değiştirirsen `.env` değeri eskir; kaynak arayüzdür.

Yönetici parolasını okuma (VM'de, yalnız kendi terminalinde):

```bash
ssh -i ~/.ssh/oksis_test ubuntu@138.2.159.102 "sudo grep '^TOLGEE_AUTHENTICATION_INITIAL_PASSWORD=' /opt/oksis/ceviri/.env | cut -d= -f2-"
```

## Proje

"OKSİS" (id 1): kaynak dil `tr`, hedef `en`, `useNamespaces=true`, `icuPlaceholders=true`. Sözlük (glossary) ücretsiz sürümde yok; terim sözlüğü kaynağı [[i18n-terim-sozlugu]] kalır. Ücretsiz sınır: en çok 10 kullanıcı, `ee/` özelliği yok. CI için proje API anahtarı (PAK) henüz oluşturulmadı (Dilim D CI kısmı).

## İşletim

```bash
cd /opt/oksis/ceviri
sudo docker compose ps
sudo docker compose logs --tail 100 tolgee
sudo docker compose restart tolgee
```

## Yedek ve geri yükleme

- Günlük 03:30 UTC (±5 dk): systemd zamanlayıcısı `oksis-ceviri-yedek.timer` → `/opt/oksis/ceviri/yedekle.sh` → `/opt/oksis/ceviri/yedek/tolgee-<UTC zaman>.sql.gz` (`pg_dump`, gzip, bütünlük denetimli; 14 günden eskiler silinir).
- Elle yedek: `sudo systemctl start oksis-ceviri-yedek.service`; durum: `systemctl list-timers oksis-ceviri-yedek.timer`.
- **Geri yükleme denemesi (2026-10-23):** yedek geçici veritabanına yüklendi (1 kullanıcı, 1 proje "OKSİS", 2 dil); geçici veritabanı silindi.
- Gerçek geri yükleme (Tolgee durdurulup, boş veritabanına):

```bash
cd /opt/oksis/ceviri
sudo docker compose stop tolgee
sudo docker compose exec -T postgres psql -U tolgee -d postgres -c "drop database tolgee" -c "create database tolgee"
zcat yedek/<dosya>.sql.gz | sudo docker compose exec -T postgres psql -U tolgee -d tolgee -v ON_ERROR_STOP=1
sudo docker compose start tolgee
```

- Deneme için `-d geri_yukleme_deneme` gibi ayrı veritabanı kullan; `docker compose exec` komutlarına betik içinde `-T` ve (heredoc içinde) `</dev/null` ver.
- Yedekler aynı VM'dedir: VM kaybında yedek de gider. Çeviriler zaten depodaki JSON dosyalarıdır; harici kopya gerekirse Faruk'a sorulur.

## Güncelleme

1. Sürüm notunu oku (<https://github.com/tolgee/tolgee-platform/releases>); yeni etiket seç (`latest` kullanılmaz).
2. Yedek al: `sudo systemctl start oksis-ceviri-yedek.service`.
3. `docker-compose.yml` içinde `tolgee/tolgee:<etiket>` değiştir, sonra `sudo docker compose pull tolgee && sudo docker compose up -d tolgee`.
4. `curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:8085/api/public/configuration` → 200 (açılış ~1 dk).
5. Sorun olursa eski etiketi geri yaz; veritabanı göçü geri gitmediyse yedekten dön.
