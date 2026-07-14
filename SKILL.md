---
name: prompt_generator
description: "Kullanıcının ihtiyacına göre AI modellerine, iş süreçlerine ve kod/test görevlerine yönelik çalışmaya hazır promptlar ve loop (otonom döngü) tasarımları oluştur. Trigger: 'prompt oluştur', 'prompt yaz', 'prompt gerek', 'nasıl sor', 'AI''ya ne desen', 'test prompt', 'code prompt', 'loop kur', 'loop prompt', 'otonom akış', 'self-correcting agent', 'agent loop', veya belirli bir görev için etkili bir prompt ya da kendi kendini denetleyen bir iş akışı gerektiğinde. Teknik (API, kod analizi), iş (sales, HR, operasyon), ve creative (yazı, tasarım) promptlar dahil."
---

# Prompt_Generator

**Versiyon**: 1.1 | **Sahip**: Orkestrix | **Repo**: [Prompt_Generator](https://github.com/Orkestrix-ai/Prompt_Generator)

Etkili promptlar oluşturmak sanat ve bilimdir. Bu skill, kullanıcının amaçlarını anlayıp, Claude (veya diğer AI modelleri) için optimize edilmiş, çalışmaya hazır promptlar üretir.

Bu skill iki modda çalışır:

| Mod | Ne üretir? | Ne zaman? |
|---|---|---|
| **Single-shot prompt** | Tek seferlik, kopyala-yapıştır prompt metni | Görev tek geçişte bitiyorsa |
| **Loop prompt (loop mühendisliği)** | Kendi kendini yöneten, denetleyen, düzelten otonom döngü tasarımı | Görev doğrulanabilir bir hedefe sahipse ve tek geçişte güvenilir bitmiyorsa |

---

## Loop Mühendisliği (Loop Engineering)

**Tanım**: En basit tanımıyla loop mühendisliği; yapay zekâya tek seferlik talimatlar (prompt) vermek yerine, yapay zekânın kendi kendini yönettiği, denetlediği, hatalarını düzeltip hedefe ulaşana kadar döndüğü otonom sistemleri ve iş akışlarını tasarlama disiplinidir.

Prompt mühendisliği **talimatı** optimize eder; loop mühendisliği **süreci** optimize eder. Bir loop'ta prompt artık çıktının kendisi değil, döngünün bir bileşenidir.

### Prompt vs. Loop

```
PROMPT (tek seferlik)
Talimat → Çıktı → [insan denetler] → [insan düzeltir]

LOOP (otonom)
Hedef → Plan → Eylem → Doğrulama (self-check) → Hata mı? → Düzelt → tekrar Eylem
                            ↓ Hayır
                     Çıkış koşulu (exit) → Sonuç
```

### Bir loop'un zorunlu 6 bileşeni

Loop tasarlarken bu altısı **eksiksiz** tanımlanmalıdır. Eksik olan her bileşen ya sonsuz döngü ya da sessiz hata üretir.

1. **Hedef (Goal)** — Tek cümlelik, ölçülebilir amaç. "İyi kod yaz" değil, "tüm testler yeşil olana kadar auth modülünü düzelt".
2. **Doğrulama sinyali (Verifier)** — Loop'un başarıyı kendi kendine ölçtüğü **nesnel** kaynak: test çıktısı, lint, type-check, build log, DB assertion, HTTP status, schema validation. Doğrulayıcısı olmayan görev loop'a uygun değildir — LLM'in "bence oldu" demesi doğrulama değildir.
3. **Çıkış koşulu (Exit criteria)** — Loop ne zaman **başarıyla** durur? Ne zaman **başarısız** kabul edip durur?
4. **Iterasyon limiti + escalation** — Maksimum tur sayısı (ör. 5) ve limit dolduğunda ne yapılacağı (insana devret, son durumu raporla, rollback).
5. **Durum/hafıza (State)** — Turlar arasında taşınan bağlam: denenen çözümler, alınan hatalar, değişen dosyalar. Aynı hatayı iki kez denemeyi engelleyen şey budur.
6. **Eylem alanı (Tools/Scope)** — Loop'un neye dokunabileceği ve **kesinlikle dokunamayacağı** sınır (prod DB'ye yazma yok, migration çalıştırma yok, dosya silme yok).

