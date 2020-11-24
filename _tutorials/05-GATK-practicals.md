---
title: GATK Tutorials
---

### Description
*GATK*: GATK4 toolkit offers a wide variety of tools with a primary focus on variant discovery and genotyping. The content on this page is borrowed from GATK webpages/courses. To get familiar with GATK tools, you can read the following:

- Read overview of GATK4 [here](https://software.broadinstitute.org/gatk/gatk4)
- Quick start guide [here](https://software.broadinstitute.org/gatk/documentation/quickstart)
- Best Practices workflows [here](https://software.broadinstitute.org/gatk/best-practices/)

### Download GATK container from Dockerhub

```docker pull broadinstitute/gatk:latest``` <br>

  or with some specific-version information  <br>
```docker pull broadinstitute/gatk:4.0.11.0```

### Start up the GATK container

```docker run -it broadinstitute/gatk:latest```

### Run a GATK command in the container

```./gatk --list```

### The general format for GATK commands is
gatk ToolName [tool args]

### exit from the container

```ctrl+p then ctrl+q```

### Download example data
You can download toy example dataset from CSC's allas objects storage, unzip it and check the path to data:

```bash
wget https://a3s.fi/Softwares/data.zip
unzip data.zip
pwd
```

### Start running gatk container and mount the location of the data bundle inside the docker. Replace /path/data with the path to data.

``` docker run -v /path/data:/gatk/data -it broadinstitute/gatk:latest ```


### Get usage information of HaplotypeCaller from GATK

```gatk HaplotypeCaller --help```

### Run HaplotypeCaller

``` gatk HaplotypeCaller -R /gatk/data/ref/ref.fasta -I data/bams/mother.bam \ ``` <br>
``` -O /gatk/data/sandbox/variants.vcf ```

> Note: Add JVM options to the command if you run into memory issues

> ``` gatk --java-options "-Xmx4G" HaplotypeCaller \ ``` <br>
> ``` -R /gatk/data/ref/ref.fasta -I /gatk/data/bams/mother.bam \ ``` <br>
> ```-O /gatk/data/sandbox/variants.vcf ```


### Run GVCF workflow tools using HaplotypeCaller, GenomicsDBImport and then GenotypeGVCFs to perform joint calling on multiple input samples.

### Run HaplotypeCaller on three input bams (mother, father, son)

``` gatk HaplotypeCaller -R /gatk/data/ref/ref.fasta -I /gatk/data/bams/mother.bam -O /gatk/data/sandbox/mother.g.vcf -ERC GVCF ```

``` gatk HaplotypeCaller -R /gatk/data/ref/ref.fasta -I /gatk/data/bams/father.bam -O /gatk/data/sandbox/father.g.vcf -ERC GVCF ```

```gatk HaplotypeCaller -R /gatk/data/ref/ref.fasta -I /gatk/data/bams/son.bam -O /gatk/data/sandbox/son.g.vcf -ERC GVCF ```

### Run GenomicsDBImport on three GVCFs to consolidate

``` gatk GenomicsDBImport -V /gatk/data/sandbox/mother.g.vcf \ ``` <br>
``` -V /gatk/data/sandbox/father.g.vcf \ ``` <br>
``` -V /gatk/data/sandbox/son.g.vcf --genomicsdb-workspace-path \ ``` <br>
```//data/sandbox/trio.gdb_workspace --intervals 20 ```

### Alternatively, use CombinedGVCFs command as an alternative to GenomicsDBImport

```gatk CombineGVCFs -R /gatk/data/ref/ref.fasta \  ``` <br>
```-V /gatk/data/sandbox/father.g.vcf \  ``` <br>
```-V /gatk/data/sandbox/mother.g.vcf  -V /gatk/data/sandbox/son.g.vcf \ ``` <br>
``` -O /gatk/data/sandbox/combine_trio_variants.vcf ```

### Run GenotypeGVCFs on the GDB workspace to produce final multisample VCF

``` gatk GenotypeGVCFs -R /gatk/data/ref/ref.fasta \   ``` <br>
``` -V gendb://data/sandbox/trio.gdb_workspace \  ``` <br>
```-G StandardAnnotation -O /gatk/data/sandbox/trio_variants.vcf ``` <br>

