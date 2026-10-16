# Diapause-associated divergence in circadian architecture between *Culex pipiens* and *Culex quinquefasciatus*

Analysis code for:

> Geddes G, Dhungana P, Jhara MS, McKay K, Sim C. Diapause-Associated Divergence in Circadian Architecture Between *Culex pipiens* and *Culex quinquefasciatus*. *Journal of Biological Rhythms* (in review).

Department of Biology, Baylor University, Waco, TX, USA

This repository contains the R scripts used to produce the statistics and figures in the paper. The individual-level data and complete statistical output for every figure are in **Supplemental Table S1**, published with the article. Each script lists the Table S1 sheet that holds its results, so the code and the numbers can be checked against each other.

---

## Repository contents

```
culex-circadian-diapause/
├── README.md
├── LICENSE
├── CITATION.cff
├── follicle/
│   └── Fig1_follicle_length.Rmd
├── activity/
│   ├── Fig2a_population_profile_longday.Rmd
│   ├── Fig2a_population_profile_shortday.Rmd
│   ├── Fig2bc_activity_entrainment.Rmd
│   ├── Fig2d_activity_constant_darkness.Rmd
│   ├── Periodogram_period_power_stats.Rmd
│   ├── Fig3_FigS2_activity_peaks.Rmd
│   └── Fig3_peak_visual_inspection.Rmd
├── qpcr/
│   └── Fig4_5_qPCR_circacompare.Rmd           <- template, run once per gene
└── ihc/
    └── Fig6_FigS5_PDF_immunoreactivity.Rmd
```

## Scripts by figure

