# Graphene Raman Analysis

Batch analysis of graphene Raman spectra exported from LabSpec (`.txt`).
The notebook performs Savitzky–Golay smoothing, ALS (asymmetric least squares)
baseline correction, and region-by-region peak fitting of the **D, G, D′, 2D,
and D+G** bands, then aggregates results across treatments and samples into
summary CSVs and comparison plots.

## Features

- **Smoothing** — Savitzky–Golay filter (window/order configurable)
- **Baseline** — ALS baseline subtraction (λ, p, iterations configurable)
- **Peak fitting** (lmfit, windowed per region):
  - D band → pseudo-Voigt (~1350 cm⁻¹)
  - G + D′ → two-Lorentzian composite (~1580 / 1620 cm⁻¹)
  - 2D band → single Lorentzian (~2680 cm⁻¹)
  - D+G band → single Lorentzian (~2950 cm⁻¹)
- **Per-spectrum reports** — position, FWHM, amplitude, area, R², reduced χ²,
  plus amplitude-based I(D)/I(G) and I(2D)/I(G)
- **Batch mode** — recursive folder scan (`treatment/sample/*.txt`),
  per-replicate fit plots, per-sample mean ± std
- **Cross-treatment comparison** — grouped bar charts with error bars,
  exported alongside two CSV summaries

## Requirements

- Python ≥ 3.9, Jupyter Notebook / JupyterLab
- `numpy`, `pandas`, `matplotlib`, `scipy`, `lmfit`

`lmfit` is auto-installed by Cell 1 if missing. Install the rest with:

```bash
pip install numpy pandas matplotlib scipy lmfit
```

## Data layout

Organise spectra as a folder hierarchy (folder names are used as labels):

```
ROOT/
├── treatment_A/
│   ├── sample_1/
│   │   ├── spot1.txt
│   │   └── spot2.txt        ← replicates
│   └── sample_2/
│       └── spot1.txt
└── treatment_B/
    └── sample_1/
        └── spot1.txt
```

Each `.txt` file is a LabSpec export: optional `#`-prefixed header lines,
followed by two whitespace-separated columns (wavenumber, intensity).
Files placed directly in `ROOT/` or in a single sub-folder are assigned to
a `default` treatment/sample.

## Usage

Run the cells in order: **Cell 1 → Cell 2 → Quick Test (optional) → Cell 3**

1. **Cell 1 — Setup.** Defines all parameters and functions. Run once;
   re-run after any parameter change.
2. **Cell 2 — Choose folder.** Paste the path to `ROOT` when prompted.
   The folder tree is scanned and printed; an `output/` directory is
   created inside `ROOT`.
3. **Quick Test.** Fits a single spectrum and shows the three-panel plot
   (raw + baseline / corrected + fits / residuals) plus a fit-quality
   table. Use this to tune parameters before committing to a full run.
4. **Cell 3 — Full analysis.** Processes every spectrum, saves one fit
   plot per replicate, prints per-sample ID/IG and I2D/IG (mean ± std),
   then builds the cross-treatment comparison figure and CSV exports.

## Outputs (written to `ROOT/output/`)

| File | Content |
|---|---|
| `<treatment>_<sample>_<file>.png` | Per-spectrum fit plot |
| `all_spectra_summary.csv` | One row per spectrum: all peak parameters and ratios |
| `sample_mean_std.csv` | Mean ± std per (treatment, sample) |
| `cross_treatment_comparison.png` | 2×3 grid: ID/IG, I2D/IG, G position/FWHM, 2D position, D FWHM |

## Key parameters (top of Cell 1)

| Parameter | Meaning | Default |
|---|---|---|
| `SG_WINDOW`, `SG_ORDER` | Savitzky–Golay window (odd) and polynomial order | 7, 3 |
| `ALS_LAMBDA` | Baseline smoothness — larger → flatter baseline | 1e5 |
| `ALS_P` | Baseline asymmetry — smaller → hugs valleys | 0.02 |
| `WIN_D / WIN_G / WIN_2D / WIN_DG` | Fit windows per band (cm⁻¹) | see code |
| `GUESS` | Initial peak-centre guesses (cm⁻¹) | D 1350, G 1580, D′ 1620, 2D 2680, D+G 2950 |
| `FLOAT` | Allowed peak-centre shift from guess (± cm⁻¹) | 30 |
| `PEAK_COLORS` | Plot colour per band | C1/C2/C5/C4/C6 |

A full troubleshooting table (baseline following peaks, failed fits, noisy
or over-smoothed spectra, etc.) is included in the notebook under
**Parameter Tuning Guide**.

## Notes & conventions

- I(D)/I(G) and I(2D)/I(G) are computed from fitted **amplitudes**
  (peak heights), not integrated areas. Area values are also exported
  in the CSVs if area-based ratios are preferred.
- Peak FWHM is reported as `2 × wid` of the fitted lineshape.
- Fits that fail or fall below the minimum point count return `NaN`
  and are flagged `not fitted` / skipped in batch mode.

## Changelog

### v1.1 (2026-06)
- Initial documented release: SG smoothing + ALS baseline + windowed
  multi-band fitting, batch hierarchy scan, cross-treatment comparison,
  CSV export.

<!-- Add future entries above this line, newest first. -->
