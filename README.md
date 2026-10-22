# amg-kit

**A DBSCAN-based toolkit for identifying candidate auxiliary metabolic genes (AMGs) in viral genomes and metagenomic assemblies.**

`amg-kit` identifies conserved viral gene regions using density-based clustering and integrates protein annotations from KEGG, Pfam, and VOG to identify candidate AMGs.

## Features

- **Viral conserved-region detection:** Uses DBSCAN-based clustering of annotated viral genes to identify conserved genomic regions.
- **KEGG annotation:** Integrates KO identifiers and functional descriptions.
- **Pfam annotation:** Incorporates protein domain information.
- **VOG annotation:** Uses viral orthologous group annotations to provide viral genomic context.
- **AMG identification:** Labels candidate AMGs using the bundled AMG database.
- **Configurable clustering:** Adjusts the maximum distance between viral genes and the minimum number of genes required to define a conserved region.

## Workflow

```text
Viral contigs and gene annotations
                |
                v
       Extract viral genes
                |
                v
    DBSCAN-based region detection
                |
                v
       Assign cluster labels
                |
                v
  Integrate KEGG, Pfam and VOG
                |
                v
       Candidate AMG labeling
                |
                v
          AMGs.txt
```

## Installation

Tested operating system: Rocky Linux 9.

### Requirements

`amg-kit` is designed for Linux environments and requires:

| Dependency | Purpose |
|---|---|
| Perl | Runs the main `amg-kit` executable |
| Bash | Executes generated workflow scripts |
| `bedtools` | Intersects viral gene coordinates with detected regions |
| `tsv-utils` | Processes and joins tabular annotations |
| `range-tk` | Performs distance-based gene clustering |
| `amg-tk` | Labels candidate AMGs using the bundled database |

The repository includes `binaries/` and `db/` directories. Check that the bundled executables are compatible with your Linux architecture and that the required database files are present.

### 1. Clone the repository

```bash
git clone https://github.com/kits-hub/amg-kit.git
cd amg-kit
```

### 2. Check the executable and bundled resources

```bash
chmod +x amg-kit

ls -l amg-kit
ls -lh binaries/
ls -lh db/
```

### 3. Check Perl and external dependencies

```bash
perl --version
bedtools --version
```

Check the bundled executables:

```bash
./binaries/range-tk --help
./binaries/amg-tk --help
```

The exact supported options depend on the bundled binary versions.

If `bedtools` or `tsv-utils` is not available, install it using your preferred Linux package manager or a reproducible environment manager, and ensure the executable is on `PATH`.

### 4. Add amg-kit to PATH (optional)

To invoke `amg-kit` from any working directory:

```bash
export AMG_KIT_HOME="$(cd /path/to/amg-kit && pwd)"
export PATH="$AMG_KIT_HOME:$PATH"
```

Replace `/path/to/amg-kit` with the actual installation directory.

To make the configuration persistent, add these lines to `~/.bashrc` and reload it:

```bash
source ~/.bashrc
```

Keep the repository's `binaries/` and `db/` directories alongside the main executable because the script locates these resources relative to its own location.

## Input files

`amg-kit` accepts five positional arguments:

| Argument | Description |
|---|---|
| `bed` | BED-formatted gene coordinates used for conserved-region detection |
| `vann` | Viral gene annotation table, including protein identifiers and VOG annotations |
| `kegg` | KEGG/KO annotation table |
| `pfam` | Pfam annotation table |
| `project` | Output project directory |

### Input preparation

Prepare the input files using consistent contig identifiers and protein identifiers across all annotation tables.

**BED file**

The BED file must contain the genomic coordinates and gene identifiers expected by the script. The implementation uses the first column for contig identifiers and the fourth column for protein identifiers.

**Viral annotation table (`vann`)**

This table supplies viral gene annotations, including the protein identifier and VOG annotation fields expected by the pipeline.

**KEGG annotation table (`kegg`)**

The KEGG table supplies protein identifiers and KO identifiers. The script adds a `protein_id` / `kofam_id` header when processing this input.