| Figure | Script | What it does | Table S1 sheet |
|---|---|---|---|
| Fig. 1 | `follicle/Fig1_follicle_length.Rmd` | Mean primary follicle length per female. Two-way ANOVA (species × photoperiod) with Tukey HSD, compact letter display, and the proportion of females above 75 µm. | `Fig1_Follicle_Length` |
| Fig. 2a | `activity/Fig2a_population_profile_longday.Rmd`, `activity/Fig2a_population_profile_shortday.Rmd` | Proportion of individuals active per 30-min bin across the 6-day recording (3 days LD, then 3 days DD). | — (plot only) |
| Fig. 2b–c | `activity/Fig2bc_activity_entrainment.Rmd` | Average 24 h activity profiles over the first 3 full LD days (11-min moving average), and daily mean activity per individual with adaptive two-way analysis. | `Fig2_3_Behavioral` (LD Activity) |
| Fig. 2d | `activity/Fig2d_activity_constant_darkness.Rmd` | The same analysis for the last 3 days in constant darkness. | `Fig2_3_Behavioral` (DD Activity) |
| Results text (period, power) | `activity/Periodogram_period_power_stats.Rmd` | Free-running period and rhythmic power from Lomb-Scargle periodograms (18–30 h), compared across species × photoperiod. | `Fig2_3_Behavioral` (Periodogram) |
| Fig. 3, Fig. S2 | `activity/Fig3_FigS2_activity_peaks.Rmd` | M- and E-peak detection in DD (adapted from Vaze et al. 2023), peak detection rates (Fisher's exact tests), E-peak phase relative to last lights-off, E-to-M interval and its deviation from the entrained night length. | `Fig2_3_Behavioral` (Peak detection; Interval Length, Interval Length Deviation, E-peak Phase) |
| Fig. 3 (validation) | `activity/Fig3_peak_visual_inspection.Rmd` | One diagnostic page per individual for checking peak calls by eye. **Run after `Fig3_FigS2_activity_peaks.Rmd` in the same R session.** | — |
| Figs. 4–5, Figs. S3–S4 | `qpcr/Fig4_5_qPCR_circacompare.Rmd` | Rhythmicity of relative mRNA expression (`circa_single`), comparison of mesor, amplitude and phase (`circacompare`), with a two-way ANOVA (group × ZT) and Tukey contrasts when a group is not rhythmic. Species compared within each photoperiod, and photoperiods within each species. | `Fig4_5_RTqPCR_CircaCompare` |
| Fig. 6, Fig. S5 | `ihc/Fig6_FigS5_PDF_immunoreactivity.Rmd` | PDF⁺ arborization area, CTCF and mean intensity in short-day brains at ZT17. Hemispheres are averaged to one value per brain (n = 4 per species); area and CTCF are log-transformed; species compared by Welch's t-test. Run at threshold 100 (Fig. 6) and at 75 and 150 (Fig. S5) to check robustness to threshold choice. | `Fig6_IHC` |

## Statistical approach (activity scripts)

For each continuous outcome, a linear model (species × photoperiod) is fitted and its residuals tested with the Shapiro–Wilk test (α = 0.05):

1. Residuals normal: Type II two-way ANOVA (`car::Anova`) with pairwise contrasts from `emmeans`.
2. Residuals non-normal, outcome strictly positive, log-scale residuals normal: the same analysis on the log scale.
3. Otherwise: Aligned Rank Transform ANOVA with ART-C contrasts (`ARTool`).

P-values are Holm-adjusted across all six pairwise comparisons among the four groups. The two species comparisons within each photoperiod are then taken from that corrected set for reporting; they are not corrected again.

## Input data

Scripts read their input files from the folder they are in. Knitting an `.Rmd` in RStudio sets the working directory to that folder automatically.

No data files are included in this repository. The scripts expect the input files described below, placed in the same folder as the script.

**Fig. 1:** `Follicle_IndividualMeans.csv`, one row per female (n = 25 per species × photoperiod), with columns `Species` (`Cx. pipiens` or `Cx. quinquefasciatus`), `Photoperiod` (`9L:15D` or `15L:9D`), `Individual` and `Mean_Follicle_Length` (µm; mean of 10 primary follicles). These per-female means, and the 10 individual follicle measurements behind each one, are in Table S1, sheet `Fig1_Follicle_Length`. To rerun Fig. 1, copy the per-female means into a CSV with these four column names.

**Activity (Figs. 2–3):** raw TriKinetics LAM recordings are available from the corresponding author on request. To rerun these scripts, place the files in `activity/` with the following names:

| File | Format | Used by |
|---|---|---|
| `pipiens_LD.txt`, `quinque_LD.txt`, `pipiens_SD.txt`, `quinque_SD.txt` | TriKinetics MonitorNN file, 1-min bins. Column 8 is the data type (`Ct` = counts), columns 11–42 are channels 1–32 (one female per channel). | Fig. 2b–d, Fig. 3 |
| `pipiens_LD_30min.txt.txt`, `quinque_LD_30min.txt.txt`, `pipiens_SD_30min.txt.txt`, `quinque_SD_30min.txt.txt` | The same recordings in 30-min bins. | Fig. 2a |
| `pipiens_LD_Periodogram-4.xlsx`, `quinque_LD_Periodogram-4.xlsx`, `pipiens_SD_Periodogram-4.xlsx`, `quinque_SD_Periodogram-4.xlsx` | Lomb-Scargle periodogram export from Rtivity (sheet `All channels`; columns `Labels`, `period`, `power`, `p_value`), restricted to individuals Rtivity classified as rhythmic. | Period and power |

Photoperiods: long day (LD) = 15L:9D, lights on 06:03; short day (SD) = 9L:15D, lights on 08:00. Each recording is 3 days of LD entrainment followed by 3 days of constant darkness (DD).

**Excluded channels:** individuals that did not survive the 6-day recording were removed. Excluded channels are listed near the top of each script (`EXCLUSIONS`, `*_exclude_ch`, `EXCLUDE_PIP` / `EXCLUDE_QUIN`).

**qPCR (Figs. 4–5):** one Excel file per gene, one row per sample, with columns `gene`, `species`, `photoperiod` (`SD`/`LD`), `zt`, `replicate` and `expr` (relative expression, 2^−ΔΔCt). Per-ZT means, SD and SE, together with all CircaCompare and ANOVA output, are in Table S1, sheet `Fig4_5_RTqPCR_CircaCompare`. Sample-level data are available from the corresponding author on request.

**IHC (Fig. 6):** `IHC_long.csv`, one row per hemisphere per threshold, with columns `species` (`pipiens`/`quinquefasciatus`), `photoperiod` (`SD`/`LD`), `brain_id`, `threshold` (`75`, `100`, `150`), `area` (µm²), `CTCF` and `mean_intensity`. The script analyzes the short-day rows (16 per threshold). The per-hemisphere FIJI measurements, background values and CTCF are in Table S1, sheet `Fig6_IHC`. Images are available from the corresponding author on request.

## Running the qPCR analysis for each gene

The same script was run for every gene; only the settings in Section 2 change. For each gene, set:

```r
input_file  <- "CYC_ddct.xlsx"   # per-gene input file
output_dir  <- "CYC_results_expanded"
target_gene <- "CYC"             # must match the `gene` column
gene_label  <- "cycle"           # italic gene name on the y-axis
```

| Gene | `gene_label` |
|---|---|
| *cycle* | `cycle` |
| *Clock* | `Clock` |
| *period* | `period` |
| *timeless* | `timeless` |
| *Par domain protein 1* | `Pdp1` |
| *vrille* | `vrille` |
| *cryptochrome 2* | `cry2` |
| *pigment-dispersing factor* | `pdf` |

Each run writes a results folder with the species and photoperiod comparisons, the plots, and `99_compiled_results_tables.xlsx`.

## Software

Analyses were run in R 4.3.1. Packages used:

- Data handling and plotting: `tidyverse` (`dplyr`, `tidyr`, `ggplot2`, `readr`, `stringr`, `purrr`), `lubridate`, `readxl`, `openxlsx`, `writexl`, `patchwork`, `ggsignif`, `ragg`, `zoo`
- Statistics: `car`, `emmeans`, `ARTool`, `multcomp`, `multcompView`
- Rhythm analysis: `circacompare` (installed from GitHub: `remotes::install_github("rwparsons/CircaCompare")`)

Periodograms were computed in Rtivity (Silva et al. 2022) and imported as Excel exports. PDF immunohistochemistry (Fig. 6) was quantified in FIJI/ImageJ.

Record the package versions you used with `sessionInfo()`.

## Key references for the methods

- Vaze KM et al. (2023). *J Comp Physiol A*: peak detection method adapted here.
- Parsons R, Parsons R, Garner N, Oster H, Rawashdeh O (2020). CircaCompare: a method to estimate and statistically support differences in mesor, amplitude and phase, between circadian rhythms. *Bioinformatics* 36(4): 1208–1212.
- Wobbrock JO, Findlater L, Gergle D, Higgins JJ (2011). The aligned rank transform for nonparametric factorial analyses using only ANOVA procedures. *Proc. ACM CHI 2011*, 143–146.
- Elkin LA, Kay M, Higgins JJ, Wobbrock JO (2021). An aligned rank transform procedure for multifactor contrast tests. *Proc. ACM UIST 2021*, 754–768.

## Citation

If you use this code, please cite the paper above and this repository (see `CITATION.cff`, or use GitHub's "Cite this repository" button).

## License

Code is released under the MIT License (see `LICENSE`).

## Contact

Gavin Geddes, Department of Biology, Baylor University. Corresponding author: Cheolho Sim, Department of Biology, Baylor University.
