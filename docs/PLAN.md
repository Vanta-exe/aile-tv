# Aile TV — Ayrıntılı Teknik Plan

> Sürüm: 1.0 · Tarih: 2026-10-25 · Hazırlayan: Claude (kullanıcıyla birlikte)
> Bu doküman, yarın (ve sonrasında) projeyi devralacak her ajan için **tek başına yeterli** olacak şekilde yazılmıştır.
> Kararların gerekçeleri: `DECISIONS.md` · Güncel durum: `PROGRESS.md` · Son oturum: `HANDOFF.md`

---

## 0. Kullanıcının İsteği (Orijinal Bağlam)

Kullanıcı, evdeki Android TV için annesi ve babasının kullanacağı özel bir canlı TV uygulaması istiyor.

**Sorun:** Uydu sisteminde frekanslar değişiyor, sinyal gidiyor, kanal listesi bozuluyor; anne-baba bu durumda TV'yi kullanamıyor.
**İstenen:** TV açılınca klasik uydu alıcısı gibi doğrudan kanal görmek, kumandayla zahmetsiz kanal değiştirmek.

Kullanıcının dört beklentisi (birebir korunmalı):
1. **Klasik TV hissi:** Açılır açılmaz menü/afiş/ayar göstermeden son izlenen kanal tam ekran.
2. **Kumanda:** Yukarı/aşağı (veya CH+/CH−) ile sonraki/önceki kanal. OK ile ekran kenarında büyük yazılı sade kanal listesi. Rakam tuşlarıyla kanal numarası.
3. **Bozulmazlık:** Geri tuşuna üst üste basılsa bile uygulamadan çıkılamaz, yabancı menülere düşülmez.
4. **Kesintisiz internet yayını:** ~20 popüler ulusal/haber kanalı (TRT 1, ATV, Kanal D, Show TV, Star, Now, TV8, Habertürk, NTV, CNN Türk vb.). Linkler/token'lar değişse de kullanıcı eve gidip güncelleme yapmak zorunda kalmamalı; en az bakım gerektiren akıllı bir sistem.

**Gerçekçi hedef (kullanıcıyla netleştirildi):** "Sıfır bakım" mümkün değil (kanallar linklerini değiştiriyor). Hedef:
**bozulma olunca sistem kullanıcıya haber verir, kullanıcı eve gitmeden telefondan dakikalar içinde düzeltir.**

---

## 1. Mimari Genel Bakış

```
┌──────────────────────── Android TV (evde) ─────────────────────────┐
│  Aile TV (Kotlin)                                                    │
│                                                                      │
│  MainActivity (HOME launcher, singleTask, Geri tuşu yutulur)          │
│    └─ PlayerScreen (Compose)                                         │
│         ├─ PlayerView (Media3)                                       │
│         ├─ ChannelOsd · ChannelListPanel · NumberEntryOverlay         │
│         ├─ StatusOverlay ("Yayın bekleniyor…")                        │
│         └─ AdminScreen (gizli, 0000)                                  │
│                                                                      │
│  TvViewModel ── PlayerController ── PlaybackWatchdog / RetryPolicy   │
│        │              │                                              │
│        │              └─ StreamResolver ── StepExecutor (OkHttp)      │
│        │                       └─ ResolvedUrlCache                    │
│        └─ ConfigRepository ── ConfigFetcher ── SignatureVerifier      │
│                 └─ SettingsStore (DataStore: son kanal, son konfig)   │
│  ConfigRefreshWorker (WorkManager, 3 saatte bir)                      │
│  ErrorReporter (opsiyonel: Telegram'a hata bildirimi)                 │
└───────────────────────────────▲─────────────────────────────────────┘
                                │ HTTPS GET dist/channels.json + .sig
┌───────────────────────────────┴──────── GitHub ─────────────────────┐
│  config/channels.json      ← kullanıcı telefondan düzenler          │
│  publish-config.yml        → doğrula + imzala → dist/ (raw + jsDelivr)│
│  healthcheck.yml (cron 2s) → tüm kanalları test et → Telegram uyarısı │
└─────────────────────────────────────────────────────────────────────┘
```

**Temel ilke:** Uygulama "aptal ama dayanıklı" bir oynatıcıdır; **değişebilecek her şey (kanal listesi, linkler, link bulma tarifleri) uzaktaki konfigdedir.**
APK güncellemesi sadece yeni özellik veya yeni tarif tipi gerektiğinde yapılır.

---

## 2. Teknoloji Yığını

