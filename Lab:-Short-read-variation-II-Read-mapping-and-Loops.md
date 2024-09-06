# BWA (Burrows-Wheeler Alignment)
In today's lab, we will be mapping our reads to the _Solenopsis invicta_ reference genome using [bwa](http://bio-bwa.sourceforge.net/). Before we get started, let's prep our directories and retrieve the reference genome index files.

First, change into your personal directory and make a new directory titled `3_bwa`. As a reminder, the path to your personal working directory should be 
```
/pickett_sphinx/teaching/EPP622_2024/analysis/<your_utk_username>`
```

Then, create a new directory called `3_bwa` and change into it.

```bash
mkdir 3_bwa
cd 3_bwa
```

The next step of this process is to download the reference genome and index it. In the interest of time, we already downloaded and indexed the genome for you. The necessary files can be found here: 
```
/pickett_sphinx/teaching/EPP622_2024/raw_data/solenopsis_invicta/genome
```

However, if you need to index a reference genome for your project, the command is:

```bash
#for your reference, do not perform for this lab!
bwa index <your_organism_ref_genome.fasta>
```

Let's symbolically link the _S. invicta_ genome and index files to our working directories for easier access. First, create a new directory within your `3_bwa` directory (called `solenopsis_genome_index`).

```bash
mkdir solenopsis_genome_index
cd solenopsis_genome_index
ln -s ../../../../raw_data/solenopsis_invicta/genome/UNIL_Sinv_3.0.fasta* .
```
Next, change back into your `3_bwa` directory (`cd ../`) and symbolically link your trimmed fastq file.

```bash
cd ../
ln -s ../2_skewer/<your_assigned_read_set>_trimmed.fastq .
```

Now, we are almost ready! Load `bwa` and `samtools` using `spack`. The output of your `bwa` run is a SAM file and we will need to convert it to a BAM file. Using the fancy `|` notation we learned in the Software Carpentry lessons, we can combine these commands to save time.