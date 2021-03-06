# alignment-nf

## Nextflow pipeline for BAM realignment or fastq alignment
[![CircleCI](https://circleci.com/gh/IARCbioinfo/alignment-nf.svg?style=svg)](https://circleci.com/gh/IARCbioinfo/alignment-nf)
[![Docker Hub](https://img.shields.io/badge/docker-ready-blue.svg)](https://hub.docker.com/r/iarcbioinfo/alignment-nf/)
[![https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg](https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg)](https://singularity-hub.org/collections/1404)
[![DOI](https://zenodo.org/badge/.svg)](https://zenodo.org/badge/latestdoi/)


![Workflow representation](WESpipeline.png?raw=true "Scheme of alignment/realignment Workflow")

## Description

Nextflow pipeline to perform BAM realignment or fastq alignment, with/without local indel realignment and base quality score recalibration.

## Dependencies

1. Nextflow : for common installation procedures see the [IARC-nf](https://github.com/IARCbioinfo/IARC-nf) repository.

### Basic fastq alignment
2. [*bwa*](https://github.com/lh3/bwa)
3. [*samblaster*](https://github.com/GregoryFaust/samblaster)
4. [*sambamba*](https://github.com/lomereiter/sambamba)

### BAM files realignment
5. [*samtools*](http://samtools.sourceforge.net/)

### Adapter sequence trimming
6. [*AdapterRemoval*](https://github.com/MikkelSchubert/adapterremoval)

### ALT contigs handling
7. the *k8* javascript execution shell (e.g., available in the [*bwakit*](https://sourceforge.net/projects/bio-bwa/files/bwakit/) archive); must be in the PATH
8. javascript bwa-postalt.js and the additional fasta reference *.alt* file from [*bwakit*](https://github.com/lh3/bwa/tree/master/bwakit) must be in the same directory as the reference genome file.

### QC
9. [Qualimap](http://qualimap.bioinfo.cipf.es). 
10. [Multiqc](http://multiqc.info). 

### Base quality score recalibration
11. [GATK4](https://software.broadinstitute.org/gatk/guide/quickstart); wrapper 'gatk' must be in the path
12. [GATK bundle](https://software.broadinstitute.org/gatk/download/bundle) VCF files with lists of indels and SNVs (recommended: Mills gold standard indels VCFs, dbsnp VCF), and corresponding tabix indexes (.tbi)

To avoid installing the previous tools, install Docker. Docker installation is described in the [IARC-nf](https://github.com/IARCbioinfo/IARC-nf) repository.

## Input 
 | Type      | Description     |
  |-----------|---------------|
  | --input_folder    | a folder with fastq files or bam files |

## Parameters

* #### Mandatory

| Name | Example value | Description |
|-----------|--------------|-------------| 
|--ref    | hg19.fasta | genome reference  with its index files (*.fai*, *.sa*, *.bwt*, *.ann*, *.amb*, *.pac*, and *.dict*; in the same directory) |
|--output_folder   | . | Output folder for aligned BAMs|

* #### Optional

| Name | Default value | Description |
|-----------|--------------|-------------| 
|--input_file   | null | Input file (comma-separated) with 3 columns: sample name, read_group_ID, and file path. |
|--cpu          | 8 | number of CPUs |
|--mem         | 32 | memory|
|--mem\_sambamba | 1 | memory for software *sambamba*|
|--RG           | PL:ILLUMINA | sequencing information for aligned (for *bwa*)|
|--fastq_ext    | fastq.gz | extension of fastq files|
|--suffix1      | \_1 | suffix for second element of read files pair|
|--suffix2      | \_2 | suffix for second element of read files pair|
|--bed    | | bed file with interval list|
|--snp_vcf  | dbsnp.vcf | path to SNP VCF from GATK bundle (default : dbsnp.vcf) |
|--indel_vcf  | Mills_1000G_indels.vcf | path to indel VCF from GATK bundle (default : Mills_1000G_indels.vcf) |
|--postaltjs    | bwa-postalt.js" | path to postalignment javascript *bwa-postalt.js*|


* #### Flags

Flags are special parameters without value.

| Name  | Description |
|-----------|-------------| 
| --help | print usage and optional parameters |
|--trim     | enable adapter sequence trimming|
|--recalibration  | perform quality score recalibration (GATK)|
|--alt         | enable alternative contig handling (for reference genome hg38)|
|--bwa_option_M  | Trigger the -M option in bwa and the corresponding compatibility option in samblaster (marks shorter split hits as secondary) |

## Usage
```bash
nextflow run iarcbioinfo/alignment-nf --input_folder input --fasta_ref hg19.fasta --out_folder output
```
### Enable adapter trimming
To use the adapter trimming step, you must add the ***--trim* option**, as well as satisfy the requirements above mentionned. For example:
```bash
nextflow run iarcbioinfo/alignment-nf --input_folder input --fasta_ref reference/hs38DH.fa -out_folder output --trim
```

### Enable ALT mode
To use the alternative contigs handling mode, you must provide the **path to an ALT aware genome reference** (e.g., hg38) AND add the ***--alt* option**, as well as satisfy the requirements above mentionned. For example:
```bash
nextflow run iarcbioinfo/alignment-nf --input_folder input --fasta_ref reference/hs38DH.fa --js /user/bin/k8/k8 --postaltjs /user/bin/bwa-0.7.15/bwakit/bwa-postalt.js -out_folder output --alt
```

### Enable base quality score recalibration
To use the base quality score recalibration step, you must provide the **path to 2 GATK bundle VCF files** with lists of known snps and indels, respectively, AND add the ***--recalibration* option**, as well as satisfy the requirements above mentionned. For example:
```bash
nextflow run iarcbioinfo/alignment-nf --snp_vcf GATKbundle/dbsnp.vcf.gz --input_folder input --fasta_ref reference/hg19.fa --GATK_folder /user/bin7GATK-3.6-0 --intervals reference/hg19_intervals.bed --out_folder output --recalibration
```

## Output 
  | Type      | Description     |
  |-----------|---------------|
  | BAM/    | folder with BAM and BAI files of alignments or realignments |
  | QC/qualimap/multiqc_qualimap_flagstat_report.html  | multiQC report for qualimap and samtools flagstat (duplicates) |
  | QC/qualimap/multiqc_qualimap_flagstat_report_data  | data used for the multiQC report |
  | QC/qualimap/file_BQSRecalibrated.stats.txt  | qualimap summary file |
  | QC/qualimap/file_BQSRecalibrated/  | qualimap files |
  | QC/BAM/BQSR/  | GATK base quality score recalibration outputs (tables and pdf comparing scores before/after recalibration)|
  
## Directed Acyclic Graph
[![DAG](dag.png)](http://htmlpreview.github.io/?https://github.com/IARCbioinfo/alignment-nf/blob/dev/dag.html)

## FAQ
### Why did Indel realignment disappear from version 1.0?
Indel realignment was removed following new GATK best practices for pre-processing.

## Contributions

  | Name      | Email | Description     |
  |-----------|---------------|-----------------| 
  | Nicolas Alcala*    | AlcalaN@fellows.iarc.fr    | Developer to contact for support |
  | Catherine Voegele    |     VoegeleC@iarc.fr | Tester |
  | Vincent Cahais | CahaisV@iarc.fr | Tester |
  | Alexis Robitaille | RobitailleA@students.iarc.fr | Tester |
  
