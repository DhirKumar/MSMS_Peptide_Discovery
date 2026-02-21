# Benchmarking

This directory contains utilities to evaluate the pipeline's performance
against datasets with known correct answers.

## compare_results.py

Compares pipeline output (`identified_psms.csv`) against an answer key CSV
and reports sensitivity, precision, F1, and FDR accuracy.

### Usage

```bash
python benchmark/compare_results.py \
    --our-psms   pipeline_output/identified_psms.csv \
    --answer-key benchmark/answer_key.csv \
    --key-col    Peptide \
    --our-col    Peptide \
    --fdr        0.01
```

### Output

```
════════════════════════════════════════════════════════════
  MS/MS PIPELINE BENCHMARK
════════════════════════════════════════════════════════════
  Metric                          Value
────────────────────────────────────────────────────────────
  Our unique peptides             1,842
  Answer key peptides             2,105
────────────────────────────────────────────────────────────
  True Positives (TP)             1,731
  False Positives (FP)              111
  False Negatives (FN)              374
────────────────────────────────────────────────────────────
  Sensitivity (Recall)            82.2%
  Precision                       94.0%
  F1 Score                        0.877
────────────────────────────────────────────────────────────
  Nominal FDR                      1.0%
  Actual FDR (FP / total)          6.0%
  FDR status              ⚠ inflated
────────────────────────────────────────────────────────────
```

Four CSV files are saved to `pipeline_output/`:

| File | Contents |
|---|---|
| `benchmark_summary.csv` | Single-row metrics summary |
| `benchmark_true_positives.csv` | Our PSMs that match the answer key |
| `benchmark_false_positives.csv` | Our PSMs not in the answer key |
| `benchmark_false_negatives.csv` | Answer key PSMs we missed |

## Recommended Datasets

### iPRG2012 (gold standard)

The ABRF iPRG 2012 study provides a benchmark dataset with a published
list of correct PSMs, making it ideal for sensitivity/precision evaluation.

- **PRIDE accession:** PXD000612
- **Publication:** Choi et al., J Proteome Res 2012
- **Spectra:** ~10,000 MS2 scans
- **Answer key:** available from the ABRF iPRG committee

### PXD000561

Erwinia carotovora TMT — the same organism as the default demo dataset,
but a different LC-MS/MS run. Useful for reproducibility testing.

### PXD001197

Human HeLa cell digest with extensive manual curation. Good for testing
against a complex human proteome.

## Interpreting Results

| Metric | Target | Notes |
|---|---|---|
| Sensitivity | > 80% | % of true PSMs found |
| Precision | > 90% | % of our PSMs that are correct |
| F1 Score | > 0.85 | Balance of sensitivity and precision |
| Actual FDR | ≤ 2× nominal | FDR should not be severely inflated |
| Score separation | > 5 | TP scores should clearly exceed FP scores |

If actual FDR is much higher than nominal:
- Tighten fragment tolerance
- Increase min fragment matches
- Use stricter FDR threshold (0.1% instead of 1%)

If sensitivity is low:
- Widen precursor tolerance (pilot pass handles this automatically in v5)
- Increase missed cleavages
- Check FASTA covers the correct organism
