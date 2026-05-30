# oklch-plus

Research and implementation tools for perceptual color improvement in OKLCH space.

**Oklch+** is a color difference metric that surpasses CIEDE2000 using only 3 parameters,
built on the Naka-Rushton response function applied to the chroma axis of OKLCH.

---

## Key Results

| Model | COMBVD STRESS (↓ better) | vs Oklab |
|-------|--------------------------|----------|
| Oklab (baseline) | 47.35 | — |
| **Oklch+ (α=0.73, n=0.87, σ=0.34)** | **29.10** | **−18.3** |
| CIEDE2000 (reference) | 29.18 | −18.2 |

Cross-validation (test set: BFD-P D65, 2,028 pairs): STRESS = 23.96

Note: RIT-DuPont sub-dataset has all DV values normalized to 1.02,
making quantitative interpretation of its individual STRESS score unreliable.

---

## Tools

### `TOKIIRO_v1.html` — 時彩（ときいろ）Ver. 1.0

Narrative color selection tool based on OKLCH perceptual uniformity.
Choose a **scene** (情景 → Hue) and **time of day** (時間帯 → L/C) to generate an AI-ready color system.

**Features:**
- Scene × time-of-day 2-axis selection (6 scenes × 6 times = 36 combinations)
- Auto-generates 11-color CSS Variable palette in OKLCH format
- Exports `.md` for AI tools (Claude, ChatGPT, Cursor, etc.) and CSS for direct use
- RESEARCH tab: Oklch+ correction visualization and L-C gamut cross-section

---

### `okluch_gradient_pub.html` — Ver. 1.0-pub

Interactive gradient comparison tool with full pipeline visualization.

**Features:**
- Gradient interpolation: Oklab / Oklch / Oklch+ side-by-side
- Hue modulation: continuous NR-weighted blend (differentiable at all points)
- Gamut Mapping Layer 3: gamut limit (L only) / gamut limit (L,H) with A/B toggle
- ab-plane interpolation path visualization
- Chroma C profile canvas

---

## Methods

### Oklch+ (Color Difference Metric)

Naka-Rushton compression on the chroma axis of OKLCH:

```
L′ = L^α
C′ = C^n / (C^n + σ^n)
h′ = h                          (unchanged; hue weight = C′ implicitly)
a′ = C′ cos(h),  b′ = C′ sin(h)
ΔE = √(ΔL′² + Δa′² + Δb′²)   [Oklab Euclidean in transformed space]

Optimal parameters: α=0.73, n=0.87, σ=0.34
```

Evaluated on COMBVD dataset (3,813 pairs, 6 sub-datasets).
Sub-datasets: BFD-P / Witt / Leeds / RIT-DuPont / Luo-Rigg / He et al. 2022
Source: https://opticapublishing.figshare.com/articles/dataset/CIE_Combined_Corrected_data_set_xlsx/19678395
(Dataset not included in this repository)
Surpasses CIEDE2000 with 3 parameters vs ~17 empirical constants.

### Hue Modulation (Interpolation Singularity Resolution)

OKLCH interpolation produces unintended color casts in two scenarios:

1. **Chromatic-to-chromatic paths** passing through low-chroma regions
   (e.g., blue→yellow: a greenish band appears in the middle)
2. **Achromatic endpoints** (black, white, grey) where hue is undefined
   (existing approaches: binary NaN-inheritance (W3C CSS) or discard hue control (Oklab fallback))

This implementation uses a continuous NR-weighted blend between OKLCH polar and Oklab linear
interpolation, resolving both cases with a single differentiable function:

```javascript
w_h(C) = C^0.87 / (C^0.87 + 0.34^0.87)
// C → 0 : Oklab linear (hue contribution = 0)
// C >> σ : OKLCH polar  (full hue control)
```

No additional parameters — reuses Oklch+ chroma parameters.
Differentiable and continuous at all points.
The interpolation path lies between Oklab and Oklch, combining stability at low chroma
with hue control at high chroma.

---

## Articles

**時彩（ときいろ / TOKIIRO）**
- [「清らかな朝のような色」がそのまま色座標になる — 情景×時間帯で色を選ぶツール「時彩」を作った](https://zenn.dev/nekotrack/articles/9613d1a1df1f24) *(Zenn)*
- [色を「選ぶ」より「出会う」— 時彩(TOKIIRO) で AI・デザインツールに渡すパレットを作る](https://note.com/nekotrack/n/n923cf8f27588) *(note)*

**Oklch+ / Hue Modulation**

- [OKLCH グラデーションで色かぶりが起きる理由と、なめらかな解消法](https://zenn.dev/nekotrack/articles/35849cff2ba1c4) *(Zenn)*
- [OKLCH グラデーションで意図しない色が現れる理由](https://note.com/nekotrack/n/n822dd5bbcefd) *(note)*
- [OKLCH 極座標補間の特異点を「滑らかに」解消する](https://zenn.dev/nekotrack/articles/7c3487c5a9375a) *(Zenn)*
- [Oklch+ — 3パラメータで CIEDE2000 を超える色差改善の記録](https://zenn.dev/nekotrack/articles/290dea9ac90fa9) *(Zenn)*

---

## Related Work

- Oklab / Oklch: Björn Ottosson (2020) — https://bottosson.github.io/posts/oklab/
  (Oklch is the polar coordinate form of Oklab — same author)
- W3C CSS Color 4 hue interpolation: Issue [#6107](https://github.com/w3c/csswg-drafts/issues/6107), [#9436](https://github.com/w3c/csswg-drafts/issues/9436)
- CIEDE2000: Luo et al. (2001)
- CIECAM02: Li et al. (2002)
- Naka & Rushton (1966): S-potentials from colour units in the retina of fish

---

## License

- **Tools** (`tools/`): [MIT License](LICENSE-MIT)
- **Articles and documents** (`articles/`, `docs/`): [CC BY 4.0](LICENSE-CC-BY-4.0)

---

## Acknowledgements

Algorithm design, parameter optimization, and all research decisions by nekotrack.
Implementation code assisted by [Claude](https://claude.ai) (Anthropic).

---

© 2026 nekotrack
