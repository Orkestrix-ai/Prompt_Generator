# Prompt_Generator

[![Orkestrix](https://img.shields.io/badge/Author-Orkestrix-blue)](https://github.com/Orkestrix-ai)
[![Version](https://img.shields.io/badge/Version-1.1-green)]()
[![License](https://img.shields.io/badge/License-MIT-yellow)]()

Çalışmaya hazır **promptlar** ve kendi kendini denetleyen **otonom loop'lar** üreten Claude skill'i.

> Prompt mühendisliği talimatı optimize eder. **Loop mühendisliği süreci optimize eder.**

---

## 🎯 Ne İşe Yarar?

| Tür | Kapsam | Mod |
|---|---|---|
| **Teknik** | Kod yazma, review/debug, test, SQL, DevOps | Single-shot |
| **İş** | Sales, HR, finans, pazarlama, contract review | Single-shot |
| **Creative** | İçerik, tasarım briefi, UX copy | Single-shot |
| **Loop** | Doğrulayıcısı olan tekrarlı görevler: test/lint/build yeşile dönene kadar debug, schema geçene kadar veri temizleme, retry + self-correction akışları | Loop |

**Seçim kuralı**: nesnel doğrulayıcı (test/lint/build/schema/HTTP) yoksa → single-shot. Varsa ve görev tek geçişte bitmiyorsa → loop.

---

## 🔁 Loop Mühendisliği

**Tanım**: En basit tanımıyla loop mühendisliği; yapay zekâya tek seferlik talimatlar (prompt) vermek yerine, yapay zekânın kendi kendini yönettiği, denetlediği, hatalarını düzeltip hedefe ulaşana kadar döndüğü otonom sistemleri ve iş akışlarını tasarlama disiplinidir.

```
PROMPT   Talimat → Çıktı → [insan denetler] → [insan düzeltir]

LOOP     Hedef → Plan → Eylem → Doğrula → Hata mı? → Düzelt ↺
                                    ↓ Hayır
                              Çıkış koşulu → Sonuç
```

### Bir loop'un zorunlu 6 bileşeni

| # | Bileşen | Neden zorunlu? |
|---|---|---|
| 1 | **Hedef** | Ölçülebilir tek cümle. "İyi kod yaz" değil. |
| 2 | **Doğrulayıcı (Verifier)** | Nesnel sinyal: test, lint, build, schema, HTTP. Modelin "bence oldu" demesi doğrulama değildir. |
| 3 | **Çıkış koşulu** | Ne zaman başarıyla, ne zaman başarısız durur? |
| 4 | **Limit + escalation** | Max tur sayısı ve dolduğunda ne olacağı (insana devret). |
| 5 | **State** | Denenen ve işe yaramayan çözümler taşınır → aynı hata tekrarlanmaz. |
| 6 | **Scope** | Dokunulmaz alanlar (prod, secrets, migration, silme). |

### Loop anti-pattern'leri

```
❌ Doğrulayıcısız loop  → model kendi çıktısını beğenir, döngü anlamsız
❌ Limitsiz loop        → token yakar, sonsuza gider
❌ State'siz loop       → aynı hatalı çözümü tekrar dener
❌ Yıkıcı scope         → geri alınamayan yetki (prod delete, force push)
❌ Gereksiz loop        → tek seferlik görev loop istemez
```

---

## 🚀 Hızlı Başlangıç

```
- "prompt oluştur: [görev açıklaması]"
- "prompt yaz: RLS koruması kuran React hook"
- "loop kur: testler yeşile dönene kadar debug etsin"
- "otonom akış: schema validation geçene kadar veriyi temizlesin"
```

---

## 📋 5 Adımda İyi Prompt

1. **Görev** — Ne yapılacak? (1-2 cümle)
2. **Context** — Stack, kütüphaneler, kısıtlar
3. **Başarı kriteri** — Ne "başarı" sayılır? *(Loop ise: bunu hangi komut ölçüyor?)*
4. **Format** — JSON / TypeScript / Markdown / plain text
5. **Constraints** — Neleri yapmasın?

### ❌ Kaçın

```
❌ "Bana bir prompt yaz"        (belirsiz)
❌ "Kod yaz"                     (context yok)
❌ "İyi bir sales pitch"         (success criteria yok)
❌ "Beğenene kadar iyileştir"    (loop'ta doğrulayıcı yok)

✅ "Auth kuran TypeScript prompt, friendly tone, max 150 kelime"
✅ "Loop: `npm test` 0 failing olana kadar düzelt, max 5 tur, testlere dokunma"
```

---

## 📚 Örnekler

### 1. Single-shot — RLS Hook

**İstek**: "RLS korumalı veri erişim hook'u, React + TypeScript, loading/error states"

**Çıktı** (prompt):
```
RLS korumalı veri erişim hook'u yaz.

Context:
- users table: id, email, tenant_id
- RLS: WHERE auth.uid() = user_id
- React + TypeScript
- Unauthorized → user-friendly error message

Hook:
- useDataQuery(table, conditions)
- Loading, error, data states
- Retry logic
- Type-safe

Output: Kopyala-yapıştır React hook
```

### 2. Loop — Test yeşile dönene kadar debug

**İstek**: "Modül testleri kırmızı, kendi kendini düzelten bir loop kur"

**Çıktı** (loop prompt):
```
ROL: Testleri yeşile döndürene kadar otonom çalışan debug agent'sın.

HEDEF: [modül] altındaki tüm testler geçene kadar üretim kodunu düzelt.

BAŞARI TANIMI (Verifier):
`npm test -- [modül]` çıktısında 0 failing. Bu komutun gerçek çıktısı
tek doğruluk kaynağıdır. Çıktı uydurma.

DÖNGÜ:
1. PLAN    → Son hatayı oku, kök nedeni tek cümlede yaz, tek düzeltme seç.
2. EYLEM   → Sadece o düzeltmeyi uygula.
3. DOĞRULA → Verifier komutunu çalıştır, çıktıyı olduğu gibi al.
4. KARAR   → 0 failing ise DUR. Değilse STATE'i güncelle, 1'e dön.

STATE (her turda yazdır):
- Tur no / Failing sayısı (önceki → şimdiki) / Son hata
- Denenmiş ve işe yaramamış düzeltmeler
- Değiştirilen dosyalar

KISITLAR:
- Max 5 tur. Testleri değiştirme, silme, skip etme.
- Migration çalıştırma, .env dosyalarına dokunma.
- Aynı düzeltmeyi iki kez deneme.

ESCALATION:
Limit dolarsa veya failing sayısı 2 tur azalmazsa: DUR, "TAKILDIM" yaz,
son hatayı + denenenleri + hipotezini raporla.

FINAL ÇIKTI:
Sonuç (BAŞARILI/TAKILDIM) · Tur sayısı · Değişen dosyalar · Kök neden
```

---

## 📦 Kurulum

### Repo yapısı

```
Prompt_Generator/
├── SKILL.md      # Skill tanımı (frontmatter + tüm şablonlar/örnekler)
├── README.md     # Bu dosya
└── LICENSE       # MIT
```

### 1. Klonlama

```bash
git clone https://github.com/Orkestrix-ai/Prompt_Generator.git
cd Prompt_Generator
```

SSH ile:
```bash
git clone git@github.com:Orkestrix-ai/Prompt_Generator.git
```

### 2. Skill olarak kurma

**Claude Code (kullanıcı seviyesi — tüm projelerde aktif):**
```bash
mkdir -p ~/.claude/skills/prompt-generator
cp SKILL.md README.md ~/.claude/skills/prompt-generator/
```

**Claude Code (proje seviyesi — sadece bu repo'da aktif):**
```bash
mkdir -p .claude/skills/prompt-generator
cp SKILL.md README.md .claude/skills/prompt-generator/
```

Doğrudan klonlayarak da kurabilirsin:
```bash
git clone https://github.com/Orkestrix-ai/Prompt_Generator.git \
  ~/.claude/skills/prompt-generator
```

Kurulumu doğrula:
```bash
ls ~/.claude/skills/prompt-generator/SKILL.md
```
Claude'u yeniden başlat, ardından `"prompt oluştur: ..."` veya `"loop kur: ..."` yaz — skill tetiklenmeli.

**Claude.ai (web):** SKILL.md içeriğini Settings → Capabilities → Skills altından yükle.

### 3. Güncelleme

```bash
cd ~/.claude/skills/prompt-generator
git pull origin main
```

### 4. Katkı / Push (maintainer)

```bash
git checkout -b feature/loop-engineering
# SKILL.md ve/veya README.md üzerinde değişiklik yap
git add SKILL.md README.md
git commit -m "feat: loop mühendisliği modu ve loop prompt şablonu (v1.1)"
git push -u origin feature/loop-engineering
```
Ardından GitHub'da PR aç. Doğrudan `main`'e:
```bash
git add . && git commit -m "docs: v1.1" && git push origin main
```

> **Loop modu** için agent runtime'ı olan bir ortam gerekir (Claude Code, n8n, kendi orchestrator'ın). Bu skill loop'u **tasarlar ve prompt'unu üretir**; runtime'ı kurmaz.

---

## 🔧 Troubleshooting

| Problem | Çözüm |
|---|---|
| Prompt hâlâ belirsiz | Constraints ve örnek ekle |
| Çıktı çok uzun | "Kısaca" / "max 200 kelime" / bullet points |
| Format yanlış | Beklenen output'un örneğini göster |
| **Loop durmuyor** | Çıkış koşulu nesnel değil → verifier'ı komuta bağla, limit ekle |
| **Loop "başardım" diyor ama başarmamış** | Kendi çıktısını doğruluyor → doğrulamayı dış sinyale taşı |
| **Loop aynı hatayı tekrarlıyor** | State eksik → "denenmiş çözümler" listesini her turda taşıt |

---

## 📖 Referanslar

- **SKILL.md** — Detaylı dokümantasyon, şablonlar, tüm örnekler
- **Claude Docs** — [Prompting Guide](https://docs.anthropic.com)

---

## 🤝 Katkı & Destek

- **GitHub**: [@Orkestrix-ai](https://github.com/Orkestrix-ai) · [Issues](https://github.com/Orkestrix-ai/Prompt_Generator/issues)
- **Email**: orkestrix@gmail.com

## 📄 License

MIT License © 2026 Orkestrix

---

**Last Updated**: 2026-07-14
**Version**: 1.1
**Maintainer**: Orkestrix
