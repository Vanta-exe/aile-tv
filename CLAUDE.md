# CLAUDE.md

Bu projenin tek doğruluk kaynağı `AGENTS.md` dosyasıdır (Codex ile ortak kullanılır). Aşağıdaki satır onu içe aktarır:

@AGENTS.md

## Claude'a özel notlar
- Proje bilgisini kendi özel hafızana (`~/.claude/...memory`) değil, `docs/` altındaki ortak dosyalara yaz. Codex onları görebilir, senin özel hafızanı göremez.
- Oturum sonunda `docs/HANDOFF.md`'ye kayıt eklemeden işi bitmiş sayma.
