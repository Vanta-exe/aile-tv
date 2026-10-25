# İlerleme ve Görev Kilitleri

> Görev ayrıntıları ve kabul kriterleri: `PLAN.md` Bölüm 10.
> Durum: ⬜ başlanmadı · 🔒 üzerinde çalışılıyor (kim + tarih) · 🟨 kısmen / doğrulanmadı · ✅ bitti (doğrulandı) · ⛔ engelli (sebep yaz)
> Görevi almadan önce kilitle; bitince ✅ yap ve kilidi kaldır.

**Şu anki aşama:** F0 — Hazırlık
**Sıradaki iş:** F0.1–F0.3 için kullanıcıdan bilgi almak (HANDOFF.md'deki açık sorular)

## F0 — Hazırlık
| ID | Görev | Durum | Not |
|---|---|---|---|
| F0.1 | TV marka/model/Android sürümü | ⛔ | Kullanıcıdan bekleniyor |
| F0.2 | GitHub + Telegram hesapları, repo | 🟨 | Repo hazır: Vanta-exe/aile-tv · Telegram bekleniyor |
| F0.3 | Kanal listesi kesinleşmesi | ⛔ | Taslak `CHANNELS.md`'de, kullanıcı onayı bekleniyor |
| F0.4 | Kanal kaynak araştırması | ⬜ | F0.3'ten sonra |
| F0.5 | Geliştirme ortamı kurulumu | ⬜ | |

## F1 — Oynatıcı İskeleti
| ID | Görev | Durum | Not |
|---|---|---|---|
| F1.1 | Gradle projesi + paket yapısı + CI | ⬜ | |
| F1.2 | MainActivity tam ekran + manifest | ⬜ | |
| F1.3 | Sabit tek HLS kanalı oynatma | ⬜ | |
| F1.4 | Geri tuşu kilidi | ⬜ | |

## F2 — Kumanda ve Arayüz
| ID | Görev | Durum | Not |
|---|---|---|---|
| F2.1 | Modeller + gömülü varsayılan konfig | ⬜ | |
| F2.2 | RemoteKeyHandler + testler | ⬜ | |
| F2.3 | Kanal değiştirme + OSD + debounce | ⬜ | |
| F2.4 | ChannelListPanel | ⬜ | |
| F2.5 | Numara girişi | ⬜ | |
| F2.6 | Son kanal hafızası | ⬜ | |
| F2.7 | StatusOverlay + yükleniyor | ⬜ | |

## F3 — Uzaktan Konfig
| ID | Görev | Durum | Not |
|---|---|---|---|
| F3.1 | Anahtar/imza/doğrulama araçları | ⬜ | |
| F3.2 | publish-config.yml | ⬜ | |
| F3.3 | SignatureVerifier + testler | ⬜ | |
| F3.4 | ConfigRepository + Worker | ⬜ | |
| F3.5 | Konfig değişiminde yayını kesmeme | ⬜ | |

## F4 — Link Çözücü
| ID | Görev | Durum | Not |
|---|---|---|---|
| F4.1 | Şablon motoru + StepExecutor | ⬜ | |
| F4.2 | Ortak test vektörleri | ⬜ | |
| F4.3 | ResolvedUrlCache | ⬜ | |
| F4.4 | Gerçek kanal tarifleri | ⬜ | |

## F5 — Dayanıklılık
| ID | Görev | Durum | Not |
|---|---|---|---|
| F5.1 | RetryPolicy + durum makinesi | ⬜ | |
| F5.2 | PlaybackWatchdog | ⬜ | |
| F5.3 | Ağ izleme | ⬜ | |
| F5.4 | BehindLiveWindow + token yenileme | ⬜ | |

## F6 — Kiosk ve Kurulum
| ID | Görev | Durum | Not |
|---|---|---|---|
| F6.1 | HOME launcher doğrulama | ⬜ | |
| F6.2 | Yedek açılış (BOOT) | ⬜ | |
| F6.3 | AdminScreen | ⬜ | |
| F6.4 | Release imzalama + SETUP_TV.md | ⬜ | |

## F7 — İzleme ve Uzaktan Bakım
| ID | Görev | Durum | Not |
|---|---|---|---|
| F7.1 | healthcheck + Telegram | ⬜ | |
| F7.2 | ErrorReporter | ⬜ | |
| F7.3 | Tailscale + uzaktan ADB dokümanı | ⬜ | |
| F7.4 | BAKIM.md | ⬜ | |

## F8 — Saha Testi
| ID | Görev | Durum | Not |
|---|---|---|---|
| F8.1 | 1 hafta kullanım | ⬜ | |
| F8.2 | Düzeltmeler → v1.0 | ⬜ | |

## Ortak hafıza altyapısı
| ID | Görev | Durum | Not |
|---|---|---|---|
| M0.1 | AGENTS.md, CLAUDE.md, docs/ dosyaları | ✅ | claude 2026-10-25 |
| M0.2 | devral / devret ortak skill'leri | 🟨 | claude 2026-10-25 · yazıldı, Claude ve Codex'te gerçek oturumda denenmedi |
