# Data — prices and delegation scenarios

**English** · [Español](#español)

Machine-readable companions to the dossier. Prices are a snapshot of **30 October 2026**; they are not a live feed. The formulas are in [`../en/economics/01-token-accounting.md`](../en/economics/01-token-accounting.md) and [`../es/economia/01-contabilidad-de-tokens.md`](../es/economia/01-contabilidad-de-tokens.md).

## Files

| File | Content |
|---|---|
| [`prices-2026-10-30.csv`](prices-2026-10-30.csv) | Per-model list prices in USD per million tokens: input, output, cache read, short and long cache write |
| [`delegation-scenarios.csv`](delegation-scenarios.csv) | Net saving of a subagent for a fixed scenario, by main/sub model and remaining turns |

## Scenario assumptions

`delegation-scenarios.csv` uses:

- \(D = 100{,}000\) tokens kept out of the main thread.
- \(S = 30{,}000\) startup tokens, written to a 5-minute cache at \(1.25\times\).
- \(W = 100{,}000\) work tokens (input).
- \(O = 10{,}000\) output tokens.
- \(R = 2{,}000\) summary tokens returned to the main thread.
- Warm main cache; cache read at the model's published multiplier (0.05× for Opus/Sonnet 5.5).
- No compaction, no retries, no human review.

$$
\Delta = D \cdot T \cdot r \cdot p_{\text{in,main}}
\;-\;
\left[ (1.25S + W) \cdot p_{\text{in,sub}} + O \cdot p_{\text{out,sub}} + R \cdot p_{\text{in,main}} \right]
$$

## How to recompute

```python
# pip install pandas (or use the csv module)
import csv
prices = {
    row["model"]: row
    for row in csv.DictReader(open("prices-2026-10-30.csv"))
}

D, S, W, O, R = 100_000, 30_000, 100_000, 10_000, 2_000

def net(main, sub, T):
    pm, ps = prices[main], prices[sub]
    pin_m = float(pm["input_usd_per_mtok"])
    pin_s = float(ps["input_usd_per_mtok"])
    pout_s = float(ps["output_usd_per_mtok"])
    cr_m = float(pm["cache_read_usd_per_mtok"])
    save = D * T * (cr_m / 1_000_000)
    cost = ((1.25 * S + W) * pin_s + O * pout_s + R * pin_m) / 1_000_000
    return save, cost, save - cost

# Example: Opus main, Sonnet subagent, 25 turns left
print(net("Claude Opus 5.5", "Claude Sonnet 5.5", 25))
```

The CSV labels are the full model names (`Claude Opus 5.5`, `Claude Sonnet 5.5`, `GPT-6.1 Sol`, etc.), so the snippet works as written.

## Caveats

- **Subscription plans are not dollars.** Codex credits and Claude/OpenCode plan windows are quota, not API spend.
- **Tokenizers differ.** Claude 4.7+ produces ~30% more tokens for the same text.
- **Cache assumptions dominate.** A cold cache changes the result; the dossier uses a warm main cache as the conservative baseline.
- **Quality is excluded.** The CSV says nothing about whether the subagent succeeds; use cost per accepted task for that.

---

## Español

Datos que acompañan al dossier. Los precios son una foto del **30 de octubre de 2026**, no un feed en vivo. Las fórmulas están en [`../es/economia/01-contabilidad-de-tokens.md`](../es/economia/01-contabilidad-de-tokens.md).

- `prices-2026-10-30.csv`: precios por modelo (entrada, salida, lectura de caché, escritura corta y larga), en USD por millón de tokens.
- `delegation-scenarios.csv`: ahorro neto de un subagente para el escenario fijo (\(D=100k\), \(S=30k\), \(W=100k\), \(O=10k\), \(R=2k\)), por modelo principal/subagente y turnos restantes.

Supuestos: caché principal caliente, lectura al multiplicador publicado (0,05× en Opus/Sonnet 5.5), sin compactación, sin reintentos y sin revisión humana. Si el caché está frío, el ahorro cambia; si el subagente falla, el coste por tarea aceptada cambia más.

Para recalcular, usa la fórmula del documento 01 o el fragmento Python de arriba. Las etiquetas del CSV son las que aparecen en la tabla; ajústalas si tu proveedor usa otro nombre.
