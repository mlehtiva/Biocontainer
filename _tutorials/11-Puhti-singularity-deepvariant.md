---
title: Running Singularity Applications on Puhti
---

### Deepvariant pipeline as developed by GoogleAI ###
Run [deepvariant method](https://github.com/google/deepvariant)   to perform variant calling on WGS and WES data sets in Puhti supercomputing environment using deepvariant singularity container


## How to get started
One needs the DeepVariant programs (https://github.com/google/deepvariant) and some test data. One can download data from Pouta object storage  (or from google bucket)  Storage.

### Prepare deepvariant singulairty image from docker image
Deepvariant (this example is with version 0.8) uses docker to run the binaries instead of copying binaries to local machines first.

One needs to get deepvariant docker image, models and test data in order to run the pipeline. Additionally, other prerequisites for running deepvariant method includes 1) obtaining A reference genome in [FASTA](https://en.wikipedia.org/wiki/FASTA_format) format and its corresponding index file (.fai). 2) An aligned reads file in [BAM](http://genome.sph.umich.edu/wiki/BAM) format and its corresponding index file (.bai).

### Download deepvariant docker image

```
docker pull gcr.io/deepvariant-docker/deepvariant:0.8.0
```

#### _Convert docker image to singularity on your own VM ( or download singularity image and test data below and change the file paths appropriately )_ ####

```
docker tag gcr.io/deepvariant-docker/deepvariant:0.8.0 localhost:5000/deepvariant:0.8.0
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker push localhost:5000/deepvariant:0.8.0
SINGULARITY_NOHTTPS=1 singularity build deepvariant_cpu.simg docker://localhost:5000/deepvariant:0.8.0

#or 
# wget https://object.pouta.csc.fi/pilot_projects/Deepvariant_singulairty.zip

```
> note: if there are issues due to mismatching between Sha256 digests while building singularity image, one may use `docker system prune -a` command to fix it.

Once you have singularity image, you can copy singularity image to a directory under your username on scratch drive (e.g., /scratch/project_xxxx/$USER/Deepvariant_singularity) using the following command:

```
scp deepvariant_cpu.simg username@puhti.csc.fi:/scratch/project_xxxx/$USER/Deepvariant_singularity
```
You can now login to puhti and  copy testdata for deepvariant analysis as below:

```
cp -fr /scratch/project_xxxx/Deepvariant_singularity/testdata  /scratch/project_xxxx/$USER/Deepvariant_singularity

```

### Prepare slurm scripts to run on Puhti (e.g., deepvariant_puhti.sh)

```
#!/bin/bash
#SBATCH --time=00:05:00
#SBATCH --partition=test
#SBATCH --account=project_xxxx

mkdir -p tmp
export TMPDIR=$PWD/tmp

singularity -s exec -B $PWD:/scratch/project_xxxx/$USER/Deepvariant_singularity \
deepvariant_cpu.simg \
/opt/deepvariant/bin/run_deepvariant \
--model_type=WGS   --ref=/scratch/project_xxxx/$USER/Deepvariant_singularity/testdata/ucsc.hg19.chr20.unittest.fasta \
--reads=/scratch/project_xxxx/$USER/Deepvariant_singularity/testdata/NA12878_S1.chr20.10_10p1mb.bam \
--regions "chr20:10,000,000-10,010,000"   --output_vcf=output.vcf.gz \
--output_gvcf=output.g.vcf.gz
```

### Submit the job using sbatch command

```
sbatch  deepvariant_puhti.sh
```

Please **note** that one can use gpu version of deepvariant with the following sbatch script

```
#!/bin/bash
#SBATCH --time=00:05:00
#SBATCH --partition=gputest
#SBATCH --gres=gpu:v100:1
#SBATCH --account=project_2003682

mkdir -p tmp
export TMPDIR=$PWD/tmp

singularity -s exec --nv  -B $PWD:/scratch/project_2003682/$USER/Deepvariant_singularity \
deepvariant_gpu.simg \
/opt/deepvariant/bin/run_deepvariant \
--model_type=WGS   --ref=/scratch/project_2003682/$USER/Deepvariant_singularity/testdata/ucsc.hg19.chr20.unittest.fasta \
--reads=/scratch/project_2003682/$USER/Deepvariant_singularity/testdata/NA12878_S1.chr20.10_10p1mb.bam \
--regions "chr20:10,000,000-10,010,000"   --output_vcf=outputi_gpu.vcf.gz \
--output_gvcf=output_gpu.g.vcf.gz

```
