# tb-lfx-classifier
TB Levofloxacin Resistance Classifier (scikit-learn, Colab)

This link will open the notebook directly in Colab.
---------------------------------------------------------

https://colab.research.google.com/drive/1AGKYmtY-JwSSvryzjdh-ugKIZX5dRyqZ?usp=sharing

Binary classifier to predict **levofloxacin resistance** in *M. tuberculosis* isolates from public sources. Implements **Logistic Regression** and **Random Forest** in **scikit-learn**, with evaluation on an imbalanced test set.

---
**Summary & interpretation**

*Background*

Fluoroquinolone (FQ) resistance in Mycobacterium tuberculosis (Mtb) is predominantly driven by mutations in the quinolone-resistance–determining regions (QRDRs) of gyrA and gyrB. Large collaborative resources (e.g., CRyPTIC) have shown strong genotype↔phenotype concordance for canonical sites (gyrA codons 90/91/94 and gyrB 533–538), while highlighting that non-canonical QRDR changes and cumulative mutational burden can also shift MICs. See: CRyPTIC Consortium, 2024 (PMC11237524).

-----------------------------------------------------

**Data & features**

**We assembled a training table by merging:**

- A levofloxacin MIC + binary label table (sample_id, lev_mic_mg_L, log2_mic, resistant_label), with per-sample variant summaries derived from **VARIANTS.csv.gz.**

**From variants we computed four burden features:**

- gyrA_burden – count of any nonsynonymous changes in gyrA QRDR (AA 67–106).

- gyrB_burden – count in gyrB QRDR (AA 460–550).

- total_qrdr_burden – gyrA_burden + gyrB_burden.

- has_gyrB_520 – 1 if any change exactly at gyrB 520, else 0.

-----------------------------------------------------

***Merging was by ENA/ERS sample_id; rows without variant summaries were imputed to zeros for the burden flags to avoid leakage or sample loss.***

-----------------------------------------------------

**Models**


We trained and evaluated logistic regression and random forest classifiers (with isotonic calibration for probability quality), using a stratified 80/20 split and reporting AUROC/AUPRC, accuracy, precision, recall, and F1. We also swept the decision threshold to illustrate rule-in vs rule-out use cases.

-----------------------------------------------------


Findings (this notebook)

**Test AUROC ≈ 0.92; AUPRC ≈ 0.76 at prevalence ≈0.16.**

**F1 ≈ 0.84 at a mid-range threshold (≈0.5), with accuracy ≈0.95, precision ≈0.85, recall ≈0.85.**

- Logistic regression and random forest produced near-identical performance, implying the signal is largely monotonic and additive (burden-like), not requiring complex non-linear interactions to capture.

- Random-forest permutation/importances concentrate on total_qrdr_burden and gyrA_burden, consistent with the literature that gyrA QRDR changes account for the majority of FQ resistance, while gyrB contributes in a smaller but sometimes specific way (e.g., 520).

-----------------------------------------------------

**Interpretation:**
-  These results replicate the central message of CRyPTIC: QRDR mutations alone provide high diagnostic information for levofloxacin resistance. Our “burden-only” model is intentionally compact, transparent, and performant—useful for quick triage, surveillance, and as a building block for broader genomic predictors.

-----------------------------------------------------

**How this compares to the reference study**

The CRyPTIC work emphasizes canonical gyrA 90/91/94 (and select gyrB sites) as strongest determinants of elevated MIC and clinical resistance. Our feature importance aligns: QRDR mutation presence and number dominate, with gyrB 520 flagged as a specific, potentially high-impact site. While CRyPTIC used much richer locus-level features and larger cohorts, our minimal burden representation still attains strong AUROC/AUPRC, underscoring how concentrated the FQ signal is in QRDR.

-----------------------------------------------------

**Limitations**

- Trained and validated within one aggregated dataset; external validation against independent cohorts and local lab MIC methods is essential before clinical deployment.

- Burden features ignore exact amino-acid identities and combinations; a richer model that keeps codon-level dummies can refine probabilities for borderline cases.

- We modeled levofloxacin only; results may differ for moxifloxacin breakpoints.

- As always, genotype ≠ phenotype in a subset of isolates (heteroresistance, measurement noise, novel mechanisms).

-----------------------------------------------------

**Practical use cases**

- Clinical rule-in (high precision): set a higher threshold to prioritize specificity—useful when starting/continuing FQs carries risk.

- Clinical rule-out (high sensitivity): set a lower threshold to avoid missing resistant cases; combine with confirmatory AST.

- Laboratory triage: flag QRDR-positive isolates for immediate review while MIC is pending.

- Surveillance/epi: monitor geographic or lineage shifts in FQ resistance burden.

- Trial stratification: enrich cohorts for likely FQ-resistant or -susceptible arms quickly.

- Research: fast hypothesis testing for new QRDR patterns; extend with codon-specific effects.

-----------------------------------------------------

**What clinicians/labs need to use this**

***Minimum input to the predictor (per isolate):***

- Gene (GENE) – at least gyrA and gyrB.

- Amino-acid position (AMINO_ACID_NUMBER / equivalent).

- (Optional) Amino-acid change (e.g., A90V). The current model ignores identity but future versions can use it.

**Where that comes from:**

- Whole-genome sequencing (WGS) pipelines that output VCF/variant tables (commonly used in public health labs). Tools like TB-Profiler or Mykrobe annotate gyrA/B QRDRs and can yield exactly the columns the helper cell expects.

- Targeted amplicon sequencing of gyrA/B QRDRs (low-cost panel) is sufficient for this predictor.

- Commercial line-probe assays (LPAs) such as GenoType MTBDRsl v2 detect canonical gyrA codons 90/91/94 and parts of gyrB (533–538). These results can be mapped to the features (gyrA_burden, gyrB_burden, has_gyrB_520) even without WGS.

- Xpert MTB/XDR cartridges can detect a subset of FQ resistance mutations; those outputs can also be mapped when codons are returned.

- Sanger sequencing of gyrA/B QRDRs is an inexpensive fallback to obtain AA positions/changes.

-----------------------------------------------------

**Turn-key workflow:**

- Obtain QRDR variant calls by any of the above (WGS VCF → annotated table, LPA callouts, amplicon results).

- Load into the notebook’s predict helper (CSV with UNIQUEID, GENE, AMINO_ACID_NUMBER).

- Get a calibrated probability of levofloxacin resistance and a label at your chosen threshold.

- Use in conjunction with phenotypic AST and clinical judgement—especially around the MIC breakpoint.