**Pfam annotation table (`pfam`)**

The Pfam table must contain the `protein_id` column and the annotation field expected by the pipeline.

> Important: Column order and identifier consistency matter. Verify the input schemas against the implementation before running a large dataset. The script expects particular fields rather than automatically detecting arbitrary annotation formats.

## Usage

Run the program with the following syntax:

```bash
amg-kit [options] <bed> <vann> <kegg> <pfam> <project>
```

When invoking the executable from the repository directory:

```bash
./amg-kit [options] <bed> <vann> <kegg> <pfam> <project>
```

### Options

| Option | Default | Description |
|---|---:|---|
| `-d INT` | `5` | Maximum gene distance used for DBSCAN-based conserved-region detection |
| `-r INT` | `5` | Minimum number of genes required to define a conserved viral region |

The precise interpretation of distance follows the bundled `range-tk enrich` implementation.

### Example

Assume the following input files:

```text
test-data/
├── SRR4408051.bed
├── kegg.txt
├── pfam.txt
└── vog.txt
```

Run `amg-kit` with default clustering parameters:

```bash
./amg-kit \
    test-data/SRR4408051.bed \
    test-data/vog.txt \
    test-data/kegg.txt \
    test-data/pfam.txt \
    test-results
```

Run with explicitly specified parameters:

```bash
./amg-kit \
    -d 5 \
    -r 5 \
    test-data/SRR4408051.bed \
    test-data/vog.txt \
    test-data/kegg.txt \
    test-data/pfam.txt \
    test-results
```

Use absolute paths if inputs and outputs are located in different directories. Ensure that the output directory's parent exists and is writable.

## Output files

The pipeline creates intermediate and result directories under the specified project directory.

```text
test-results/
├── dbscan/
│   ├── cluster.bed
│   ├── label.tsv
│   ├── kegg.tsv
│   ├── pfam.tsv
│   └── annotation.tsv
└── report/
    └── AMGs.txt
```

| File | Description |
|---|---|
| `dbscan/cluster.bed` | Genes assigned to detected conserved regions |
| `dbscan/label.tsv` | Mapping between viral contigs, protein identifiers, and cluster identifiers |
| `dbscan/kegg.tsv` | Processed KEGG/KO annotations |
| `dbscan/pfam.tsv` | Processed Pfam annotations |
| `dbscan/annotation.tsv` | Integrated viral, VOG, KEGG, and Pfam annotation table |
| `report/AMGs.txt` | Candidate AMG labeling results |

The exact contents and column definitions depend on the input annotations and bundled database version.

## Interpretation

An AMG candidate is a viral-associated gene with a potential metabolic function that may contribute to host–virus metabolic interactions.

However, a functional annotation alone does not establish that a gene is an AMG. For downstream analyses, consider additional evidence:

- Viral genomic context and neighboring genes.
- Functional plausibility of the proposed metabolic role.
- Exclusion of viral housekeeping genes and genes associated with cellular contamination.
- Support from independent annotation databases and established AMG-calling criteria.

Treat `AMGs.txt` as a candidate list that may require further validation rather than as definitive evidence of metabolic activity.

## Troubleshooting

### Command not found

Check that the executable has execute permission and that the required programs are available on `PATH`.

```bash
chmod +x amg-kit
command -v perl
command -v bash
command -v bedtools
command -v tsv-utils
```

### Missing database files

Confirm that `db/` and its expected subdirectories remain in the repository installation directory.

### Empty or unexpected results

Check the following:

1. Contig identifiers match between the BED and annotation files.
2. Protein identifiers match across the KEGG, Pfam, and viral annotation tables.
3. The input columns follow the expected schemas.
4. The clustering parameters are appropriate for the gene density and genome organization of the dataset.
5. The bundled AMG database contains the expected reference identifiers.

### Reproducibility

Record the Git commit, database versions, annotation software versions, input checksums, and clustering parameters for each analysis.

## License

This project is distributed under the MIT License. See [LICENSE](LICENSE).