### Loop Prompt Şablonu

```
ROL: Sen [X] görevini hedefe ulaşana kadar otonom yürüten bir agent'sın.

HEDEF:
[Tek cümle, ölçülebilir]

BAŞARI TANIMI (Verifier):
[Nesnel komut/sinyal, ör: `npm test` çıktısında 0 failing]

DÖNGÜ:
1. PLAN    → Mevcut duruma bak, bir sonraki tek adımı seç ve gerekçelendir.
2. EYLEM   → Sadece o adımı uygula.
3. DOĞRULA → [verifier komutunu] çalıştır. Çıktıyı olduğu gibi oku, yorum katma.
4. KARAR   →
   - Başarı koşulu sağlandıysa → DUR ve final raporu ver.
   - Sağlanmadıysa → Hatanın kök nedenini yaz, denenmiş çözümler listesine ekle, 1'e dön.

STATE (her turda güncelle ve taşı):
- Tur no:
- Son hata:
- Denenmiş ve işe yaramamış çözümler: [tekrar deneme]
- Değiştirilen dosyalar:

KISITLAR:
- Maksimum [N] tur. Limitte durur, insana devredersin.
- [Dokunulmaz alanlar: prod, secrets, migration, silme işlemleri]
- Aynı çözümü iki kez deneme.
- Doğrulama çıktısını uydurma; komutu gerçekten çalıştır.

ESCALATION:
Limit dolarsa veya aynı hata 2 turdur değişmiyorsa: dur, "TAKILDIM" yaz,
son hatayı + denenenleri + önerini raporla.

FINAL ÇIKTI FORMATI:
[Beklenen rapor formatı]
```

### Loop Anti-Patterns

❌ **Doğrulayıcısız loop** — "Beğenene kadar iyileştir." Model kendi çıktısını beğenir, döngü anlamsız döner.
❌ **Limitsiz loop** — Çıkış koşulu yoksa token yakar, sonsuza gider.
❌ **State'siz loop** — Her tur sıfırdan başlar, aynı hatalı çözümü tekrar dener.
❌ **Yıkıcı eylem alanı** — Loop'a geri alınamayan yetki (prod delete, force push) vermek.
❌ **Her şeyi loop'a sokmak** — Blog yazısı, tek seferlik SQL, basit refactor loop istemez. Loop maliyeti (token + risk) ancak doğrulanabilir + tekrarlı görevlerde geri döner.

### Loop ne zaman DOĞRU araçtır?

✅ Nesnel bir doğrulayıcı var (test, lint, build, schema, API response)
✅ Görev tek geçişte %100 doğru bitmiyor, iterasyon gerekiyor
✅ Hata sinyali makine tarafından okunabilir
✅ Yanlış adımın maliyeti geri alınabilir (branch, staging, dry-run)

Bu dördü sağlanmıyorsa **single-shot prompt** üret, loop üretme.

---

## Kullanım Durumları (Use Cases)

| Tür | Kapsam | Mod |
|---|---|---|
| **Teknik** | Kod yazma, review/debug, test, SQL, DevOps | Single-shot |
| **İş** | Sales, HR, finans, pazarlama, contract review | Single-shot |
| **Creative** | İçerik, tasarım briefi, UX copy | Single-shot |
| **Loop** | Doğrulayıcısı olan tekrarlı görevler: test/lint/build yeşile dönene kadar debug, schema geçene kadar veri temizleme, retry + self-correction akışları | Loop |

Kural: doğrulayıcı (test/lint/build/schema/HTTP) yoksa → single-shot. Varsa ve tek geçişte bitmiyorsa → loop.

## Anti-Patterns (Kaçınılması Gerekenler)

