# AGENTS.md — Aile TV Projesi (Ortak Hafıza Giriş Noktası)

> Bu dosya projede çalışan **tüm yapay zeka ajanlarının** (Claude Code, OpenAI Codex ve diğerleri) ortak giriş noktasıdır.
> Claude Code bu dosyayı `CLAUDE.md` üzerinden, Codex ise doğrudan otomatik okur.
> **Her oturumun başında bu dosyayı ve aşağıdaki "Okuma Sırası"ndaki dosyaları oku.**

---

## 1. Proje Özeti (30 saniyede)

Kullanıcının annesi ve babası için **Android TV'de çalışan, uydu alıcısı gibi davranan canlı TV uygulaması**.
- Açılınca menü göstermeden **son izlenen kanalı tam ekran** oynatır.
- Kumanda: Yukarı/Aşağı = kanal değiştir, OK = büyük yazılı kanal listesi, rakamlar = kanal numarası.
- **Geri tuşu uygulamadan çıkaramaz**, uygulama TV'nin ana ekranı (launcher) olarak çalışır.
- ~20 Türk ulusal/haber kanalını internetten (HLS) oynatır. Linkler GitHub'daki **imzalı `channels.json`** dosyasından gelir;
  token'lı linkleri **TV'nin kendisi** uzaktan gelen "tariflerle" çözer. Hedef: kullanıcının eve gitmeden, telefondan bakım yapabilmesi.

Ayrıntılı plan: [`docs/PLAN.md`](docs/PLAN.md)

---

## 2. Ortak Hafıza Dosyaları ve Okuma Sırası

| Sıra | Dosya | Ne işe yarar | Kim günceller |
|---|---|---|---|
| 1 | `AGENTS.md` (bu dosya) | Kurallar, protokol, hızlı bağlam | Sadece kullanıcı onayıyla |
| 2 | `docs/HANDOFF.md` | **Son oturumda ne yapıldı, sıradaki adım ne, açık sorular** | Her oturum sonunda (en üste yeni kayıt) |
| 3 | `docs/PROGRESS.md` | Görev listesi, durum, **görev kilitleri** | Görev alınca / bitince |
| 4 | `docs/PLAN.md` | Mimari, teknik şartname, aşamalar, kabul kriterleri | Plan değişince (kullanıcı onayıyla) |
| 5 | `docs/DECISIONS.md` | Alınmış teknik kararlar (ADR) ve gerekçeleri | Yeni karar alındığında |
| 6 | `docs/CHANNELS.md` | Kanal listesi, kaynak araştırma durumu, bilinen sorunlar | Kanal araştırması / bakım yapıldıkça |

**Kural:** Bu dosyalar dışında "gizli" hafıza yoktur. Bir ajanın kendi özel hafızasına yazdığı şey diğer ajan tarafından görülmez.
Önemli olan her bilgi **mutlaka bu dosyalara** yazılmalıdır.

---

## 3. Oturum Protokolü

Bu protokol iki ortak skill olarak da paketlendi (tek kaynak `.agents/skills/`, Claude için `.claude/skills/` ince yönlendirme):
- **`devral`** — oturum başı. Claude: `/devral` · Codex: `$devral` · ya da sadece "devral" yaz.
- **`devret`** — oturum sonu. Claude: `/devret` · Codex: `$devret` · ya da "devret" yaz.
Skill yüklenmese bile aşağıdaki adımlar geçerlidir. Protokol değişirse **önce `.agents/skills/*/SKILL.md`**, sonra bu bölüm güncellenir.

### Oturum başında
1. `AGENTS.md` → `docs/HANDOFF.md` (en üstteki kayıt) → `docs/PROGRESS.md` oku. Gerekirse `PLAN.md` ve `DECISIONS.md`'nin ilgili bölümü.
2. `git status` ve `git log --oneline -10` ile son değişiklikleri kontrol et (başka ajan çalışmış olabilir).
3. `PROGRESS.md`'de üzerinde çalışacağın görevi **kilitle** (bkz. Bölüm 4).