| Katman | Seçim | Not |
|---|---|---|
| Dil | Kotlin (2.x, en güncel kararlı) | |
| Build | Gradle Kotlin DSL + Version Catalog (`gradle/libs.versions.toml`) | Sürümler kurulum günü güncel kararlıya sabitlenir |
| minSdk / targetSdk | minSdk 21 (geçici, TV modeline göre yükseltilebilir) / target en güncel | TV modeli öğrenilince ADR ile kesinleşecek |
| UI | Jetpack Compose + `androidx.tv:tv-material` | Leanback kullanılmıyor |
| Oynatıcı | `androidx.media3:media3-exoplayer`, `-exoplayer-hls`, `-ui`, `-datasource-okhttp` | |
| HTTP | OkHttp | Resolver ve ExoPlayer aynı client'ı paylaşır (çerez/UA tutarlılığı) |
| JSON | kotlinx.serialization | |
| İmza doğrulama | Ed25519 — `net.i2p.crypto:eddsa` veya BouncyCastle (minSdk 21'de yerleşik Ed25519 yok) | ADR-007 |
| Kalıcı ayar | Jetpack DataStore (Preferences) | |
| Arka plan | WorkManager | |
| DI | Yok (elle basit `AppContainer`) | Küçük proje, Hilt gereksiz karmaşıklık |
| Test | JUnit + kotlinx-coroutines-test + MockWebServer (OkHttp) | |
| Araçlar (tools/) | Python 3 (`pynacl`, `requests`) | İmzalama + sağlık kontrolü |
| CI | GitHub Actions | |

**Paket adı:** `com.aile.tv` · **Uygulama adı:** "Aile TV" (kullanıcı değiştirebilir).

---

## 3. Repo Yapısı

```
tv/
├─ AGENTS.md                 # ortak hafıza giriş noktası
├─ CLAUDE.md                 # @AGENTS.md içe aktarır
├─ docs/                     # PLAN, DECISIONS, PROGRESS, HANDOFF, CHANNELS, SETUP_TV (F6'da)
├─ app/                      # Android uygulaması
│  └─ src/main/java/com/aile/tv/
│     ├─ AileTvApp.kt            # Application, AppContainer kurulumu
│     ├─ MainActivity.kt         # tek activity, tuş yönlendirme, Geri kilidi
│     ├─ ui/                     # PlayerScreen, ChannelOsd, ChannelListPanel,
│     │                          # NumberEntryOverlay, StatusOverlay, AdminScreen, theme/
│     ├─ input/RemoteKeyHandler.kt   # tuş → Intent (saf Kotlin, test edilebilir)
│     ├─ state/TvViewModel.kt        # ekran durumu (UiState), kanal geçiş mantığı
│     ├─ player/                 # PlayerController, PlaybackWatchdog, RetryPolicy
│     ├─ resolver/               # StreamResolver, StepExecutor, ResolvedUrlCache, Template
│     ├─ config/                 # ConfigRepository, ConfigFetcher, SignatureVerifier, model/
│     ├─ data/SettingsStore.kt
│     ├─ work/ConfigRefreshWorker.kt
│     └─ report/ErrorReporter.kt
├─ config/
│  └─ channels.json          # DÜZENLENEN kaynak (imzasız)
├─ dist/                     # Action tarafından üretilir: channels.json + channels.json.sig
├─ tools/
│  ├─ sign_config.py         # doğrula + imzala (yerelde veya Action'da)
│  ├─ validate_config.py     # şema kontrolü
│  ├─ healthcheck.py         # tüm kanalları çöz ve test et
│  └─ gen_keys.py            # tek seferlik Ed25519 anahtar üretimi
└─ .github/workflows/
   ├─ publish-config.yml     # config/ değişince: validate → sign → dist/'e commit
   ├─ healthcheck.yml        # cron: her 2 saatte bir
   └─ android-ci.yml         # push'ta: build + unit test
```

**Repo görünürlüğü:** Tek **public** GitHub repo (ADR-006). Repoda gizli bilgi yok; konfig herkese açık URL'den çekilebilir.

---

## 4. Konfig Şeması (`channels.json`)

```json
{
  "schema": 1,
  "version": 17,
  "updatedAt": "2026-10-26T10:00:00Z",
  "minAppVersion": 1,
  "defaults": {
    "userAgent": "Mozilla/5.0 (Linux; Android 11; Android TV) AppleWebKit/537.36 Chrome/120 Safari/537.36",
    "resolvedTtlSec": 1800
  },
  "channels": [
    {
      "number": 1,
      "id": "trt1",
      "name": "TRT 1",
      "enabled": true,
      "sources": [
        {
          "label": "resmi-direkt",
          "type": "direct",
          "url": "https://ornek.cdn/trt1/master.m3u8",
          "headers": {}
        },
        {
          "label": "resmi-sayfa",
          "type": "steps",
          "ttlSec": 900,
          "steps": [
            { "op": "fetch", "url": "https://kanal.com.tr/canli-yayin",
              "headers": { "Referer": "https://kanal.com.tr/" }, "save": "page" },
            { "op": "regex", "from": "page", "pattern": "\"(https://[^\"]+\\.m3u8[^\"]*)\"", "group": 1, "save": "stream" }
          ],
          "result": "{stream}",
          "playHeaders": { "Referer": "https://kanal.com.tr/" }
        }
      ]
    }
  ]
}
```

### 4.1 Alan kuralları
- `number`: benzersiz, 1..999. Kanal listesi bu sıraya göre gösterilir.
- `id`: benzersiz, küçük harf, değişmez (son kanal hafızası bu id ile tutulur, numara değişse bile doğru kanal açılır).
- `enabled: false` → listede görünmez (geçici bozuk kanalı gizlemek için).
- `sources`: sırayla denenir. En az 1 kaynak zorunlu.
- `minAppVersion`: uygulamanın `versionCode`'u bundan küçükse konfig **uygulanmaz**, eldeki son konfig kullanılır (yeni tarif tipi eski APK'yı bozmasın).
- `version`: tam sayı, her değişiklikte artar. TV daha düşük versiyonu kabul etmez (geri alma saldırısı ve önbellek karışıklığına karşı).

