---
layout: post
title: Starting tag-seq
subtitle:
tags: PSMFC-Mytilus-byssus-pilot
comments: true
---

I ran some code from Matt to start the tag-seq alignment process (pretty sure we're aligning to a new transcriptome that members of the Roberts lab assembled). Here's all the QC and trimming code. For some reason when I try to run commands with Matt's data it says permission denied, so I've just been redoing the qc stuff in my own directory and it's worked well, but if you know how to deal with the permission issue let me know, because he's done all the QC already and has it stored in his Raven folder.

```{bash, engine.opts='-l'}
echo $PATH
```

# Download tag-seq data
I appear to have the raw data in .fastq files in my directory (in addition to the fastq.gz files), so I think this step worked and the next worked.
```{bash}
mkdir raw-data/
cd raw-data/
Â
wget -r \
--no-directories --no-parent \
-P . \
-A .fastq.gz https://gannet.fish.washington.edu/panopea/PSMFC-mytilus-byssus-pilot/20220405-tagseq/ \
--no-check-certificate

```

# unzip .fastq.gz files

```{bash}
cd raw-data/
gunzip *.fastq.gz

```

# Run fastqc on untrimmed files
```{bash}
mkdir fastqc/
mkdir fastqc/untrimmed/

/home/shared/FastQC/fastqc \
/home/shared/8TB_HDD_02/mattgeorgephd/PSMFC-mytilus-byssus-pilot/raw-data/*.fastq \
--outdir /home/shared/8TB_HDD_02/mattgeorgephd/PSMFC-mytilus-byssus-pilot/fastqc/untrimmed/ \
--quiet

```

# Run multiqc
```{bash}

eval "$(/opt/anaconda/anaconda3/bin/conda shell.bash hook)"
conda activate

cd fastqc/untrimmed/

multiqc .

```

# trim adapter sequences
Here's where I'm getting stuck, I need to install cutadapt but I can't as I am not "the root." If I could run this code in Matt's folder then it wouldn't be an issue, because he has it installed (I also wouldn't even need to do this because he's done the QC already).
```{bash}
mkdir trim-fastq/
cd raw-data

for F in *.fastq
do
#strip .fastq and directory structure from each file, then
# add suffix .trim to create output name for each file
results_file="$(basename -a $F)"

# run cutadapt on each file
/home/shared/8TB_HDD_02/mattgeorgephd/.local/bin/cutadapt $F -a A{8} -a G{8} -a AGATCGG -u 15 -m 20 -o \
/home/shared/8TB_HDD_02/mattgeorgephd/PSMFC-mytilus-byssus-pilot/trim-fastq/$results_file
done

```

```{bash}

# Before trimming
! wc -l /home/shared/8TB_HDD_02/mattgeorgephd/PSMFC-mytilus-byssus-pilot/merged-fastq/*.fastq

# After trimming
! wc -l /home/shared/8TB_HDD_02/mattgeorgephd/PSMFC-mytilus-byssus-pilot/trim-fastq/*.fastq

```

This is from when Matt put it all together.
```{r}
# Reads remaining after trimming and filtering (%)
1472899016/1582147952*100
```


# Run fastqc on trimmed files
Haven't been able to do this yet b/c of the cutadapt/permissions issue.

```{bash}
mkdir fastqc/
mkdir fastqc/trimmed/

/home/shared/FastQC/fastqc \
/home/shared/8TB_HDD_02/mattgeorgephd/PSMFC-mytilus-byssus-pilot/merged-fastq/*.fastq \
--outdir /home/shared/8TB_HDD_02/mattgeorgephd/PSMFC-mytilus-byssus-pilot/fastqc/trimmed/ \
--quiet

```

# Run multiqc on trimmed files
Haven't been able to do this yet b/c of the cutadapt/permissions issue.
```{bash}

eval "$(/opt/anaconda/anaconda3/bin/conda shell.bash hook)"
conda activate

cd fastqc/trimmed/

multiqc .

```

# Interleave sequences
Haven't been able to do this yet b/c of the cutadapt/permissions issue.
```{bash}

cd trim-fastq/

for filename in *_R1_*.fastq
do
# first, make the base by removing .extract.fastq.gz
  base=$(basename $filename .fastq)
  echo $base

# now, construct the R2 filename by replacing R1 with R2
baseR2=${base/_R1/_R2}
echo $baseR2

# construct the output filename
  output=${base/_R1/}.pe.fastq

  (interleave-reads.py ${base}.fastq ${baseR2}.fastq | \
  gzip > $output)
done

```


# concatenate fastq files by lane
Haven't been able to do this yet b/c of the cutadapt/permissions issue. 
```{bash}
mkdir merged-fastq
cd trim-fastq/

printf '%s\n' *.fastq | sed 's/^\([^_]*_[^_]*\).*/\1/' | uniq |
while read prefix; do
    cat "$prefix"*R1*.fastq >"${prefix}_R1.fastq"
    # cat "$prefix"*R2*.fastq >"${prefix}_R2.fastq" # include if more than one run
done

# I moved files to merged-fastq