❌ **Kötü**: "Bana bir prompt yaz"
- Amaç belirsiz
- Context yok
- Beklentiler tanımlanmamış

✅ **İyi**: "Supabase ile auth kuran React component'ı yazıp test etmesini söylemek için bir prompt yaz. TypeScript kullan, error handling ekle"

---

## Prompt Oluşturma Süreci

### 0. Mod Seçimi (ilk karar)

Önce sor/karar ver: **single-shot mu, loop mu?**
- Nesnel doğrulayıcı var mı? → yoksa single-shot.
- Tek geçişte biter mi? → biterse single-shot.
- İkisi de "hayır" ise → loop tasarla, Loop Prompt Şablonu'nu kullan.

### 1. Gereksinim Toplama

Eğer kullanıcı eksik bilgi sağlamışsa, sor:

- **Görev**: Tam olarak ne yapılsın?
- **Context**: Hangi teknolojiler/frameworks/tools?
- **Başarı Kriterleri**: Ne "başarı" demek bu görev için? (loop ise: hangi komut/sinyal bunu ölçüyor?)
- **Constraints**: Varsa sınırlamalar (dosya boyutu, hız, uyum, vb)?
- **Tone/Style**: Resmi mi, casual mi, teknik mi?
- **(Loop ise)** Iterasyon limiti, dokunulmaz alanlar, escalation hedefi?

### 2. Prompt Tasarımı

İyi bir prompt:

- **Net ve öz** - amacı net belirtir, uzun değildir
- **Context sağlar** - arkaplan bilgisini verir
- **Rol/Perspektif** - "sen X rolündesin" veya "şöyle davran"
- **Format bekler** - output formatını açıkça söyler
- **Örnek sağlar** - mümkünse input/output örneği
- **Constraints listeler** - neleri yapmasın?

İyi bir loop, bunlara ek olarak: **verifier + exit + limit + state + scope** içerir.

### 3. Optimizasyon

Prompt'u test et:
- Bir iki kez çalıştır, sonuçları gözlemle
- Amaca ulaşıyor mu? Değilse neden?
- Çok uzun mu? Kısalt
- Belirsiz mi? Spesifikleş

Loop'u test et:
- İlk turda doğru duruyor mu (false positive exit yok mu)?
- Kasıtlı bir hata enjekte et — loop yakalıyor mu?
- Limit gerçekten devreye giriyor mu?
- State turlar arasında taşınıyor mu, aynı çözümü tekrar deniyor mu?

---

## Template Örnekleri

### Teknik Prompt Şablonu

```
Görev: [Tam açıklama]

Teknoloji: [Stack: React + Supabase + TypeScript, vb]

Requirements:
- [Gereksinim 1]
- [Gereksinim 2]
- [Gereksinim 3]

Output Format: [JSON/Markdown/Code snippet vb]

Constraints:
- [Kısıt 1]
- [Kısıt 2]

Example:
Input: [örnek]
Output: [beklenen çıkış]
```

### İş Prompt Şablonu

```
Kontext: [Durum açıklaması]

Hedef: [Ne başarılmak isteniyor?]

Audience: [Kime hitap ediyor?]

Tone: [Resmi/Friendly/Persuasive]

Key Points: [Ana noktalar]

Success Criteria: [Başarı ölçütü]
```

### Creative Prompt Şablonu

```
Konsept: [Temel fikir]

Target Audience: [Kimler?]

Style/Tone: [Yazı stili]

Length: [Kaç kelime/dakika?]

Constraints: [Neleri içermemeli?]

Example of desired output: [Benzeri örnek]
```

### Loop Şablonu

→ Yukarıdaki **Loop Prompt Şablonu** bölümünü kullan.

---

## Tips & Best Practices

