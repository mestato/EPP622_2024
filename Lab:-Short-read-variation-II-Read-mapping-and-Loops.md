# BWA (Burrows-Wheeler Alignment)
In today's lab, we will be mapping our reads to the _Solenopsis invicta_ reference genome using [bwa](http://bio-bwa.sourceforge.net/). Before we get started, let's prep our directories and retrieve the reference genome index files.

First, change into your personal directory and make a new directory titled `3_bwa`. As a reminder, the path to your personal working directory should be 
```
/pickett_sphinx/teaching/EPP622_2024/analyses/<your_username>
```

Then, create a new directory called `3_bwa` and change into it.

```bash
mkdir 3_bwa
cd 3_bwa
```

The next step of this process is to download the reference genome and index it. In the interest of time, we already downloaded and indexed the genome for you. The necessary files can be found here: 
```
/pickett_sphinx/teaching/EPP622_2024/raw_data/genome
```

However, if you need to index a reference genome for your project, the command is:

```bash
#This code is just for your reference, please DO NOT run this!
bwa index <your_organism_ref_genome.fasta>
```

Let's symbolically link the _S. invicta_ genome and index files to our working directories for easier access. First, create a new directory within your `3_bwa` directory (called `solenopsis_genome_index`).

```bash
mkdir solenopsis_genome_index
cd solenopsis_genome_index
ln -s ../../../../raw_data/genome/solenopsis_invicta_genome.fa.gz* .
```
Next, change back into your `3_bwa` directory (`cd ../`) and symbolically link your trimmed fastq file.

```bash
cd ../
ln -s ../2_skewer/<your_assigned_read_set>-trimmed.fastq .
```

Now, we are almost ready! Load `bwa` and `samtools` using `spack`. The output of your `bwa` run is a SAM file and we will need to convert it to a BAM file. Using the fancy `|` notation we learned in the Software Carpentry lessons, we can combine these commands to save time.

```bash
spack load bwa
spack load samtools@1.9%gcc@8.4.1

bwa mem -t 3 \
solenopsis_genome_index/solenopsis_invicta_genome.fa.gz \
<your_assigned_read_set>-trimmed.fastq \
| samtools view -bSh \
| samtools sort \
-@ 3 -m 4G \
-o <your_assigned_read_set>_sorted.bam 
```
* `-t` is used to specify the number of threads to run the job (bwa) 
* `samtools view -bSh` is used to convert the output from a SAM file to a BAM file with a header 
* `-@` and `-m` designates the number of threads to run the job and memory per thread (samtools)

**NOTE** - It takes about 5 minutes to run this command. It may take longer for you depending upon the number of reads in your assigned fastq file.

Once the command stops running, you can extract stats from your sorted.bam file using `samtools flagstat` and output to a separate file.

```bash
samtools flagstat <your_assigned_read_set>_sorted.bam > <your_assigned_read_set>_sorted.stats
```

Open up your `sorted.stats` file using `cat` to view various statistics. You may notice the % of reads mapped, and this value may be very high (>99% reads mapped). However, the percentage is wrong because `bwa` only counts reads that map to the reference as input reads. 

To rectify this, we can use the `bc` calculator tool to calculate the total number of reads from our assigned fastq file, and use this as the denominator to calculate the correct % for mapped reads.