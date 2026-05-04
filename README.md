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

---

## Tools

### `okluch_gradient_pub.html` — Ver. 1.0-pub

Interactive gradient comparison tool with full pipeline visualization.

**Features:**
- Gradient interpolation: Oklab / Oklch / Oklch+ side-by-side
- Hue modulation: continuous singularity resolution via NR weight
- Gamut Mapping Layer 3: C_max(L,H) full implementation with A/B toggle
- ab-plane interpolation path visualization
- Chroma C profile canvas

---

## Methods

### Oklch+ (Color Difference Metric)

Naka-Rushton compression on the chroma axis of OKLCH:

```
L′ = L^α
C′ = C^n / (C^n + σ^n)
h′ = h
ΔE = √(ΔL′² + Δa′² + Δb′²)   [Oklab Euclidean in transformed space]

Optimal parameters: α=0.73, n=0.87, σ=0.34
```

Evaluated on COMBVD dataset (3,813 pairs, 6 sub-datasets).
Sub-datasets: BFD-P / Witt / Leeds / RIT-DuPont / Luo-Rigg / He et al. 2022
Source: https://opticapublishing.figshare.com/articles/dataset/CIE_Combined_Corrected_data_set_xlsx/19678395
(Dataset not included in this repository)
Surpasses CIEDE2000 with 3 parameters vs ~17 empirical constants.

### Hue Modulation (Interpolation Singularity Resolution)

Achromatic colors (black, white, grey) have undefined hue in OKLCH polar coordinates.
Existing approaches use binary NaN-inheritance (W3C CSS) or discard hue control entirely (Oklab fallback).

This implementation uses a continuous NR-weighted blend between OKLCH polar and Oklab linear interpolation:

```javascript
w_h(C) = C^0.87 / (C^0.87 + 0.34^0.87)
// C → 0 : Oklab linear (hue contribution = 0)
// C >> σ : OKLCH polar  (full hue control)
```

No additional parameters — reuses Oklch+ chroma parameters.
Differentiable and continuous at all points.

---

## Articles

- [OKLCH グラデーションで色かぶりが起きる理由と、連続的な解決策](#) *(Zenn — coming soon)*
- [OKLCH 極座標補間の特異点を「滑らかに」解消する](#) *(Zenn — coming soon)*

---

## Related Work

- Oklab: Björn Ottosson (2020) — https://bottosson.github.io/posts/oklab/
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
