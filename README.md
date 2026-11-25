# c21_paraflow_diff — Bundle-quality regression against golden product bundles

> **Standalone.** Split from the [KutAI](https://github.com/msakirc/kutai) monorepo (`packages/c21_paraflow_diff/`), where it is developed. It does not need the KutAI app; its test suite passes with only its declared dependencies installed.

[English](#english) · [Türkçe](#türkçe)

<a id="english"></a>

## Purpose

An i2p mission produces a product bundle: charter, personas, PRD, user flow, screen
plans, screens, light/dark style guides and design tokens. This package compares a
mission workspace with a reference ("golden") bundle for the same product archetype
and scores three things — **coverage** (are the artifacts there), **coherence** (do
their sections have the expected shape) and **design fitness** (do the design tokens
cover the axes the style guide implies) — into one coarse verdict. It is rule-based and cheap so it can sweep every opted-in
mission regularly.

**It does NOT** judge prose quality, call a model, or block a mission. Its job is to
spot *missing whole artifacts* and gross shape drift; thresholds are deliberately
generous.

## Install

```bash
pip install "c21_paraflow_diff @ git+https://github.com/msakirc/c21-paraflow-diff.git"
```

Requires Python 3.10+. Sibling KutAI packages are installed from their own repositories.

## Public API

```python
from c21_paraflow_diff import diff_bundle, load_golden, KNOWN_ARCHETYPES

report = diff_bundle("workspace/mission_57/", "truthrate")
# -> {"archetype", "mission_workspace_path", "goldens_root",
#     "coverage", "coherence", "design_fitness", "gaps",
#     "score", "presence_score", "coherence_score", "design_score",
#     "verdict"}            # "paraflow_par" (>= 0.85) | "paraflow_partial" (>= 0.50) | "paraflow_gap"
```

A missing workspace is not an error: every artifact counts as a gap. An unknown
archetype raises `GoldenNotFoundError` (a `KeyError`).

## Dependencies

- Standard library only (Markdown headings are scanned as `^## ` lines, tokens read
  as JSON).
- Caller: Mr. Roboto's `verify_against_paraflow_goldens` mechanical action, run by a
  weekly General Beckman cron job. It visits every `mission_<id>/` that contains a
  `.paraflow_archetype` marker file (opt-in; no marker = skipped) and persists the
  drift trend to `paraflow_diff_log`.

## Gotchas

- Artifact paths are resolved through an alias table (`_ALIASES` in
  `diff_bundle.py`) — e.g. `prd.md` or `PRD.md`, `screen_plans/` or
  `.screen_plans/`. The first alias that exists wins. When the i2p workflow renames an
  output, add the new name there or coverage silently drops.
- `DEFAULT_GOLDENS_ROOT` is `tests/goldens/paraflow/` resolved **relative to the repo
  checkout** (four parents up from the module). A non-editable install must pass
  `goldens_root=` explicitly.
- The golden bundles are fixtures from a separate project and are **excluded from the
  public export** (`scripts/publish/manifest.toml`); there the package tests that need
  them cannot run.

## Tests

```powershell
python -m pytest packages/c21_paraflow_diff/tests
```

## In KutAI

A weekly General Beckman cron runs Mr. Roboto's `verify_against_paraflow_goldens` action, which calls `diff_bundle` on every opted-in mission workspace and logs the drift trend. Source: [`packages/c21_paraflow_diff/`](https://github.com/msakirc/kutai/tree/main/packages/c21_paraflow_diff) in the [KutAI monorepo](https://github.com/msakirc/kutai), where this package is developed.

## Status and limitations

- Standalone: it does not import the KutAI app, and its tests pass with only its declared dependencies.
- Version 0.1.0, part of a personal research project. Developed and tested on Windows 11 with Python 3.10; other platforms are untested.
- The golden bundles are not part of the public repos. Bring your own and pass `goldens_root=`; `KNOWN_ARCHETYPES` is currently just `truthrate`.

## License

MIT. See [LICENSE](https://github.com/msakirc/kutai/blob/main/LICENSE).

---
<a id="türkçe"></a>
## Türkçe

### Amaç

Bir i2p görevi bir ürün paketi üretir: tüzük, personalar, PRD, kullanıcı akışı, ekran
planları, ekranlar, açık/koyu stil kılavuzları ve tasarım belirteçleri. Bu paket bir
görev çalışma alanını aynı ürün arketipinin referans ("altın") paketiyle karşılaştırır
ve üç şeyi — **kapsam** (yapıtlar var mı), **tutarlılık** (bölümleri beklenen biçimde
mi) ve **tasarım uygunluğu** (tasarım belirteçleri stil kılavuzunun gerektirdiği
eksenleri karşılıyor mu) — tek, kaba bir karara puanlar. Kural tabanlı ve ucuzdur; haftalık bir zamanlanmış iş,
`.paraflow_archetype` işaret dosyası olan her görevi tarar.

Metin kalitesini **değerlendirmez**, model çağırmaz ve görevi engellemez. Görevi
*tamamen eksik yapıtları* ve kaba biçim kaymasını yakalamaktır.

### Kurulum

```bash
pip install "c21_paraflow_diff @ git+https://github.com/msakirc/c21-paraflow-diff.git"
```

Python 3.10+ gerekir. Kardeş KutAI paketleri kendi depolarından kurulur.

### Genel API

```python
from c21_paraflow_diff import diff_bundle

report = diff_bundle("workspace/mission_57/", "truthrate")
report["verdict"]     # "paraflow_par" | "paraflow_partial" | "paraflow_gap"
```

### Dikkat Edilecekler

- Yapıt yolları bir takma ad tablosundan (`_ALIASES`) çözülür; i2p bir çıktının adını
  değiştirirse yeni ad oraya eklenmeli, yoksa kapsam sessizce düşer.
- Altın paketler ayrı bir projeden gelen test verileridir ve **herkese açık dışa
  aktarımın dışında** tutulur.

### Testler

```powershell
python -m pytest packages/c21_paraflow_diff/tests
```

### KutAI içindeki yeri

Haftalık bir General Beckman cron'u Mr. Roboto'nun `verify_against_paraflow_goldens` eylemini çalıştırır; bu eylem işaretli her görev çalışma alanı için `diff_bundle` çağırır ve sapma eğilimini kaydeder. Kaynak: [KutAI monorepo](https://github.com/msakirc/kutai) içinde [`packages/c21_paraflow_diff/`](https://github.com/msakirc/kutai/tree/main/packages/c21_paraflow_diff); paket orada geliştirilir.

### Durum ve sınırlamalar

- Bağımsız: KutAI uygulamasını import etmez; testleri yalnızca bildirilen bağımlılıklarla geçer.
- Sürüm 0.1.0, kişisel bir araştırma projesinin parçası. Windows 11 ve Python 3.10 üzerinde geliştirilip test edildi; diğer platformlar denenmedi.
- Altın paketler herkese açık depolarda yok. Kendi paketinizi getirip `goldens_root=` verin; `KNOWN_ARCHETYPES` şu an yalnızca `truthrate`.

### Lisans

MIT. [LICENSE](https://github.com/msakirc/kutai/blob/main/LICENSE) dosyasına bakın.