✅ **Spesifik ol** - "prompt yaz" değil, "Supabase RLS kuran prompt yaz"
✅ **Örnek ver** - İyi bir örnek bazen 1000 kelimeden daha değerlidir
✅ **Constraints söyle** - "kısaca", "detaylı", "3 seçenek", vb
✅ **Role ver** - "sen bir senior engineer gibi..."
✅ **Format tanımla** - JSON, Markdown, pseudo-code, vb
✅ **Loop'ta doğrulayıcıyı dışarıdan al** - modelin öz-değerlendirmesi doğrulayıcı değildir
✅ **Loop'ta her zaman limit + escalation yaz** - takılan loop, hatalı loop'tan pahalıdır

❌ Vakit kaybetme - eğer prompt hala işe yaramıyorsa, direkt geri bildir ve itera yap

---

## Çıktı

Output **her zaman** çalışmaya hazır, direkt kopyala-yapıştır prompt metnidir.

Hiçbir açıklama ekleme, sadece prompt ver.

Eğer prompt çok kompleks ise, sections ile böl (ör: system prompt + user prompt).

Loop üretiliyorsa çıktı: tek blok halinde loop prompt'u (rol + hedef + verifier + döngü + state + kısıtlar + escalation + final format).

---

## Gerçek Örnekler (Real Examples)

### Örnek 1: Supabase RLS Prompt
**İstek**: "Supabase RLS kuran React hook prompt yaz"

**Çıktı**:
```
Supabase Row Level Security (RLS) politikaları ile güvenli veri erişimi sağlayan bir React hook yaz.

Gereksinimler:
- TypeScript
- Supabase JS client
- RLS policies set up (users tablosu, tenant_id ile)
- Error handling + loading states

Kod şu adımları takip etsin:
1. useEffect'te Supabase client'ı init et
2. İlgili politikaları kontrol et (WHERE auth.uid() = user_id)
3. Unauthorized hatası varsa user-friendly mesaj göster
4. Loading/error states yönet

Output format: Hazır kopyala-yapıştır React hook kodu
```

### Örnek 2: Sequential Approval Chain Prompt
**İstek**: "Sıralı onay akışını (approval workflow) handle eden prompt"

**Çıktı**:
```
Sequential approval chain'i implement eden prompt.

Gereksinimler:
- Supabase DB structure (approvals table, status enum)
- React form components
- TypeScript strict mode
- Notification system (in-app only, no email)

Flow:
1. Document upload → initial approval step (step_1)
2. Step başarısızsa → rejection, user rejected step'ten yeniden başlayabilir
3. Adımlar sırayla gitmeli (step_1 → step_2 → step_3)
4. Her adımda file/photo attachment mandatory

Output: Sequential state machine logic (TypeScript)
```

### Örnek 3: Türkçe Onboarding Email Prompt
**İstek**: "Türkçe SaaS signup sonrası welcome email prompt yaz, friendly tone"

**Çıktı**:
```
Yeni kullanıcılara hoş geldiniz diyen, Türkçe welcome email yaz.

Context:
- B2B SaaS müşterisi (KOBİ yöneticisi)
- İlk kez SaaS kullanıyor olabilir
- 24 saat içinde ilk core aksiyonu tamamlaması hedefleniyor

Tone: Samimi, yardımcı, teknik olmayan dil

Email şunları içermeli:
- Hoş geldin
- İlk adımlar (3 madde)
- Support contact (email)
- Call-to-action (ilk aksiyonu başlat)

Length: 150-200 kelime
Output: HTML email template
```

### Örnek 4 (LOOP): Test-yeşile-dönene-kadar debug loop
**İstek**: "Modül testleri kırmızı, kendi kendini düzelten bir loop kur"

