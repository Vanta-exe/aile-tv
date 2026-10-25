---
name: devral
description: Aile TV projesinde oturum başında işi devralma. Kullanıcı "devral", "kaldığın yerden devam et", "HANDOFF'tan devam", "nerede kalmıştık" dediğinde veya yeni bir çalışma oturumu kod yazmaya başlamadan önce kullan. Son devir kaydını okur, durumu özetler, görevi kilitler. Oturum sonu için değil (onun için devret skill'i).
---

# Devral — Oturum Başı Protokolü

Bu skill hem Claude Code hem Codex için ortaktır. Asıl kaynak `.agents/skills/devral/SKILL.md` dosyasıdır.
Kendini tanı: Claude isen ajan adın `claude`, Codex isen `codex`. Tarihi `YYYY-AA-GG` biçiminde kullan.

## Adımlar

1. **Bağlamı oku** (sırayla, zaten bağlamındaysa tekrar okuma):
   - `AGENTS.md`
   - `docs/HANDOFF.md` → yalnızca **en üstteki** kayıt (gerekirse bir öncekine de bak)
   - `docs/PROGRESS.md`
   - Sıradaki görevle ilgili `docs/PLAN.md` bölümü ve ilgili ADR'ler (`docs/DECISIONS.md`)

2. **Repo durumunu kontrol et:**
   ```
   git status --short
   git log --oneline -10
   ```
   - Commit edilmemiş değişiklik varsa ve HANDOFF'ta açıklanmamışsa: **dokunma**, kullanıcıya bildir (başka ajanın yarım işi olabilir).
   - Son commit'ler HANDOFF'taki kayıttan daha yeniyse, HANDOFF eksik bırakılmış demektir → commit mesajlarından ne yapıldığını çıkar ve kullanıcıya söyle.

3. **Kilitleri kontrol et** (`PROGRESS.md`'de `🔒`):
   - Başka ajanın kilidi olan göreve dokunma.
   - Kilit 2 günden eskiyse ve HANDOFF'ta açıklama yoksa kullanıcıya "kilidi kaldırayım mı?" diye sor.

4. **Kullanıcıya Türkçe, kısa özet ver** (en fazla ~10 satır):
   - Son oturumda kim, ne yaptı
   - Şu anki aşama ve sıradaki adım
   - Cevap bekleyen açık sorular (varsa — engelleyici olanları önce sor)
   - Doğrulanmamış / dikkat edilmesi gereken noktalar

5. **Görevi seç ve kilitle:**
   - Sıradaki görev HANDOFF'taki "Sıradaki adım"dır. Kullanıcı başka bir şey istediyse onu al.
   - Açık sorular görevi engelliyorsa kod yazmadan önce kullanıcıya sor.
   - `PROGRESS.md`'de görev satırının durumunu `🔒 <ajan> <tarih>` yap; dosyanın üstündeki "Şu anki aşama / Sıradaki iş" satırlarını güncelle.

6. **Çalışmaya başla.** Kullanıcı tercihi gereği: yeni bir aşamaya geçerken önce kısa plan + gerekçe sun, onay alınca kodla.

## Yapma
- HANDOFF'u okumadan kod yazma.
- `DECISIONS.md`'deki kararları kullanıcıya sormadan değiştirme.
- Bu skill'in sonunda commit atma (kilit değişikliği devret sırasında commit edilir).
