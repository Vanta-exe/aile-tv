---
name: devret
description: Aile TV projesinde oturum sonunda işi devretme. Kullanıcı "devret", "oturumu kapat", "bugünlük bu kadar", "Codex'e/Claude'a geçiyorum", "kaydet ve bitir" dediğinde veya bir görev/aşama tamamlandığında kullan. PROGRESS, HANDOFF, DECISIONS dosyalarını günceller, gizli bilgi kontrolü yapar, onay varsa commit atar. Oturum başı için değil (onun için devral skill'i).
---

# Devret — Oturum Sonu Protokolü

Bu skill hem Claude Code hem Codex için ortaktır. Asıl kaynak `.agents/skills/devret/SKILL.md` dosyasıdır.
Kendini tanı: Claude isen ajan adın `claude`, Codex isen `codex`. Tarihi `YYYY-AA-GG` biçiminde kullan.
Amaç: bir sonraki ajan (sen değil, başka bir model olabilir) **sadece dosyaları okuyarak** kaldığın yerden devam edebilmeli.

## Adımlar

1. **Bu oturumda ne değişti, topla:**
   ```
   git status --short
   git diff --stat
   git log --oneline -10
   ```

2. **Gizli bilgi kontrolü** — değişen/yeni dosyalarda şunları ara; bulursan commit etme, kullanıcıya bildir:
   - Özel anahtar (`PRIVATE KEY`, `CONFIG_SIGNING_KEY=` değeri), Telegram bot token'ı (`\d{8,10}:[A-Za-z0-9_-]{35}`), keystore şifreleri, `local.properties` içeriği
   - `.gitignore`'da olması gerekenler: `local.properties`, `*.jks`, `*.keystore`, imza özel anahtar dosyaları

3. **`docs/PROGRESS.md` güncelle:**
   - Biten ve **doğrulanan** görevler → `✅` (not sütununa ajan + tarih)
   - Kısmen biten veya doğrulanamayan → `🟨` + kısa not ("emülatörde test edilmedi" gibi)
   - Engelli → `⛔` + sebep
   - Kendi kilitlerini kaldır. Yarım bırakıp yarın sen devam edeceksen kilidi bırakabilirsin, ama HANDOFF'ta belirt.
   - Üstteki "Şu anki aşama / Sıradaki iş" satırlarını güncelle.

4. **`docs/HANDOFF.md` en üstüne yeni kayıt ekle** (dosyadaki şablonla):
   - **Yapılanlar:** somut, dosya adlarıyla
   - **Yarım kalanlar / doğrulanmayanlar:** dürüst ol; çalıştırılmayan test, denenmeyen cihaz vb.
   - **Alınan kararlar:** ve DECISIONS.md'ye işlendi mi
   - **Açık sorular:** kullanıcıdan beklenenler
   - **Sıradaki adım:** bir sonraki ajanın ilk yapacağı iş, görev ID'siyle
   - Kayıt kısa olsun (~15–25 satır); ayrıntı PLAN/DECISIONS'a ait.

5. **Yeni teknik karar alındıysa** `docs/DECISIONS.md`'ye ADR ekle (şablon dosyada). Kullanıcı onayı olmadan mevcut bir ADR'yi değiştirme.

6. **Kanal kaynağı değiştiyse** `docs/CHANNELS.md` bakım günlüğüne satır ekle.

7. **PLAN.md'den sapma olduysa** (farklı yöntem, yeni bileşen) ilgili bölüme kısa not düş veya kullanıcıya plan güncellemesi öner.

8. **Commit:**
   - Kullanıcı bu oturumda commit'e izin verdiyse (veya şimdi sorup onay alırsan):
     ```
     git add -A
     git commit -m "[<ajan>] <görev ID'leri>: <kısa özet>"
     ```
   - İzin yoksa commit atma; kullanıcıya commit edilmemiş değişiklik olduğunu söyle.
   - Push yalnızca kullanıcı açıkça isterse.

9. **Kullanıcıya Türkçe kapanış mesajı** (en fazla ~8 satır): ne bitti, ne doğrulanmadı, sıradaki adım ve "Bir sonraki oturumda (Claude veya Codex) **devral** demen yeterli."

## Yapma
- HANDOFF kaydı yazmadan oturumu bitmiş sayma.
- Doğrulanmamış işi ✅ yapma.
- Başka ajanın kilidini veya HANDOFF kaydını silme/değiştirme (eski kayıtlar tarihçedir).