**Çıktı**:
```
ROL: Sen bu repoda testleri yeşile döndürene kadar otonom çalışan bir debug agent'sın.

HEDEF:
`src/approvals/**` altındaki tüm testler geçene kadar üretim kodunu düzelt.

BAŞARI TANIMI (Verifier):
`npm test -- src/approvals` çıktısında 0 failing, 0 error.
Bu komutun gerçek çıktısı tek doğruluk kaynağıdır. Çıktı uydurma.

DÖNGÜ:
1. PLAN    → Son hata mesajını oku, kök nedeni tek cümlede yaz, tek bir düzeltme seç.
2. EYLEM   → Sadece o düzeltmeyi uygula.
3. DOĞRULA → `npm test -- src/approvals` çalıştır, çıktıyı olduğu gibi al.
4. KARAR   → 0 failing ise DUR. Değilse STATE'i güncelle ve 1'e dön.

STATE (her turda yazdır):
- Tur no:
- Failing test sayısı (önceki → şimdiki):
- Son hata:
- Denenmiş ve işe yaramamış düzeltmeler:
- Değiştirilen dosyalar:

KISITLAR:
- Maksimum 5 tur.
- Testleri değiştirme, silme veya skip etme. Sadece üretim kodunu düzelt.
- Supabase migration çalıştırma, .env dosyalarına dokunma.
- Aynı düzeltmeyi iki kez deneme.

ESCALATION:
5. tur dolarsa veya failing sayısı 2 tur üst üste azalmazsa: DUR, "TAKILDIM" yaz,
son hatayı + denenen düzeltmeleri + kendi hipotezini raporla.

FINAL ÇIKTI:
- Sonuç: BAŞARILI / TAKILDIM
- Tur sayısı
- Değiştirilen dosyalar + diff özeti
- Kök neden (1 paragraf)
```

---

## Troubleshooting

### Problem: Prompt çıktısı hâlâ belirsiz
**Çözüm**:
- Constraints'i daha spesifik yaz
- Bir örnek (good example) ekle
- "Şöyle davran" yerine "şöyle yap" diye komut ver

### Problem: Çıktı çok uzun
**Çözüm**:
- "kısaca" veya "bullet points" diye söyle
- "max 200 kelime" gibi limit koy
- İstediğin format'ı açıkça belirt

### Problem: Output formatı yanlış
**Çözüm**:
- "Çıktı XYZ formatında olmalı" diye başla
- Örnek göster (Input → Expected Output)
- JSON istersan `{"key": "value"}` şeklinde örnek ver

### Problem: Loop sonsuza dönüyor / durmuyor
**Çözüm**: Çıkış koşulu nesnel değil. Verifier'ı bir komuta bağla ve iterasyon limiti + escalation ekle.

### Problem: Loop "başardım" diyor ama başarmamış
**Çözüm**: Model kendi çıktısını doğruluyor. Doğrulamayı dışarıdaki bir sinyale (test/lint/build/HTTP) taşı ve "çıktıyı uydurma, komutu gerçekten çalıştır" kısıtını ekle.

### Problem: Loop aynı hatayı tekrar tekrar deniyor
**Çözüm**: State eksik. "Denenmiş ve işe yaramamış çözümler" listesini her turda taşıt ve tekrarı yasakla.

---

## Workflow Özeti

```
Kullanıcı İsteği
    ↓
Mod Seçimi (Single-shot mu? Loop mu?)
    ↓                          ↓
Gereksinim Toplama        Loop Tasarımı
(Context, Constraints,    (Hedef, Verifier, Exit,
 Success Criteria)         Limit, State, Scope)
    ↓                          ↓
Prompt Tasarımı           Loop Prompt Şablonu
    ↓                          ↓
Output (Çalışmaya hazır prompt / loop metni)
    ↓
[Opsiyonel] Test & Iterate
```

---

## Compatibility & Dependencies

**Gereksinimler:**
- Claude API (claude-opus, claude-sonnet, vb)
- Veya claude.ai web interface
- Türkçe dil desteği (UTF-8)
- Loop modu için: agent runtime'ı olan bir ortam (Claude Code, n8n, kendi orchestrator'ın)

**Dikkat:**
- Bu skill loop'u **tasarlar ve prompt'unu üretir**; loop'u çalıştıran runtime'ı kurmaz
- n8n orchestration bu skill'de yer almıyor (loop prompt'u n8n'e beslenebilir)
- Web search veya API integration prompt'lar hakkında bilgi verir ama implement etmez
