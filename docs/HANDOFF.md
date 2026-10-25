# Devir Teslim Kaydı (HANDOFF)

> **En yeni kayıt en üstte.** Her oturum sonunda bir kayıt ekle (şablon en altta).
> Yeni oturuma başlayan ajan: önce en üstteki kaydı oku, "Sıradaki adım"dan devam et.

---

## 2026-10-25 · claude · Planlama ve ortak hafıza kurulumu

**Yapılanlar**
- Kullanıcının isteği analiz edildi, mimari ve yol haritası kullanıcıya sunuldu (sohbette).
- Kullanıcı daha ayrıntılı plan ve Claude + Codex için ortak hafıza istedi. Oluşturulan dosyalar:
  `AGENTS.md`, `CLAUDE.md`, `docs/PLAN.md`, `docs/DECISIONS.md`, `docs/PROGRESS.md`, `docs/HANDOFF.md`, `docs/CHANNELS.md`.
- Git kuruldu ve GitHub'a bağlandı: https://github.com/Vanta-exe/aile-tv (public). İlk commit kullanıcının açtığı README commit'inin üstüne atıldı. `.gitignore` gizli dosyaları (local.properties, keystore, imza anahtarı) dışarıda tutuyor.
- Ortak oturum skill'leri yazıldı: `devral` (oturum başı) ve `devret` (oturum sonu).
  Asıl metin `.agents/skills/` (Codex buradan okur), `.claude/skills/` içindekiler oraya yönlendiren ince kopyalar. Bkz. ADR-014.
  Claude'da `/devral` ile, Codex'te `$devral` ile çağrılır. **İkisi de gerçek bir oturumda henüz denenmedi.**
- Henüz **hiç kod yazılmadı** (kullanıcının açık isteği: önce plan, anlaşınca adım adım kod).

**Kullanıcı hakkında bilinmesi gerekenler**
- Türkçe konuşuyor. Teknik kararları ajana bırakıyor ama önce açıklama/gerekçe istiyor.
- Hedef kitle: kullanıcının annesi ve babası (teknik bilgisi yok). Her UI kararı bu gözle verilmeli.
- Kullanıcı eve sık gidemiyor → uzaktan bakım kritik.

**Açık sorular (kullanıcıdan cevap bekleniyor — F0)**
1. TV'nin marka/modeli ve Android sürümü? Google TV mi, düz Android TV mi, yoksa ayrı bir Android TV kutusu mu?
2. ~~GitHub hesabı~~ ✅ (Vanta-exe/aile-tv). Telegram kullanıyor mu?
3. `docs/CHANNELS.md`'deki 20 kanallık taslak liste ve numaralar uygun mu?
4. Evdeki internet bağlantısının hızı yaklaşık ne? (HD yayın için ≥10 Mbps önerilir)

**Sıradaki adım**
- Kullanıcı sorulara cevap verince: F0.1–F0.3'ü kapat, F0.4 (kanal kaynak araştırması) başlat.
- F0.4'te her kanal için resmi canlı yayın sayfası incelenir; tür (direkt/token/DRM/geo) `CHANNELS.md`'ye yazılır. **URL uydurma; doğrulanmamış link yazma.**

**Doğrulanmamış / dikkat**
- PLAN.md'deki Google TV launcher ADB komutları genel bilgidir; model özelinde test edilmeden uygulanmamalı.
- Kütüphane sürümleri bilerek sabitlenmedi; F1.1'de güncel kararlı sürümler seçilip version catalog'a yazılacak.

---

### Şablon
```
## YYYY-AA-GG · (claude|codex) · Kısa başlık

**Yapılanlar**
-
**Yarım kalanlar / doğrulanmayanlar**
-
**Alınan kararlar** (DECISIONS.md'ye işlendi mi?)
-
**Açık sorular**
-
**Sıradaki adım**
-
```
