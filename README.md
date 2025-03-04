# gget
[![pypi version](https://img.shields.io/pypi/v/gget)](https://pypi.org/project/gget)
[![Downloads](https://pepy.tech/badge/gget)](https://pepy.tech/project/gget)
[![license](https://img.shields.io/pypi/l/gget)](LICENSE)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.6534041.svg)](https://doi.org/10.5281/zenodo.6534041)
![status](https://github.com/pachterlab/gget/workflows/CI/badge.svg)
![Code Coverage](https://img.shields.io/badge/Coverage-84%25-green.svg)  

`gget` is a free and open-source command-line tool and Python package that enables efficient querying of genomic databases. `gget`  consists of a collection of separate but interoperable modules, each designed to facilitate one type of database querying in a single line of code.  

*If you use `gget` in a publication, please [cite*](#cite):*    
Luebbert, L. & Pachter, L. (2022). Efficient querying of genomic databases for single-cell RNA-seq with gget. bioRxiv 2022.05.17.492392; doi: https://doi.org/10.1101/2022.05.17.492392
  
![alt text](https://github.com/pachterlab/gget/blob/main/figures/gget_overview.png?raw=true)
  
`gget` currently consists of the following nine modules:
- [`gget ref`](#gget-ref)  
Fetch File Transfer Protocols (FTPs) and metadata for reference genomes and annotations from [Ensembl](https://www.ensembl.org/) by species.
- [`gget search`](#gget-search)   
Fetch genes and transcripts from [Ensembl](https://www.ensembl.org/) using free-form search terms.
- [`gget info`](#gget-info)  
Fetch extensive gene and transcript metadata from [Ensembl](https://www.ensembl.org/), [UniProt](https://www.uniprot.org/), and [NCBI](https://www.ncbi.nlm.nih.gov/) using Ensembl IDs.  
- [`gget seq`](#gget-seq)  
Fetch nucleotide or amino acid sequences of genes or transcripts from [Ensembl](https://www.ensembl.org/) or [UniProt](https://www.uniprot.org/), respectively.  
- [`gget blast`](#gget-blast)  
BLAST a nucleotide or amino acid sequence to any [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi) database.
- [`gget blat`](#gget-blat)  
Find the genomic location of a nucleotide or amino acid sequence using [BLAT](https://genome.ucsc.edu/cgi-bin/hgBlat).
- [`gget muscle`](#gget-muscle)  
Align multiple nucleotide or amino acid sequences to each other using [Muscle5](https://www.drive5.com/muscle/).
- [`gget enrichr`](#gget-enrichr)  
Perform an enrichment analysis on a list of genes using [Enrichr](https://maayanlab.cloud/Enrichr/).
- [`gget archs4`](#gget-archs4)  
Find the most correlated genes to a gene of interest or find the gene's tissue expression atlas using [ARCHS4](https://maayanlab.cloud/archs4/).


## Installation
```bash
pip install gget
```
or
```bash
conda install -c bioconda gget
```

For use in Jupyter Lab / Google Colab:
```python
import gget
```

## Quick start guide
```bash
# Fetch all Homo sapiens reference and annotation FTPs from the latest Ensembl release
$ gget ref -s homo_sapiens

# Search human genes with "ace2" AND "angiotensin" in their name/description
$ gget search -sw ace2,angiotensin -s homo_sapiens -ao and 

# Look up gene ENSG00000130234 (ACE2) with expanded info (returns all transcript isoforms for genes)
$ gget info -id ENSG00000130234 -e

# Fetch the amino acid sequence of the canonical transcript of gene ENSG00000130234
$ gget seq -id ENSG00000130234 --seqtype transcript

# Quickly find the genomic location of (the start of) that amino acid sequence
$ gget blat -seq MSSSSWLLLSLVAVTAAQSTIEEQAKTFLDKFNHEAEDLFYQSSLAS

# Blast (the start of) that amino acid sequence
$ gget blast -seq MSSSSWLLLSLVAVTAAQSTIEEQAKTFLDKFNHEAEDLFYQSSLAS

# Align nucleotide or amino acid sequences stored in a FASTA file
$ gget muscle -fa path/to/file.fa

# Use Enrichr to find the ontology of a list of genes
$ gget enrichr -g ACE2 AGT AGTR1 ACE AGTRAP AGTR2 ACE3P -db ontology

# Get the human tissue expression atlas of gene ACE2
$ gget archs4 -g ACE2 -w tissue
```
Jupyter Lab / Google Colab:
```python  
gget.ref("homo_sapiens")
gget.search(["ace2", "angiotensin"], "homo_sapiens", andor="and")
gget.info("ENSG00000130234", expand=True)
gget.seq("ENSG00000130234", seqtype="transcript")
gget.blat("MSSSSWLLLSLVAVTAAQSTIEEQAKTFLDKFNHEAEDLFYQSSLAS")
gget.blast("MSSSSWLLLSLVAVTAAQSTIEEQAKTFLDKFNHEAEDLFYQSSLAS")
gget.muscle("path/to/file.fa")
gget.enrichr(["ACE2", "AGT", "AGTR1", "ACE", "AGTRAP", "AGTR2", "ACE3P"], database="ontology", plot=True)
gget.archs4("ACE2", which="tissue")
```
___

# Manual
Jupyter Lab / Google Colab arguments are equivalent to long-option arguments (`--arg`).  
The manual for any gget tool can be called from terminal using the `-h` `--help` flag.

## gget ref
Fetch FTPs and their respective metadata (or use flag `ftp` to only return the links) for reference genomes and annotations from [Ensembl](https://www.ensembl.org/) by species.  
Return format: dictionary/json.

**Required arguments**  
`-s` `--species`  
Species for which the FTPs will be fetched in the format genus_species, e.g. homo_sapiens.  
Note: Not required when calling flag [--list_species].   
Supported shortcuts: 'human', 'mouse'

**Optional arguments**  
`-w` `--which`  
Defines which results to return. Default: 'all' -> Returns all available results.  
Possible entries are one or a combination of the following:  
'gtf' - Returns the annotation (GTF).  
'cdna' - Returns the trancriptome (cDNA).  
'dna' - Returns the genome (DNA).  
'cds' - Returns the coding sequences corresponding to Ensembl genes. (Does not contain UTR or intronic sequence.)  
'cdrna' - Returns transcript sequences corresponding to non-coding RNA genes (ncRNA).  
'pep' - Returns the protein translations of Ensembl genes.  

`-r` `--release`  
Defines the Ensembl release number from which the files are fetched, e.g. 104. Default: latest Ensembl release.  

`-o` `--out`    
Path to the json file the results will be saved in, e.g. path/to/directory/results.json. Default: Standard out.  
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.

**Flags**  
`-l` `--list_species`   
Lists all available species. (Jupyter Lab / Google Colab: combine with `species=None`.)  

`-ftp` `--ftp`   
Returns only the requested FTP links.  

`-d` `--download`   
Downloads the requested FTPs to the current directory (requires [curl](https://curl.se/docs/) to be installed).  

  
### Examples
**Use `gget ref` in combination with [kallisto | bustools](https://www.kallistobus.tools/kb_usage/kb_ref/) to build a reference index:**
```bash
kb ref -i INDEX -g T2G -f1 FASTA $(gget ref --ftp -w dna,gtf -s homo_sapiens)
```
&rarr; kb ref builds a reference index using the latest DNA and GTF files of species **Homo sapiens** passed to it by `gget ref`.  
  
Get all available genomes:  
```bash
gget ref --list -r 103
```
```python
# Jupyter Lab / Google Colab:
gget.ref(species=None, list_species=True, release=103)
```
&rarr; Returns a list with all available genomes (checks if GTF and FASTAs are available) from Ensembl release 103.   
(If no release is specified, `gget ref` will always return information from the latest Ensembl release.)  
  
Get the genome reference for a specific species:   
```bash
gget ref -s homo_sapiens -w gtf dna
```
```python
# Jupyter Lab / Google Colab:
gget.ref("homo_sapiens", which=["gtf", "dna"])
```
&rarr; Returns a json with the latest human GTF and FASTA FTPs, and their respective metadata, in the format:
```
{
    "homo_sapiens": {
        "annotation_gtf": {
            "ftp": "http://ftp.ensembl.org/pub/release-106/gtf/homo_sapiens/Homo_sapiens.GRCh38.106.gtf.gz",
            "ensembl_release": 106,
            "release_date": "28-Feb-2022",
            "release_time": "23:27",
            "bytes": "51379459"
        },
        "genome_dna": {
            "ftp": "http://ftp.ensembl.org/pub/release-106/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz",
            "ensembl_release": 106,
            "release_date": "21-Feb-2022",
            "release_time": "09:35",
            "bytes": "881211416"
        }
    }
}
```

#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget search   
Fetch genes and transcripts from [Ensembl](https://www.ensembl.org/) using free-form search terms.   
Return format: data frame.

**Required arguments**  
`-sw` `--searchwords`   
One or more free form search words, e.g. gaba, nmda. (Note: Search is not case-sensitive.)  

`-s` `--species`  
Species or database to be searched.  
A species can be passed in the format 'genus_species', e.g. 'homo_sapiens'.  
To pass a specific database, pass the name of the CORE database, e.g. 'mus_musculus_dba2j_core_105_1'.  
All availabale databases can be found [here](http://ftp.ensembl.org/pub/release-106/mysql/).  
Supported shortcuts: 'human', 'mouse'. 

**Optional arguments**  
`-st` `--seqtype`  
'gene' (default) or 'transcript'  
Returns genes or transcripts, respectively.

`-ao` `--andor`  
'or' (default) or 'and'  
'or': Returns all genes that INCLUDE AT LEAST ONE of the searchwords in their name/description.  
'and': Returns only genes that INCLUDE ALL of the searchwords in their name/description.

`-l` `--limit`   
Limits the number of search results, e.g. 10. Default: None.  

`-o` `--out`  
Path to the csv the results will be saved in, e.g. path/to/directory/results.csv. Default: Standard out.   
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.

**Flags**  
`wrap_text`  
Jupyter Lab / Google Colab only. `wrap_text=True` displays data frame with wrapped text for easy reading (default: False).  
  
    
### Example
```bash
gget search -sw gaba gamma-aminobutyric -s homo_sapiens
```
```python
# Jupyter Lab / Google Colab:
gget.search(["gaba", "gamma-aminobutyric"], "homo_sapiens")
```
&rarr; Returns all genes that contain at least one of the search words in their name or Ensembl/external reference description:

| ensembl_id     | gene_name     | ensembl_description     | ext_ref_description        | biotype | url |
| -------------- |-------------------------| ------------------------| -------------- | ----------|-----|
| ENSG00000034713| GABARAPL2 | 	GABA type A receptor associated protein like 2 [Source:HGNC Symbol;Acc:HGNC:13291] | GABA type A receptor associated protein like 2 | protein_coding | https://uswest.ensembl.org/homo_sapiens/Gene/Summary?g=ENSG00000034713 |
| . . .            | . . .                     | . . .                     | . . .            | . . .       | . . . |
    
#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget info  
Fetch extensive gene and transcript metadata from [Ensembl](https://www.ensembl.org/), [UniProt](https://www.uniprot.org/), and [NCBI](https://www.ncbi.nlm.nih.gov/) using Ensembl IDs.  
Return format: data frame.

**Required arguments**  
`-id` `--ens_ids`   
One or more Ensembl IDs.

**Optional arguments**  
`-o` `--out`   
Path to the csv the results will be saved in, e.g. path/to/directory/results.csv. Default: Standard out.    
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.

**Flags**  
`-e` `--expand`   
Expands returned information (only for gene and transcript IDs).   
For genes, adds information on all known transcripts.  
For transcripts, adds information on all known translations and exons.

`wrap_text`  
Jupyter Lab / Google Colab only. `wrap_text=True` displays data frame with wrapped text for easy reading (default: False).  

  
### Example
```bash
gget info -id ENSG00000034713 ENSG00000104853 ENSG00000170296 -e 
```
```python
# Jupyter Lab / Google Colab:
gget.info(["ENSG00000034713", "ENSG00000104853", "ENSG00000170296"], expand=True)
```
&rarr; Returns extensive information about each requested Ensembl ID in data frame format:  

|      | uniprot_id     | ncbi_gene_id     | primary_gene_name | synonyms | protein_names | ensembl_description | uniprot_description | ncbi_description | biotype | canonical_transcript | ... |
| -------------- |-------------------------| ------------------------| -------------- | ----------|-----|----|----|----|----|----|----|
| ENSG00000034713| P60520 | 11345 | GABARAPL2 | [ATG8, ATG8C, FLC3A, GABARAPL2, GATE-16, GATE16, GEF-2, GEF2] | Gamma-aminobutyric acid receptor-associated protein like 2 (GABA(A) receptor-associated protein-like 2)... | GABA type A receptor associated protein like 2 [Source:HGNC Symbol;Acc:HGNC:13291] | FUNCTION: Ubiquitin-like modifier involved in intra- Golgi traffic (By similarity). Modulates intra-Golgi transport through coupling between NSF activity and ... | Enables ubiquitin protein ligase binding activity. Involved in negative regulation of proteasomal protein catabolic process and protein... | protein_coding | ENST00000037243.7 |... |
| . . .            | . . .                     | . . .                     | . . .            | . . .       | . . . | . . . | . . . | . . . | . . . | . . . | ... |
  
#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget seq  
Fetch nucleotide or amino acid sequence of a gene (and all its isoforms) or a transcript by Ensembl ID.   
Return format: FASTA.

**Required arguments**  
`-id` `--ens_ids`   
One or more Ensembl IDs.

**Optional arguments**  
`-st` `--seqtype`  
'gene' (default) or 'transcript'.  
Defines whether nucleotide or amino acid sequences are returned.  
Nucleotide sequences are fetched from [Ensembl](https://www.ensembl.org/).  
Amino acid sequences are fetched from [UniProt](https://www.uniprot.org/).

`-o` `--out`   
Path to the file the results will be saved in, e.g. path/to/directory/results.fa. Default: Standard out.   
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.

**Flags**  
`-i` `--isoforms`   
Returns the sequences of all known transcripts.  
(Only for gene IDs in combination with `seqtype=transcript`.)
   
  
### Examples  
```bash
gget seq -id ENSG00000034713 ENSG00000104853 ENSG00000170296
```
```python
# Jupyter Lab / Google Colab:
gget.seq(["ENSG00000034713", "ENSG00000104853", "ENSG00000170296"])
```
&rarr; Returns the nucleotide sequences of ENSG00000034713, ENSG00000104853, and ENSG00000170296 in FASTA format.  
  

```bash
gget seq -id ENSG00000034713 -st transcript -iso
```
```python
# Jupyter Lab / Google Colab:
gget.seq("ENSG00000034713", seqtype="transcript", isoforms=True)
```
&rarr; Returns the amino acid sequences of all known transcripts of ENSG00000034713 in FASTA format.

#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget blast
BLAST a nucleotide or amino acid sequence to any [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi) database.  
Return format: data frame.

**Required arguments**  
`-seq` `--sequence`   
Nucleotide or amino acid sequence, or path to FASTA or .txt file.

**Optional arguments**  
`-p` `--program`  
'blastn', 'blastp', 'blastx', 'tblastn', or 'tblastx'.  
Default: 'blastn' for nucleotide sequences; 'blastp' for amino acid sequences.

`-db` `--database`  
'nt', 'nr', 'refseq_rna', 'refseq_protein', 'swissprot', 'pdbaa', or 'pdbnt'.  
Default: 'nt' for nucleotide sequences; 'nr' for amino acid sequences.  
[More info on BLAST databases](https://ncbi.github.io/blast-cloud/blastdb/available-blastdbs.html)

`-l` `--limit`  
Limits number of hits to return. Default: 50.  

`-e` `--expect`  
Defines the [expect value](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=FAQ#expect) cutoff. Default: 10.0.  

`-o` `--out`   
Path to the csv the results will be saved in, e.g. path/to/directory/results.csv. Default: Standard out.   
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.

**Flags**  
`-lcf` `--low_comp_filt`  
Turns on [low complexity filter](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=FAQ#LCR).  

`-mbo` `--megablast_off`  
Turns off MegaBLAST algorithm. Default: MegaBLAST on (blastn only).  

`-q` `--quiet`   
Prevents progress information from being displayed.  

`wrap_text`  
Jupyter Lab / Google Colab only. `wrap_text=True` displays data frame with wrapped text for easy reading (default: False).   
  
### Example
```bash
gget blast -seq MKWMFKEDHSLEHRCVESAKIRAKYPDRVPVIVEKVSGSQIVDIDKRKYLVPSDITVAQFMWIIRKRIQLPSEKAIFLFVDKTVPQSR
```
```python
# Jupyter Lab / Google Colab:
gget.blast("MKWMFKEDHSLEHRCVESAKIRAKYPDRVPVIVEKVSGSQIVDIDKRKYLVPSDITVAQFMWIIRKRIQLPSEKAIFLFVDKTVPQSR")
```
&rarr; Returns the BLAST result of the sequence of interest in data frame format. `gget blast` automatically detects this sequence as an amino acid sequence and therefore sets the BLAST program to *blastp* with database *nr*.  

| Description     | Scientific Name	     | Common Name     | Taxid        | Max Score | Total Score | Query Cover | ... |
| -------------- |-------------------------| ------------------------| -------------- | ----------|-----|---|---|
| PREDICTED: gamma-aminobutyric acid receptor-as...| Colobus angolensis palliatus	 | 	NaN | 336983 | 180	 | 180 | 100% | ... |
| . . . | . . . | . . . | . . . | . . . | . . . | . . . | ... | 

BLAST from .fa or .txt file:  
```bash
gget blast -seq fasta.fa
```
```python
# Jupyter Lab / Google Colab:
gget.blast("fasta.fa")
```
&rarr; Returns the BLAST results of the first sequence contained in the fasta.fa file. 

#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget blat
Find the genomic location of a nucleotide or amino acid sequence using [BLAT](https://genome.ucsc.edu/cgi-bin/hgBlat).   
Return format: data frame.

**Required arguments**  
`-seq` `--sequence`   
Nucleotide or amino acid sequence, or path to FASTA or .txt file.

**Optional arguments**  
`-st` `--seqtype`    
'DNA', 'protein', 'translated%20RNA', or 'translated%20DNA'.   
Default: 'DNA' for nucleotide sequences; 'protein' for amino acid sequences.  

`-a` `--assembly`  
'human' (hg38) (default), 'mouse' (mm39), 'zebrafinch' (taeGut2),   
or any of the species assemblies available [here](https://genome.ucsc.edu/cgi-bin/hgBlat) (use short assembly name).

`-o` `--out`   
Path to the csv the results will be saved in, e.g. path/to/directory/results.csv. Default: Standard out.   
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.  
  
  
### Example
```bash
gget blat -seq MKWMFKEDHSLEHRCVESAKIRAKYPDRVPVIVEKVSGSQIVDIDKRKYLVPSDITVAQFMWIIRKRIQLPSEKAIFLFVDKTVPQSR -a taeGut2
```
```python
# Jupyter Lab / Google Colab:
gget.blat("MKWMFKEDHSLEHRCVESAKIRAKYPDRVPVIVEKVSGSQIVDIDKRKYLVPSDITVAQFMWIIRKRIQLPSEKAIFLFVDKTVPQSR", assembly="taeGut2")
```
&rarr; Returns BLAT results for assembly taeGut2 (zebra finch) in data frame format. In the above example, `gget blat` automatically detects this sequence as an amino acid sequence and therefore sets the BLAT seqtype to *protein*.

| genome     | query_size     | aligned_start     | aligned_end        | matches | mismatches | %_aligned | ... |
| -------------- |-------------------------| ------------------------| -------------- | ----------|-----|---|---|
| taeGut2| 88 | 	12 | 88 | 77 | 0 | 87.5 | ... |

#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget muscle  
Align multiple nucleotide or amino acid sequences to each other using [Muscle5](https://www.drive5.com/muscle/).  
Return format: ClustalW formatted standard out or aligned FASTA.  

**Required arguments**  
`-fa` `--fasta`   
Path to FASTA or .txt file containing the nucleotide or amino acid sequences to be aligned.  

**Optional arguments**  
`-o` `--out`   
Path to the aligned FASTA file the results will be saved in, e.g. path/to/directory/results.afa. Default: Standard out.   
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.

**Flags**  
`-s5` `--super5`  
Aligns input using the [Super5 algorithm](https://drive5.com/muscle5/Muscle5_SuppMat.pdf) instead of the [Parallel Perturbed Probcons (PPP) algorithm](https://drive5.com/muscle5/Muscle5_SuppMat.pdf) to decrease time and memory.  
Use for large inputs (a few hundred sequences).

`wrap_text`  
Jupyter Lab / Google Colab only. `wrap_text=True` displays data frame with wrapped text for easy reading (default: False).   
   
   
### Example
```bash
gget muscle -fa fasta.fa
```
```python
# Jupyter Lab / Google Colab:
gget.muscle("fasta.fa")
```
&rarr; Returns an overview of the aligned sequences with ClustalW coloring. (To return an aligned FASTA (.afa) file, use `--out` argument (or `save=True` in Jupyter Lab/Google Colab).) In the above example, the 'fasta.fa' includes several sequences to be aligned (e.g. isoforms returned from `gget seq`). 

![alt text](https://github.com/pachterlab/gget/blob/main/figures/example_muscle_return.png?raw=true)

#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget enrichr
Perform an enrichment analysis on a list of genes using [Enrichr](https://maayanlab.cloud/Enrichr/).  
Return format: data frame.

**Required arguments**  
`-g` `--genes`  
Short names (gene symbols) of genes to perform enrichment analysis on, e.g. 'PHF14 RBM3 MSL1 PHF21A'.  

`-db` `--database`  
Database to use as reference for the enrichment analysis.  
Supports any database listed [here](https://maayanlab.cloud/Enrichr/#libraries) under 'Gene-set Library' or one of the following shortcuts:  <br />
'pathway'       (KEGG_2021_Human)  
'transcription'     (ChEA_2016)  
'ontology'      (GO_Biological_Process_2021)  
'diseases_drugs'   (GWAS_Catalog_2019)   
'celltypes'      (PanglaoDB_Augmented_2021)  
'kinase_interactions'   (KEA_2015)  

**Optional arguments**  
`-o` `--out`   
Path to the csv the results will be saved in, e.g. path/to/directory/results.csv. Default: Standard out.   
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.

**Flags**  
`plot`  
Jupyter Lab / Google Colab only. `plot=True` provides a graphical overview of the first 15 results (default: False).  
  
  
### Example
```bash
gget enrichr -g ACE2 AGT AGTR1 -db ontology
```
```python
# Jupyter Lab / Google Colab:
gget.enrichr(["ACE2", "AGT", "AGTR1"], database="ontology", plot=True)
```
&rarr; Returns pathways/functions involving genes ACE2, AGT, and AGTR1 from the *GO Biological Process 2021* database in data frame format. In Jupyter Lab / Google Colab, `plot=True` returns a graphical overview of the results:

![alt text](https://github.com/pachterlab/gget/blob/main/figures/gget_enrichr_results.png?raw=true)

#### [More examples](https://github.com/pachterlab/gget_examples)
___

## gget archs4
Find the most correlated genes to a gene of interest or find the gene's tissue expression atlas using [ARCHS4](https://maayanlab.cloud/archs4/).  
Return format: data frame.  

**Required arguments**  
`-g` `--gene`  
Short name (gene symbol) of gene of interest, e.g. 'STAT4'.

**Optional arguments**  
 `-w` `--which`  
'correlation' (default) or 'tissue'.  
'correlation' returns a gene correlation table that contains the 100 most correlated genes to the gene of interest. The Pearson correlation is calculated over all samples and tissues in [ARCHS4](https://maayanlab.cloud/archs4/).  
'tissue' returns a tissue expression atlas calculated from human or mouse samples (as defined by 'species') in [ARCHS4](https://maayanlab.cloud/archs4/).  

`-s` `--species`  
'human' (default) or 'mouse'.   
Defines whether to use human or mouse samples from [ARCHS4](https://maayanlab.cloud/archs4/).  
(Only for tissue expression atlas.)

`-o` `--out`   
Path to the csv the results will be saved in, e.g. path/to/directory/results.csv. Default: Standard out.   
Jupyter Lab / Google Colab: `save=True` will save the output in the current working directory.  
  
  
### Examples
```bash
gget archs4 -g ACE2
```
```python
# Jupyter Lab / Google Colab:
gget.archs4("ACE2")
```
&rarr; Returns the 100 most correlated genes to ACE2 in a data frame:  

| gene_symbol     | pearson_correlation     |
| -------------- |-------------------------| 
| SLC5A1 | 0.579634 | 	
| CYP2C18 | 0.576577 | 	
| . . . | . . . | 	

```bash
gget archs4 -g ACE2 -w tissue
```
```python
# Jupyter Lab / Google Colab:
gget.archs4("ACE2", which="tissue")
```
&rarr; Returns the tissue expression of ACE2 in a data frame (by default, human data is used):

| id     | min     | q1 |  median | q3 | max |
| ------ |--------| ------ |--------| ------ |--------| 
| System.Urogenital/Reproductive System.Kidney.RENAL CORTEX | 0.113644 | 8.274060 | 9.695840 | 10.51670 | 11.21970 |
| System.Digestive System.Intestine.INTESTINAL EPITHELIAL CELL | 0.113644 | 	5.905560 | 9.570450 | 13.26470 | 13.83590 | 
| . . . | . . . | . . . | . . . | . . . | . . . |

#### [More examples](https://github.com/pachterlab/gget_examples)
___

# Cite 

If you use `gget` in a publication, please cite:   
Luebbert L. & Pachter L. (2022). Efficient querying of genomic databases for single-cell RNA-seq with gget. bioRxiv 2022.05.17.492392; doi: https://doi.org/10.1101/2022.05.17.492392

- If using gget archs4, please also cite:   
  Lachmann A, Torre D, Keenan AB, Jagodnik KM, Lee HJ, Wang L, Silverstein MC, Ma’ayan A. Massive mining of publicly available RNA-seq data from human and mouse. Nature Communications 9. Article number: 1366 (2018), doi:10.1038/s41467-018-03751-6

  Bray NL, Pimentel H, Melsted P and Pachter L, Near optimal probabilistic RNA-seq quantification, Nature Biotechnology 34, p 525--527 (2016). https://doi.org/10.1038/nbt.3519

- If using gget enrichr, please also cite:     
  Chen EY, Tan CM, Kou Y, Duan Q, Wang Z, Meirelles GV, Clark NR, Ma'ayan A.
Enrichr: interactive and collaborative HTML5 gene list enrichment analysis tool. BMC Bioinformatics. 2013; 128(14).  

  Kuleshov MV, Jones MR, Rouillard AD, Fernandez NF, Duan Q, Wang Z, Koplev S, Jenkins SL, Jagodnik KM, Lachmann A, McDermott MG, Monteiro CD, Gundersen GW, Ma'ayan A.
Enrichr: a comprehensive gene set enrichment analysis web server 2016 update. Nucleic Acids Research. 2016; gkw377.  

  Xie Z, Bailey A, Kuleshov MV, Clarke DJB., Evangelista JE, Jenkins SL, Lachmann A, Wojciechowicz ML, Kropiwnicki E, Jagodnik KM, Jeon M, & Ma’ayan A.
Gene set knowledge discovery with Enrichr. Current Protocols, 1, e90. 2021. doi: 10.1002/cpz1.90. 

- If using gget blast, please also cite:  
  Altschul SF, Gish W, Miller W, Myers EW, Lipman DJ. Basic local alignment search tool. J Mol Biol. 1990 Oct 5;215(3):403-10. doi: 10.1016/S0022-2836(05)80360-2. PMID: 2231712.

- If using gget blat, please also cite:   
  Kent WJ. BLAT--the BLAST-like alignment tool. Genome Res. 2002 Apr;12(4):656-64. doi: 10.1101/gr.229202. PMID: 11932250; PMCID: PMC187518.

- If using gget muscle, please also cite:  
  Edgar RC (2021), MUSCLE v5 enables improved estimates of phylogenetic tree confidence by ensemble bootstrapping, bioRxiv 2021.06.20.449169. https://doi.org/10.1101/2021.06.20.449169.