### 4.2 Kaynak tipleri
| type | Anlamı |
|---|---|
| `direct` | `url` doğrudan oynatılır. |
| `steps` | Adımlar sırayla çalıştırılır, `result` şablonu oynatılacak URL'yi verir. |

### 4.3 `steps` adım operasyonları (v1)
| op | Parametreler | Ne yapar |
|---|---|---|
| `fetch` | `url`, `method` (GET/POST), `headers`, `body`, `save` | HTTP isteği; gövdeyi değişkene kaydeder. Yönlendirmeleri izler. Son URL `{save}_url` olarak da kaydedilir. |
| `regex` | `from`, `pattern`, `group`, `save` | İlk eşleşmenin grubunu kaydeder. Eşleşme yoksa kaynak başarısız. |
| `json` | `from`, `path` (ör. `data.streams[0].url`), `save` | JSON içinden değer seçer. |
| `replace` | `from`, `find`, `with`, `save` | Basit metin değişimi (kaçış karakterli linkler için, ör. `\/` → `/`). |
| `urldecode` / `unescape` | `from`, `save` | URL kodu / JSON kaçışını çözer. |

- Şablonlarda `{değişken}` kullanılabilir (url, header, body, result). Hazır değişkenler: `{now_ms}`, `{now_s}`.
- Bir tarif ihtiyacı v1 adımlarıyla karşılanamıyorsa: **yeni op eklenir (APK güncellemesi)**, `minAppVersion` artırılır. Uzaktan JS çalıştırma v1'de YOK (ADR-008).
- **Python sağlık kontrolü aynı adımları birebir uygular** → op semantiği basit ve iki dilde aynı tutulmalı. Yeni op eklenirse `tools/healthcheck.py` de güncellenir.

