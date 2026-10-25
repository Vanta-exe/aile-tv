# Teknik Kararlar (ADR Kaydı)

> Kural: Bir kararı değiştirmek için kullanıcı onayı gerekir. Değişirse eski kaydı silme; durumunu "Yerine geçti: ADR-0XX" yap ve yeni ADR ekle.
> Şablon en altta.

---

### ADR-001 · Yerel Kotlin + Android (çapraz platform değil)
- **Durum:** Kabul · 2026-10-25
- **Karar:** Uygulama Kotlin ile yerel Android olarak yazılır.
- **Gerekçe:** Android TV'de en iyi performans, kumanda tuşlarına (KeyEvent) doğrudan erişim, Media3 ile birinci sınıf entegrasyon. Flutter/React Native ek katman ve eski TV kutularında takılma riski getirir; iOS/web hedefi yok.

### ADR-002 · Oynatıcı: AndroidX Media3 ExoPlayer
- **Durum:** Kabul · 2026-10-25
- **Karar:** HLS oynatma için Media3 ExoPlayer + OkHttp data source.
- **Gerekçe:** Google'ın resmi, aktif bakımlı oynatıcısı; ayrıntılı hata tipleri (HTTP kodu ayırt edilebiliyor) yeniden çözme/yedek kaynak mantığı için şart. LibVLC daha ağır ve hata kontrolü zayıf.

### ADR-003 · UI: Compose + androidx.tv, Leanback yok
- **Durum:** Kabul · 2026-10-25
- **Gerekçe:** Uygulamanın arayüzü sadece video + birkaç overlay. Leanback'in afiş/ızgara yapısı gereksiz; Compose ile büyük yazılı, özel tasarım kolay.

### ADR-004 · Uzaktan, imzalı JSON konfig; sunucu yok
- **Durum:** Kabul · 2026-10-25
- **Karar:** Kanal listesi ve link tarifleri GitHub'daki `channels.json`'da. Kalıcı sunucu (VPS/backend) kullanılmaz.
- **Gerekçe:** Maliyet sıfır, bakımı gereken sunucu yok; kullanıcı telefondan düzenleyebiliyor. Yedek CDN olarak jsDelivr.

### ADR-005 · Link çözme TV üzerinde yapılır
- **Durum:** Kabul · 2026-10-25
- **Karar:** Token'lı linkler, konfigdeki "steps" tarifleriyle **cihazda** çözülür.
- **Gerekçe:** Yayın token'ları çoğunlukla isteyen IP'ye bağlı; sunucuda çözülen link evdeki TV'de 403 verir. Ayrıca Türkiye IP'si gerektiren yayınlar için cihaz zaten doğru konumda.
- **Bedeli:** Tarif mantığı hem Kotlin'de (uygulama) hem Python'da (sağlık kontrolü) var → ortak test vektörleriyle senkron tutulur.

### ADR-006 · Tek public GitHub reposu
- **Durum:** Kabul (kullanıcı değiştirebilir) · 2026-10-25
- **Gerekçe:** Konfig herkese açık raw URL'den çekilebilir; özel repo + GitHub Pages ücretli plan gerektirir. Repoda gizli bilgi olmadığı için risk düşük.
- **Alternatif:** Kod özel repo + konfig ayrı public repo (gizlilik istenirse).

### ADR-007 · Konfig imzası Ed25519
- **Durum:** Kabul · 2026-10-25
- **Karar:** `dist/channels.json.sig` = Ed25519 imza (base64). Açık anahtar APK'da gömülü. Özel anahtar GitHub Secret `CONFIG_SIGNING_KEY`.
- **Gerekçe:** Kısa anahtar, hızlı, basit. minSdk 21'de platformda Ed25519 olmadığından küçük bir kütüphane (eddsa veya BouncyCastle) kullanılır.

### ADR-008 · Uzaktan kod (JS) çalıştırma v1'de yok
- **Durum:** Kabul · 2026-10-25
- **Karar:** Tarifler bildirimsel adımlardır (fetch/regex/json/replace/urldecode). QuickJS/Rhino ile uzaktan script çalıştırılmaz.
- **Gerekçe:** Güvenlik ve basitlik; Python sağlık kontrolüyle birebir eşleme kolay. Yetersiz kalırsa yeni op eklenir (APK güncellemesi). İleride ihtiyaç olursa yeniden değerlendirilir.