### Oturum sonunda (atlanamaz)
1. `PROGRESS.md`: görev durumunu güncelle, kilidi kaldır veya "devam ediyor" olarak bırak.
2. `HANDOFF.md`: **en üste** yeni kayıt ekle (şablon dosyada). Ne yapıldı, ne yarım kaldı, sıradaki adım, açık sorular.
3. Yeni teknik karar aldıysan `DECISIONS.md`'ye ADR ekle.
4. Kullanıcı commit'e izin verdiyse commit at: `[claude] F2.3: kanal OSD eklendi` / `[codex] F4.1: scrape adımı`.

---

## 4. Görev Kilitleme (Çakışmayı Önleme)

- Aynı anda **bir görev = bir ajan**. Görevi almadan önce `PROGRESS.md`'deki satıra şunu yaz:
  `🔒 claude 2026-10-26` veya `🔒 codex 2026-10-26`
- Başkasının kilidi olan göreve **dokunma**. Kilit 2 günden eskiyse ve HANDOFF'ta bilgi yoksa kullanıcıya sor.
- Aynı dosyalara dokunan iki görev aynı anda yürütülmez. Modül sahipliği önerisi PLAN.md Bölüm 12'de.

---

## 5. Değiştirilemez Kurallar

1. **Kullanıcı ile Türkçe konuş.** Dokümanlar Türkçe. Kod tanımlayıcıları (sınıf/fonksiyon/değişken) İngilizce, ekranda görünen metinler Türkçe.
2. **Korsan IPTV listeleri, DRM kırma, ücretli içerik atlatma YOK.** Sadece kanalların herkese açık ücretsiz yayınları.
3. **Gizli bilgi repoya girmez:** imza özel anahtarı, Telegram bot token'ı vb. → `local.properties`, GitHub Secrets veya ortam değişkeni.
4. `DECISIONS.md`'deki bir kararı **kullanıcıya sormadan değiştirme.** Değişiklik gerekiyorsa gerekçeyle öner; onaylanırsa yeni ADR yaz, eskisini "Yerine geçti: ADR-0XX" diye işaretle.
5. Annenin/babanın göreceği ekranda **asla** teknik hata mesajı, stack trace, İngilizce metin, küçük yazı olmaz.
6. Geri tuşu uygulamadan çıkış yaptırmaz. Bu davranışı bozan değişiklik kabul edilmez.
7. Çalışmayan kodu "bitti" diye işaretleme. Doğrulanmadıysa HANDOFF'a "doğrulanmadı" yaz.

---

## 6. Hızlı Teknik Bağlam

- **Dil/Platform:** Kotlin, yerel Android (Android TV). Arayüz: Jetpack Compose + `androidx.tv` (Leanback yok).
- **Oynatıcı:** AndroidX Media3 ExoPlayer + HLS + OkHttp data source.
- **Konfig:** `config/channels.json` → GitHub Action imzalar (Ed25519) → `dist/` olarak yayınlar → TV çeker. Uygulamada açık anahtar gömülü.
- **Link çözme:** TV üzerinde, `channels.json` içindeki "steps" tarifleriyle (fetch → regex/json → url). Sunucu tarafında çözme YOK (token'lar IP'ye bağlı).
- **Dayanıklılık:** kaynak başına yeniden çözme → sonraki kaynak → artan beklemeyle sonsuz yeniden deneme; takılma bekçisi (watchdog); ağ geri gelince devam.
- **Kiosk:** Uygulama HOME launcher olarak kayıtlı; Geri tuşu yutulur; gizli yönetici menüsü (`0000` tuşla).
- **İzleme:** GitHub Actions sağlık kontrolü + Telegram uyarısı; uzaktan APK güncelleme için Tailscale + ADB.

## 7. Repo ve Komutlar

- **GitHub:** https://github.com/Vanta-exe/aile-tv (public, varsayılan dal `main`)
- **Konfig URL'leri (F3'te `dist/` oluşunca geçerli olacak):**
  - `https://raw.githubusercontent.com/Vanta-exe/aile-tv/main/dist/channels.json`
  - `https://cdn.jsdelivr.net/gh/Vanta-exe/aile-tv@main/dist/channels.json`
- Push yalnızca kullanıcı isteyince.

Komutlar (proje kurulduktan sonra doldurulacak):

```
# derleme       : ./gradlew assembleDebug
# birim testleri: ./gradlew testDebugUnitTest
# TV'ye kurulum : adb install -r app/build/outputs/apk/debug/app-debug.apk
# konfig imzala : python tools/sign_config.py
# sağlık testi  : python tools/healthcheck.py
```