### 4.4 İmza ve yayınlama
- Kullanıcı yalnızca `config/channels.json`'u düzenler (telefondan GitHub web arayüzüyle).
- `publish-config.yml`: `validate_config.py` → `version` otomatik +1 değil, **doğrulama artışı kontrol eder** (unutulduysa Action kendisi artırır ve commit eder) → `sign_config.py` (özel anahtar `CONFIG_SIGNING_KEY` secret'ından) → `dist/channels.json` + `dist/channels.json.sig` (base64 Ed25519, ham dosya baytları üzerinde).
- TV iki URL'yi sırayla dener:
  1. `https://raw.githubusercontent.com/<kullanici>/<repo>/main/dist/channels.json`
  2. `https://cdn.jsdelivr.net/gh/<kullanici>/<repo>@main/dist/channels.json` (yedek CDN)
- İmza tutmazsa / şema bozuksa / versiyon düşükse → yeni konfig reddedilir, eldeki konfigle devam edilir, `ErrorReporter`'a bildirilir.
- **Gömülü varsayılan konfig:** APK içinde `assets/channels.default.json` (+ imza) bulunur; ilk kurulumda internet yoksa bile liste var.

---

## 5. Uygulama Bileşenleri — Şartname

### 5.1 MainActivity
- Tek activity. `launchMode="singleTask"`, `screenOrientation="landscape"`, `excludeFromRecents="true"`.
- Intent filter: `MAIN` + `LAUNCHER` + `LEANBACK_LAUNCHER` + **`HOME` + `DEFAULT`** (launcher olabilmek için).
- `FLAG_KEEP_SCREEN_ON`, tam ekran (immersive), sistem çubukları gizli.
- `OnBackPressedDispatcher`'a kalıcı callback: Geri → `TvViewModel.onBack()` (liste/overlay açıksa kapatır, değilse hiçbir şey).
- `dispatchKeyEvent` / `onKeyDown` → `RemoteKeyHandler`.
- `onStop`'ta oynatıcı durdurulur (TV kapatılınca kaynak boşa harcanmasın), `onStart`'ta son kanal yeniden başlatılır.

### 5.2 Tuş Haritası (`RemoteKeyHandler`)
| Tuş | Overlay kapalıyken | Kanal listesi açıkken |
|---|---|---|
| `DPAD_UP`, `CHANNEL_UP`, `PAGE_UP` | Sonraki kanal (numara +1, sonda başa sarar) | Listede yukarı |
| `DPAD_DOWN`, `CHANNEL_DOWN`, `PAGE_DOWN` | Önceki kanal | Listede aşağı |
| `DPAD_CENTER`, `ENTER`, `NUMPAD_ENTER` | Kanal listesini aç (izlenen kanal seçili) | Seçili kanala geç, listeyi kapat |
| `0`–`9`, `NUMPAD_0`–`9` | Numara girişi başlar (bkz. 5.4) | Numara girişi başlar |
| `BACK` | **Hiçbir şey** | Listeyi kapat |
| `DPAD_LEFT` / `RIGHT` | Kanal bilgi kutusunu (OSD) göster | Listeyi kapat (RIGHT) |
| `INFO`, `GUIDE` | OSD göster | — |
| `LAST_CHANNEL` (varsa) | Bir önceki izlenen kanala dön | — |
| Ses, Mute, Power | Sisteme bırakılır (müdahale yok) | |
| `MENU`, `SETTINGS` | Yutulur | Yutulur |

- Uzun basmada tekrar eden UP/DOWN: 300 ms'den sık kanal değişimi olmaz (debounce); hızlı basışlarda **yalnızca OSD'de numara ilerler**, oynatıcı son seçimde 600 ms durunca kanalı açar (uydu alıcısı davranışı, gereksiz yayın açmayı önler).
- `RemoteKeyHandler` Android'e bağımlı olmayan saf Kotlin sınıfıdır: `KeyEvent.keyCode → TvIntent` (birim testli).

### 5.3 Ekran Öğeleri (tasarım kuralları)
Genel: siyah/koyu yarı saydam arka plan, beyaz yazı, **en küçük yazı 32sp**, yüksek kontrast, animasyon kısa (≤200 ms), tüm metinler Türkçe.

| Öğe | Davranış |
|---|---|
| **ChannelOsd** | Kanal değişince üstte ortada: "**3 · Kanal D**" (numara ~64sp, isim ~48sp). 3 sn sonra kaybolur. |
| **ChannelListPanel** | Sağ kenarda ekranın ~%35'i. Satır: "3   Kanal D" (~40sp). Aynı anda ~7–8 satır, seçili satır kalın çerçeve + açık arka plan. İzlenen kanal yanında ▶ işareti. 8 sn tuşa basılmazsa kapanır. Arkada video oynamaya devam eder. |
| **NumberEntryOverlay** | Sağ üstte büyük rakamlar "1_", "12". |
| **StatusOverlay** | Yayın açılamazken ekran ortasında: "**Yayın bekleniyor…**" + altında kanal adı. Görüntü yoksa koyu arka plan. Asla hata kodu yok. İnternet tamamen yoksa: "**İnternet bağlantısı bekleniyor…**". |
| **Yükleniyor** | Kanal geçişinde ilk 1,5 sn sadece OSD; 1,5 sn'yi geçerse küçük dönen gösterge + "Yükleniyor…". |
| **AdminScreen** | Bkz. 5.9. |

### 5.4 Numara Girişi
- Rakam tuşu → giriş tamponu (en fazla 3 hane). Her rakamda 2 sn'lik zamanlayıcı sıfırlanır.
- Süre dolunca veya OK basılınca: o numarada kanal varsa geçilir; yoksa "**Kanal yok**" 2 sn gösterilir, mevcut kanal sürer.
- Mevcut en büyük numaradan daha uzun olamayacak girişlerde (ör. 20 kanal varken "3") beklemeden geçmek yerine yine 2 sn beklenir (tutarlılık).
- Tampon `0000` ise → **AdminScreen** açılır (0 numaralı kanal yoktur).

### 5.5 TvViewModel / Durum
```kotlin
data class UiState(
  val channels: List<Channel>,
  val currentId: String?,
  val osdVisible: Boolean,
  val listOpen: Boolean, val listSelection: Int,
  val numberBuffer: String,
  val playback: PlaybackStatus   // Loading, Playing, Waiting(reason), NoInternet
)
```
- Başlangıç: `SettingsStore.lastChannelId` → yoksa ilk kanal. Kanal artık konfigde yoksa → aynı numaradaki kanal → yoksa ilk kanal.
- Her başarılı kanal geçişinde (oynatma başladığında değil, seçimde) `lastChannelId` ve `previousChannelId` kaydedilir.

### 5.6 PlayerController ve Dayanıklılık
Durum makinesi:
```
Idle → Resolving → Buffering → Playing
            │           │          │
            └──fail──→ NextSource ←┘ (hata / takılma)
                         │ (kaynaklar bitti)
                         └→ Waiting (artan bekleme) → Resolving (tekrar 1. kaynaktan)
```
**Kurallar:**
1. Kanal açılışı: `ResolvedUrlCache`'te süresi dolmamış URL varsa **çözmeden hemen oynat** (hız). Hata olursa çözülmüş URL'yi geçersiz kıl ve yeniden çöz.
2. HTTP 401/403/404/410 veya oynatma listesi hatası → aynı kaynağı **bir kez taze çöz** → yine hata → sonraki kaynak.
3. Tüm kaynaklar başarısız → `Waiting`; bekleme: 5 sn, 15 sn, 30 sn, 60 sn, sonra hep 60 sn. **Sonsuza kadar denenir.** Kullanıcı kanal değiştirirse sayaç sıfırlanır.
4. `BehindLiveWindowException` → `seekToDefaultPosition()` + `prepare()` (kaynak değiştirmeden).
5. **PlaybackWatchdog:** Playing/Buffering durumunda oynatma konumu 12 sn ilerlemiyorsa → aynı kaynağı yeniden hazırla; 2. kez olursa yeniden çöz; 3. kez sonraki kaynak.
6. **Ağ izleme:** `ConnectivityManager.NetworkCallback`. Ağ gidince `NoInternet` durumu; ağ gelince bekleme beklenmeden hemen yeniden dene.
7. Uzun süre aynı kanal: token'lı kaynaklarda `ttlSec` dolmadan ~1 dk önce arka planda yeniden çözüp **bir sonraki hata anında** kullanmak üzere hazır tut (izlemeyi kesmeden).
8. ExoPlayer ayarları: canlı yayın için `LiveConfiguration` varsayılan; `LoadControl` tampon min 15 sn / max 50 sn; HLS `allowChunklessPreparation` açık; ağ zaman aşımları 10 sn bağlantı / 15 sn okuma.
9. Kaynak başına `playHeaders` (Referer, Origin, User-Agent) OkHttp data source'a verilir.
10. Her kanal geçişinde eski çözme işi iptal edilir (coroutine `Job` iptali).

### 5.7 StreamResolver / StepExecutor
- `suspend fun resolve(source: Source): ResolvedStream` (url + playHeaders + expiresAt).
- Değişken haritası: `Map<String,String>`; şablon motoru `{isim}` yerleştirmesi (bilinmeyen değişken → hata).
- Toplam çözme zaman aşımı: 15 sn. Yanıt gövdesi sınırı: 2 MB.
- `ResolvedUrlCache`: bellekte + DataStore'da (yeniden açılışta hızlı başlangıç), anahtar `channelId#sourceIndex`, süre `ttlSec` (yoksa `defaults.resolvedTtlSec`).
- Ayrıntılı hata sebebi (`RegexNoMatch`, `Http(403)`, `Timeout`...) yalnızca AdminScreen'de ve ErrorReporter'da görünür.

### 5.8 ConfigRepository
- Açılışta: (1) DataStore'daki son geçerli konfig → yoksa (2) gömülü `assets` konfigi. **Hemen** bununla oynat.
- Arka planda uzaktan çek → doğrula (imza, şema, `version` > mevcut, `minAppVersion` ≤ uygulama) → kaydet → `StateFlow` ile yayınla.
- Konfig değişince oynayan kanal hâlâ varsa ve kaynakları değişmediyse **yayın kesilmez**; kaynakları değiştiyse sıradaki hatada yeni kaynaklar kullanılır (izleyen kişinin yayını gereksiz yere kesilmesin).
- `ConfigRefreshWorker`: 3 saatte bir + uygulama her ön plana geldiğinde.

### 5.9 Gizli Yönetici Ekranı (`0000`)
Büyük yazılı basit menü:
- Uygulama sürümü, konfig sürümü + tarihi, son konfig çekme sonucu
- İnternet durumu, IP (yerel)
- **Kanal testi:** tüm kanalları sırayla çözer, ✓/✗ + hata sebebi listeler
- "Konfigi şimdi yenile"
- "Önbelleği temizle"
- "Android ayarlarını aç" (kullanıcı için: Wi-Fi değiştirme vb.)
- "Uygulamadan çık" (geçici olarak başka uygulamaya geçiş)
- 30 sn işlem yoksa kapanır. Geri tuşu bu ekranı kapatır.

### 5.10 ErrorReporter (Aşama 7, opsiyonel ama önerilir)
- Gerçek IP'den, gerçek sorunları raporlar (GitHub sağlık kontrolünün coğrafi kısıt sorununu çözer).
- Olaylar: bir kanal 10 dk boyunca açılamadı, konfig imzası reddedildi, uygulama çöktü (bir sonraki açılışta).
- Kanal: Telegram Bot API `sendMessage`. Bot token'ı `local.properties` → `BuildConfig` (repoda yok).
- Sınırlama: aynı olay için saatte en fazla 1 mesaj.

---

## 6. Kiosk / Ana Ekran Kurulumu

1. **Normal Android TV (launcher seçilebiliyor):** Uygulama kurulunca Home tuşunda "varsayılan uygulama" sorulur → Aile TV → "Her zaman".
2. **Google TV / launcher seçtirmeyen cihazlar:** Bilgisayarla tek seferlik ADB:
   - TV'de Geliştirici Seçenekleri → USB/Ağ hata ayıklama açık.
   - Mevcut launcher paketi tespit edilir: `adb shell cmd package resolve-activity -a android.intent.action.MAIN -c android.intent.category.HOME`
   - `adb shell pm disable-user --user 0 <launcher_paketi>` → Aile TV tek HOME olur.
   - **Geri alma:** `adb shell pm enable <launcher_paketi>` (docs/SETUP_TV.md'ye yazılacak).
   - ⚠️ Model bazında test edilmeden uygulanmaz; bazı modellerde kurulum sihirbazı paketi de etkilenebilir. Önce yedek plan (aşağıdaki madde 3, BOOT yöntemi) denenebilir.
3. **Yedek açılış yöntemi:** `BOOT_COMPLETED` alıcısı + `SYSTEM_ALERT_WINDOW` izni (ADB ile `appops set com.aile.tv SYSTEM_ALERT_WINDOW allow`) → açılışta activity başlatılır.
4. **Lock Task (Device Owner) modu:** Sadece TV sıfırlanıp hesapsız kurulursa mümkün; varsayılan olarak **kullanılmayacak** (ADR-010).
5. TV'nin kendi ayarlarından: ekran koruyucu / otomatik uyku süresi uzatılır, otomatik güncellemeler gece saatine alınır.
6. Tüm adımlar F6'da `docs/SETUP_TV.md` olarak model özelinde yazılır.

---

## 7. Uzaktan Bakım ve İzleme

| Sorun | Çözüm yolu | Kullanıcının yapması gereken |
|---|---|---|
| Kanal linki bozuldu | `config/channels.json` düzenle → Action imzalar → TV ≤3 saatte çeker (ya da TV yeniden açılınca) | Telefondan GitHub'da düzenleme |
| Yeni kanal ekle/çıkar/sırala | Aynı | Aynı |
| Yeni tarif tipi / hata düzeltmesi | Yeni APK → Tailscale üzerinden `adb connect <tv-tailscale-ip>:5555` → `adb install -r` | Bilgisayardan, evden uzakta |
| Bozulmayı fark etme | `healthcheck.yml` (2 saatte bir) + ErrorReporter → Telegram | Bildirime bakmak |

**healthcheck.yml:** `tools/healthcheck.py` her kanalın her kaynağını çözer, master m3u8'i ve ilk segmenti indirir. Bir kanalın **tüm** kaynakları başarısızsa Telegram'a mesaj (aynı kanal için durum değişene kadar tekrar mesaj yok; düzelince "düzeldi" mesajı).
Sınır: GitHub sunucuları yurt dışında → yalnızca Türkiye'den açılan yayınlar burada "bozuk" görünebilir. Bu kanallar `channels.json`'da `"geoTR": true` ile işaretlenir ve sağlık kontrolünde atlanır; onları ErrorReporter izler.

**Tailscale:** TV'ye Tailscale (Android TV uygulaması mevcut) kurulur, kullanıcının hesabına bağlanır. TV'de ağ üzerinden ADB açık kalır. Güvenlik: ADB yalnızca Tailscale ağından erişilebilir olmalı (ev ağı paylaşılıyorsa not düşülecek).

---

## 8. Güvenlik ve Gizlilik
- Konfig Ed25519 ile imzalı; açık anahtar APK'da. Özel anahtar yalnızca GitHub Secret + kullanıcının yedeği (parola yöneticisi).
- Sadece HTTPS konfig URL'leri. Yayın URL'leri HTTP olabilir (bazı kanallar) → `network_security_config` ile cleartext yalnızca medya için izinli (ADR'de değerlendirilecek).
- Uygulama kullanıcı verisi toplamaz. ErrorReporter yalnızca kanal adı + hata sebebi + uygulama sürümü gönderir.
- Release APK kullanıcının kendi keystore'uyla imzalanır; keystore repoda değil (yedeği kullanıcıda, kaybolursa güncelleme yerine kaldır-kur gerekir).

---

## 9. Test Stratejisi
| Seviye | Kapsam |
|---|---|
| Birim | `RemoteKeyHandler`, numara girişi zamanlayıcısı, `TvViewModel` kanal geçişleri, `StepExecutor` (MockWebServer ile tüm op'lar), şablon motoru, `RetryPolicy`, konfig doğrulama (imza, versiyon, minAppVersion) |
| Python | `healthcheck.py` ve `StepExecutor` için **ortak test vektörleri** (`tools/testdata/*.json`: girdi sayfa + tarif → beklenen URL). İki gerçekleme aynı vektörlerden geçmeli. |
| Emülatör | Android TV emülatörü (1080p): tuşlar, overlay'ler, Geri kilidi, launcher davranışı |
| Gerçek cihaz | Gerçek TV'de 20 kanal; internet kesme/geri getirme; router yeniden başlatma; TV kapat-aç; 8 saat kesintisiz oynatma |
| Saha | 1 hafta anne-baba kullanımı, ErrorReporter kayıtları |

---

## 10. Aşamalar, Görevler ve Kabul Kriterleri

Görev kimlikleri (`F1.2` gibi) `PROGRESS.md`'de kilit ve durum takibi için kullanılır.

### F0 — Hazırlık (kod yok)
| ID | Görev | Kabul kriteri |
|---|---|---|
| F0.1 | Kullanıcıdan TV marka/model/Android sürümü, Google TV mi? | `HANDOFF.md`'de yazılı; minSdk ADR'si |
| F0.2 | Kullanıcının GitHub + Telegram hesapları, repo adı | Repo oluşturuldu, `AGENTS.md` Bölüm 7'de URL'ler |
| F0.3 | Kanal listesi kesinleşmesi (20 kanal, numaraları) | `CHANNELS.md` kullanıcı onaylı |
| F0.4 | Her kanal için resmi canlı yayın kaynağı araştırması: direkt mi, token'lı mı, DRM'li mi, coğrafi kısıtlı mı | `CHANNELS.md`'de her kanal için durum + aday tarif; VLC/curl ile doğrulama notu |
| F0.5 | Geliştirme ortamı: Android Studio, JDK, Android TV emülatörü, ADB, Python | `AGENTS.md` Bölüm 7 komutları çalışıyor |

### F1 — Oynatıcı İskeleti
| ID | Görev | Kabul kriteri |
|---|---|---|
| F1.1 | Gradle projesi, version catalog, paket yapısı (Bölüm 3) | `assembleDebug` başarılı, `android-ci.yml` yeşil |
| F1.2 | MainActivity: tam ekran, ekran açık kalır, manifest (HOME filtreleri dahil) | Emülatörde siyah tam ekran açılır |
| F1.3 | Sabit tek HLS kanalı Media3 ile oynat | Görüntü + ses var |
| F1.4 | Geri tuşu kilidi | 20 kez Geri → uygulama ön planda kalır |

### F2 — Kumanda ve Arayüz
| ID | Görev | Kabul kriteri |
|---|---|---|
| F2.1 | Model sınıfları + gömülü `assets/channels.default.json` (direkt kaynaklarla) | Liste uygulamaya yükleniyor |
| F2.2 | `RemoteKeyHandler` + birim testleri | Tüm tuş haritası testli |
| F2.3 | Kanal değiştirme + ChannelOsd + debounce | Yukarı/aşağıyla geçiş, OSD 3 sn |
| F2.4 | ChannelListPanel | OK açar, kaydırma, seçim, 8 sn kapanma, Geri kapatır |
| F2.5 | NumberEntryOverlay + "Kanal yok" | "12" tuşla → 2 sn sonra 12. kanal |
| F2.6 | SettingsStore: son kanal / önceki kanal | Uygulama kapat-aç → son kanal açılır |
| F2.7 | StatusOverlay + yükleniyor göstergesi (Türkçe, büyük) | Bozuk URL'de "Yayın bekleniyor…" |

### F3 — Uzaktan Konfig
| ID | Görev | Kabul kriteri |
|---|---|---|
| F3.1 | `tools/gen_keys.py`, `sign_config.py`, `validate_config.py` | Yerelde imzalı dist üretiliyor |
| F3.2 | `publish-config.yml` | `config/` commit'i → `dist/` otomatik güncelleniyor |
| F3.3 | `SignatureVerifier` + testler (geçerli/bozuk imza, düşük versiyon, yüksek minAppVersion) | Testler yeşil |
| F3.4 | `ConfigRepository` (önbellek → asset → uzak, iki URL) + `ConfigRefreshWorker` | İnternetsiz açılışta son konfig; GitHub'da değişiklik → yenilemede uygulamaya yansır |
| F3.5 | Konfig değişiminde yayını kesmeme kuralı (5.8) | Oynayan kanal kesilmiyor |

### F4 — Link Çözücü
| ID | Görev | Kabul kriteri |
|---|---|---|
| F4.1 | Şablon motoru + `StepExecutor` (fetch, regex, json, replace, urldecode/unescape) | MockWebServer testleri yeşil |
| F4.2 | Ortak test vektörleri (`tools/testdata/`) | Kotlin testleri vektörlerden geçiyor |
| F4.3 | `ResolvedUrlCache` (bellek + DataStore, TTL) | Yeniden açılışta çözmeden oynatma |
| F4.4 | F0.4'teki token'lı kanalların tariflerini `config/channels.json`'a yaz | Gerçek TV/emülatörde bu kanallar açılıyor |

### F5 — Dayanıklılık
| ID | Görev | Kabul kriteri |
|---|---|---|
| F5.1 | `RetryPolicy` + durum makinesi (5.6 kural 1–3) + testler | Bozuk 1. kaynakta 2.'ye geçiş; hepsi bozukta artan bekleme |
| F5.2 | `PlaybackWatchdog` | Akış dondurulunca (proxy ile) 12 sn içinde toparlanma |
| F5.3 | Ağ izleme + NoInternet durumu | Wi-Fi kes → mesaj; aç → ≤5 sn'de yayın |
| F5.4 | BehindLiveWindow, token yenileme (kural 4, 7) | 8 saatlik kesintisiz oynatma testi |

### F6 — Kiosk ve Kurulum
| ID | Görev | Kabul kriteri |
|---|---|---|
| F6.1 | HOME launcher davranışının cihazda doğrulanması | Home tuşu → Aile TV |
| F6.2 | Yedek açılış (BOOT_COMPLETED + overlay izni) | TV kapat-aç → doğrudan yayın |
| F6.3 | AdminScreen (5.9) | `0000` ile açılıyor, kanal testi çalışıyor |
| F6.4 | Release imzalama, `docs/SETUP_TV.md` (model özelinde ADB adımları + geri alma) | Kullanıcı dokümana bakarak kurabiliyor |

### F7 — İzleme ve Uzaktan Bakım
| ID | Görev | Kabul kriteri |
|---|---|---|
| F7.1 | `healthcheck.py` + `healthcheck.yml` + Telegram | Bozuk kanal → Telegram mesajı; düzelince "düzeldi" |
| F7.2 | ErrorReporter (Telegram, hız sınırı) | TV'de bozuk kanal 10 dk → mesaj |
| F7.3 | Tailscale + uzaktan ADB kurulumu dokümanı | Kullanıcı evden uzaktayken `adb install` yapabildi |
| F7.4 | `docs/BAKIM.md`: "Kanal bozuldu, ne yaparım?" telefondan adım adım | Kullanıcı bir linki tek başına düzeltebildi |

### F8 — Saha Testi
| ID | Görev | Kabul kriteri |
|---|---|---|
| F8.1 | 1 hafta anne-baba kullanımı | Kullanıcı geri bildirimi `HANDOFF.md`'de |
| F8.2 | Geri bildirim düzeltmeleri | Kullanıcı onayı → v1.0 |

**Kritik yol:** F0 → F1 → F2 → F3 → F4 → F5 → F6 → F8. F7 (izleme), F3 bittikten sonra paralel yürüyebilir.

---

## 11. Riskler ve Önlemler
| Risk | Etki | Önlem |
|---|---|---|
| Kanal DRM'e geçer / internet yayınını kapatır | Kanal düşer | Alternatif resmi kaynak (ör. kanalın resmi YouTube canlı yayını — ADR gerekir); olmazsa `enabled:false` |
| Kanal sitesi değişir, tarif bozulur | Kanal düşer | Çoklu kaynak + Telegram uyarısı + telefondan düzeltme |
| Google TV launcher değişimine izin vermez | Home tuşu kaçış yolu olur | ADB ile stok launcher devre dışı; yedek BOOT yöntemi |
| TV sistem güncellemesi launcher ayarını sıfırlar | Kiosk bozulur | SETUP_TV.md'de tekrar adımları; Tailscale ile uzaktan düzeltme |
| GitHub erişilemez | Konfig güncellenmez | jsDelivr yedeği; eldeki konfigle çalışmaya devam |
| İmza anahtarı kaybolur | Konfig yayınlanamaz | Anahtar yedeği; yeni anahtar → APK güncellemesi |
| İnternet yavaş | Takılma | ABR (otomatik kalite), tampon ayarları |
| İki ajan aynı dosyayı değiştirir | Çakışma | Görev kilidi + modül sahipliği (Bölüm 12) |

---

## 12. Çoklu Ajan Çalışma Düzeni (Claude + Codex)

**Ortak hafıza = bu repodaki dosyalar + git geçmişi.** Ajanların kendi özel hafızaları paylaşılmaz.

- Claude Code `CLAUDE.md`'yi okur (o da `@AGENTS.md` ile içe aktarır); Codex `AGENTS.md`'yi okur → iki ajan **aynı kuralları** görür.
- **Sıralı çalışma** (önerilen): Aynı anda tek ajan kod yazar. Kullanıcı bir ajanla oturumu kapatır, diğerini açar; devir `HANDOFF.md` üzerinden olur.
- **Paralel çalışma** gerekirse: her ajan ayrı git dalında (`claude/F4.1`, `codex/F7.1`) ve **farklı modüllerde** çalışır, kullanıcı birleştirir.
- **Modül sahipliği önerisi (paralel çalışmada):**
  - Claude: `player/`, `state/`, `resolver/`, mimari ve inceleme
  - Codex: `tools/`, `.github/workflows/`, `ui/` bileşenleri, testler
  - (Kullanıcı istediği gibi değiştirebilir; değişirse buraya yazılır.)
- **Çapraz inceleme:** Bir ajanın tamamladığı aşama, diğer ajan tarafından kabul kriterlerine göre gözden geçirilir; bulgular `HANDOFF.md`'ye yazılır.
- Commit mesajı öneki: `[claude]` / `[codex]` / `[kullanici]` + görev ID.