### ADR-009 · Kiosk: HOME launcher + Geri tuşu yutma
- **Durum:** Kabul · 2026-10-25
- **Karar:** Uygulama HOME intent filtresiyle ana ekran olur; Geri tuşu çıkış yaptırmaz; gizli yönetici menüsü `0000`.
- **Gerekçe:** Home tuşunu normal uygulama engelleyemez; launcher olmak tek güvenilir yol. Google TV'de ADB ile stok launcher devre dışı bırakılır.

### ADR-010 · Device Owner / Lock Task modu kullanılmaz (varsayılan)
- **Durum:** Kabul · 2026-10-25
- **Gerekçe:** TV'nin fabrika ayarlarına döndürülüp hesapsız kurulmasını gerektirir; Google hesabı/uygulamaları bozar. Launcher yöntemi yeterli.

### ADR-011 · Sadece resmi, ücretsiz, herkese açık yayın kaynakları
- **Durum:** Kabul · 2026-10-25
- **Gerekçe:** Korsan IPTV listeleri hem yasal risk hem de en sık bozulan kaynak. DRM kırma yapılmaz. Kanal sitelerinden link çekmenin site şartlarına uygunluğu kullanıcının kişisel kullanım kararıdır.

### ADR-012 · Uzaktan bakım: Telegram uyarısı + Tailscale/ADB
- **Durum:** Kabul · 2026-10-25
- **Karar:** GitHub Actions sağlık kontrolü ve TV'deki ErrorReporter Telegram'a bildirir; APK güncellemeleri Tailscale üzerinden uzaktan ADB ile kurulur.
- **Gerekçe:** Kullanıcının eve gitmeden bakım yapabilmesi ana hedef.

### ADR-013 · Ortak hafıza: repo içi Markdown dosyaları
- **Durum:** Kabul · 2026-10-25
- **Karar:** Claude ve Codex için tek kaynak `AGENTS.md`; `CLAUDE.md` onu `@AGENTS.md` ile içe aktarır. Durum/devir `docs/HANDOFF.md`, `docs/PROGRESS.md` dosyalarında; geçmiş git'te.
- **Gerekçe:** Her iki araç da bu dosyaları otomatik okur; ajanların özel hafızaları birbirine görünmez.

### ADR-014 · Oturum protokolü ortak skill olarak (devral / devret)
- **Durum:** Kabul · 2026-10-25 · claude
- **Karar:** Oturum başı/sonu adımları iki skill'e paketlendi. Tek kaynak `.agents/skills/<ad>/SKILL.md` (Codex'in repo içi skill konumu). Claude Code `.claude/skills/<ad>/SKILL.md` okuduğu için orada yalnızca asıl dosyaya yönlendiren ince bir kopya var.
- **Gerekçe:** Kuralların ajanın "hatırlamasına" bırakılmaması; iki ajanın aynı adımları izlemesi. Hazır üçüncü taraf skill'ler (ör. ShareMemory) ikinci bir hafıza klasörü açacağı ve kodu incelenmediği için kullanılmadı.
- **Alternatifler / Bedeli:** Sembolik link (Windows'ta ve git'te güvenilmez) yerine yönlendirme dosyası seçildi; skill açıklaması (description) değişirse iki yerde güncellenmeli.

### Bekleyen kararlar
- **ADR-0xx · minSdk:** TV modeli öğrenilince (F0.1).
- **ADR-0xx · Cleartext (HTTP) yayın izni:** F0.4 kanal araştırmasından sonra.
- **ADR-0xx · YouTube resmi canlı yayınları yedek kaynak olarak kullanılsın mı?** Gerekirse F0.4 sonrası.

---

### Şablon
```
### ADR-0XX · Başlık
- **Durum:** Önerildi | Kabul | Yerine geçti: ADR-0YY · tarih · (claude|codex|kullanici)
- **Karar:**
- **Gerekçe:**
- **Alternatifler / Bedeli:**
```
