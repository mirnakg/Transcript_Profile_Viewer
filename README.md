# Transcript Profile Viewer

A command-line tool for visualizing read coverage around genes/transcripts from RNA-seq BAM files, using custom GTF annotations. Built for working with recombinant or non-standard genomes where off-the-shelf tools fall short.

## What it does

Given a BAM file and a GTF annotation, this tool generates per-gene coverage profiles anchored at a feature of interest (TSS, TES, CDS start/end). It handles:

- **Custom GTF annotations** — works with non-standard or recombinant genomes where gene names and loci may not match public references
- **GTF position shifting** — includes a utility to shift genomic coordinates in a GTF file by a fixed offset (useful when your insert is at a different position than the reference)
- **Flexible anchoring** — anchor profiles at the TSS, TES, CDS start, or CDS end
- **Strand-aware coverage** — supports unstranded, fr-firststrand, and fr-secondstrand libraries
- **RPM normalization** and optional log transforms (log2, log10, ln)
- **Binning** for smoothing noisy coverage signals
- **Per-gene plots and TSVs** for downstream analysis or publication figures

## Requirements

```
pip install pysam biopython numpy pandas matplotlib
```

## Usage

### Coverage profiles

```bash
python tss_cov_custom.py \
  --bam your_sample.bam \
  --gtf your_annotation.gtf \
  --genes GENE1 GENE2 \
  --feature gene \
  --anchor tss \
  --upstream 6000 --downstream 6000 \
  --bin 10 --norm rpm \
  --strand unstranded \
  --log log10 --pseudocount 1 \
  --plot \
  --outdir profiles
```

You can also pass a file of gene IDs (one per line) with `--genes-file`.

### GTF position shifting

If your recombinant insert sits at a different genomic position than the reference GTF, you can shift coordinates:

```python
from tss_cov_custom import shift_gtf_positions

shift_gtf_positions(
    "original.gtf",
    "shifted.gtf",
    offset=5766,
    gene_keyword="ACIAD_RS07960"  # start shifting at this gene
)
```

### Key arguments

| Argument | Description |
|----------|-------------|
| `--bam` | Input BAM file (must be indexed) |
| `--gtf` | Custom GTF annotation |
| `--genes` | Gene IDs/names to profile |
| `--feature` | Anchor feature: `gene`, `transcript`, or `CDS` |
| `--anchor` | Anchor point: `tss`, `tes`, `cds_start`, `cds_end` |
| `--upstream` / `--downstream` | Window size in bp (default 1000 each) |
| `--bin` | Bin size for smoothing (default 10) |
| `--norm` | Normalization: `none` or `rpm` |
| `--strand` | Library type: `unstranded`, `fr-firststrand`, `fr-secondstrand` |
| `--log` | Log transform: `none`, `log2`, `log10`, `ln` |
| `--attrs` | GTF attribute keys to match gene IDs (order matters) |

## Output

- **TSV files** with columns: `rel_bp`, `coverage_rpm` (and optionally log-transformed values)
- **PNG plots** (with `--plot`) showing coverage around the anchor point
- **manifest.json** listing all generated profiles and their parameters

## Notes

- BAM files must be sorted and indexed (`.bai` file required)
- The tool handles edge cases like features near contig boundaries by clamping the window
- If gene IDs aren't found, check your `--attrs` flag — custom GTFs may use non-standard attribute keys (e.g., `RecombinantID`, `locus_tag`)
- For CDS features, the tool collapses all CDS entries per gene into one span
