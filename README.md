# RASopathy Functional Evidence Atlas

**One-directional functional evidence in RASopathy variant classification.**

How much functional evidence underpins ClinGen RASopathy VCEP variant classifications,
and what is missing when a classification cannot be reached.

Reproducible from public data: two frozen monthly ClinVar releases, the ClinVar
submission summary, the MaveDB API, and UniProt. No credentials, no private data, no
local setup.

| Notebook | Platform | Notes |
|---|---|---|
| [`rasopathy_atlas_kaggle.ipynb`](rasopathy_atlas_kaggle.ipynb) | Kaggle | Enable Internet in notebook options |
| [`rasopathy_atlas_colab.ipynb`](rasopathy_atlas_colab.ipynb) | Google Colab | Set `MOUNT_DRIVE = True` to cache downloads |

The two are the same analysis; only the configuration cell differs. Runtime is 40–60
minutes, dominated by ClinVar downloads (~900 MB).

## Findings

Across the 14 genes covered by the ClinGen RASopathy Variant Curation Expert Panel:

**Reclassification is close to static.** Between ClinVar 2022-01 and 2026-08, 20 of
1,483 missense VUS reached a confident classification; 26 moved the other way. The
largest single destination was conflicting interpretation (175). The missense cohort
grew from 2,378 to 8,457 over the same window.

**Functional evidence is one-directional.** In the panel's own ClinVar submissions,
functional criteria appear in roughly 55% of pathogenic classifications and 3% of
benign ones — two BS3 applications in total.

**Curation coverage tracks familiarity, not burden.** LZTR1 contributes 2,150 missense
entries with 12 curated; CBL contributes 1,316 with none. Together they are 41% of the
cohort.

**Forty variants are curated and still uncertain**, most with no applied functional
criterion. That is what prompted the MaveDB search: if the evidence is absent from the
record, the next question is whether it exists in the literature and simply has not
been applied.

**A coordinate convention hid deposited MAVE data.** An HGVS-string lookup against
MaveDB returned zero coverage of the blocked set. That was wrong: the HRAS score sets
number positions against the mature protein. After correcting the offset, two blocked
VUS are present with quantitative scores across three independent score sets. RAF1 and
BRAF carry larger offsets (+51, +476) because those depositions cover isolated domains,
which places the remaining blocked variants outside the construct rather than absent
from it.

MaveDB publishes the offset in its score-set metadata. The notebook reads it *and*
derives it independently by aligning the assay's own reference sequence against
UniProt, then reports any disagreement. On the current data the two routes agree
exactly for all three genes.

**The recovered data is not usable as-is.** The source assay (Bandaru et al., *eLife*
2017, PMID 28686159) is a bacterial two-hybrid system coupling Ras·GTP:Raf-RBD binding
to antibiotic resistance — no membrane, a single effector, and ITC validation with
explicitly greater variance near wild-type binding, which is where both recovered
variants sit. Neither variant is discussed individually in the paper, and the authors
deliberately avoided clinical interpretation. This is a fitness-for-purpose problem,
not a scarcity problem.

## Errors found in this pipeline that changed conclusions

Documented on purpose. Each was caught by instrumentation added after the fact, and a
reader should know which results were once reported differently. Both false positives
ran in the same direction — toward having found something.

| Error | Effect | Fix |
|---|---|---|
| Missense regex admitted `p.Arg498Ter` | Nonsense counted as missense, inflating the reclassification count the study design rested on | Both residues constrained to the 20 standard amino acids |
| Criteria matched on a ±90-character window | 46% of ACMG code mentions unresolvable | Each code scoped to its own clause; unresolved fell to 6% |
| Top-N truncation when reporting criteria frequency | BS3 read as absent when it was 2 | Report the crosstab, never the head of a sorted list |
| Blocked set defined by extraction success | Two variants dropped for being hard to parse | Membership from curation status; extraction annotates rather than selects |
| MaveDB matched across genes | A string from an HRAS scan attributed to LZTR1, in an output where LZTR1 showed zero score sets | Gene-scoped target sets |
| MaveDB lookup by HGVS string | Zero coverage reported where two variants exist | Coordinate offset resolved before matching |

## Data sources and versions

| Source | Version |
|---|---|
| ClinVar `variant_summary` | `archive/2022/variant_summary_2022-01.txt.gz` |
| ClinVar `variant_summary` | `archive/variant_summary_2026-08.txt.gz` |
| ClinVar `submission_summary` | current at run time |
| MaveDB | API v1, queried at run time |
| UniProt | P01112 (HRAS), P04049 (RAF1), P15056 (BRAF) |

Both sides of the temporal comparison are frozen monthly archive releases. The undated
weekly file would make the result irreproducible.

## Invariants

The notebook carries the figures above as tripwires and reports any divergence at the
end of the run. Divergence is not necessarily an error — ClinVar and MaveDB both change
— but it means the frozen numbers no longer describe that run and should not be quoted
without checking.

## Limitations

1. Detection, not absence. These searches establish what was found. Manual review of the blocked records is outstanding.
2. Vocabulary-based detection of functional evidence undercounts, and the undercount favours this project's thesis.
3. The blocked set has three denominators — curated (40), parseable (38), adjudicated (36). They are not interchangeable.
4. Whether a bare list of ACMG codes means "applied" is unresolved; it changes the pathogenic count by roughly half. Both figures are reported.
5. Percentages rest on 36–115 variants. Wilson intervals accompany every point estimate.
6. VCEP attribution is a substring match on the submitter name, and absence of a deposited classification is not absence of curation. Reclassifications made in clinical practice and never submitted are invisible here.
7. Separation between existing ClinVar classifications is a precondition for calibration, not a calibration. Those labels derive in part from the same literature as the assays.
8. Paralogue transfer is not an ACMG evidence pathway. PS1 and PM5 operate within a gene.

## Outputs

Written to `outputs/` at run time. The copies committed here are from the frozen run
described above.

```
gene_cohort_summary.csv        missense count and VUS share per gene
code_mentions.csv              every ACMG code mention with its clause and verdict
per_variant_applied.csv        applied codes per curated variant
headline_table.csv             functional evidence by classification
gene_curation_table.csv        curation coverage against variant burden
blocked_vus_worksheet.csv      the blocked set, with empty columns for manual review
mavedb_inventory.csv           score sets per gene
mavedb_recovered.csv           blocked VUS recovered after offset correction
recovered_percentiles.csv      where those variants sit in each distribution
calibration_feasibility.csv    controls available per score set
```

`mavedb_scores/` holds the score-set CSVs exactly as retrieved, so the offset resolution
can be checked without re-querying the API. The ClinVar releases are not committed —
they exceed GitHub's file size limit and are permanently archived at the NCBI FTP paths
listed above.

## Status

Work in progress. The blocked-variant records are being reviewed by hand; until that
is finished, the criteria counts here come from a text parser over ClinVar submission
free text and should be treated as provisional.

Corrections are welcome — several of the findings above exist because an earlier
version was wrong.
