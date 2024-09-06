### BWA (Burrows-Wheeler Alignment)
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

What % of reads mapped from your assigned fastq file? Is it different from the one you got using samtools flagstat? 

To rectify this, we can use the `bc` calculator tool to calculate the total number of reads from our assigned fastq file, and use this as the denominator to calculate the correct % for mapped reads.

```bash
spack load bc%gcc@8.4.1
echo $(cat <your_assigned_read_set>-trimmed.fastq | wc -l)/4 | bc
```
% reads mapped = (total # of reads mapped from flagstat)/(total number of reads from bc command) * 100

### For your reference - Using loops to run `bwa` (or any program) on multiple files.

In this lab, you are running each program on a single file. In reality, you will have dozens of files, and typing out the command for each file is tedious and cumbersome. Instead, we can use a for loop to run a program through all our files.

To exemplify this, I created a directory `for_loop_exercise` with 2 sub-directories - `single_end` and `paired_end`. We will see how to run for loop on both type of data.
```bash
(base) [bkapoor@sphinx bkapoor]$ tree for_loop_exercise/
for_loop_exercise/
├── paired_end
│   ├── SRR6922148_1.fastq
│   ├── SRR6922148_2.fastq
│   ├── SRR6922291_1.fastq
│   ├── SRR6922291_2.fastq
│   ├── SRR6922311_1.fastq
│   └── SRR6922311_2.fastq
└── single_end
    ├── SRR6922148.fastq
    ├── SRR6922291.fastq
    └── SRR6922311.fastq

2 directories, 9 files
```
Change into your personal working directory and copy this new directory into it. Make sure to use the recursive flag (-r) to copy the directory and all its contents.

```bash
cd /pickett_sphinx/teaching/EPP622_2024/analyses/<your_utk_username>
cp -r /pickett_sphinx/teaching/EPP622_2024/bkapoor/for_loop_exercise .
```
First, let's run for loop on single end files. Change into the `single_end` folder and start writing the script using `nano`. Name it whatever you want, but make sure it's relevant to your task and it ends in `.sh`. Here is what I named mine:

```bash
cd single_end/
nano bwa_se.sh
```

The first thing we need to do is extract the base name from each of the files. We can do this using `sed`; also known as "stream editor", `sed` is incredibly powerful, as it can be used to replace, search, insert, or delete items in a file.  

**NOTE**: although these scripts use `bwa`, you can adapt these scripts to any program you want to run.

```bash
for file in *.fastq
do
    basename=$(echo "$file" | sed 's/.fastq//')
    echo $file
    echo $basename
done
```

The `sed 's/.fastq//'` part of the command substitutes (the "s") the ".fastq" part of the filename with nothing (notice the emptiness between the slashes). Execute the script by typing `bash bwa.sh`. It will print the original name of the file (`echo $file`), followed by the new-and-improved basename (`$echo basename`).

```bash
SRR6922148_1.fastq
SRR6922148_1
SRR6922291_1.fastq
SRR6922291_1
SRR6922311_1.fastq
SRR6922311_1
```

Now, let's modify and combine this new code with our old `bwa` code. Also, you can add your software packages here instead of loading before running the script. In comparison to our original `bwa` run in the lab session, note that the only thing that changes is the first part of the input and output files will be now be ${basename} instead of the file's name hardcoded in. 

```bash
spack load bwa
spack load samtools@1.9%gcc@8.4.1

for file in *.fastq
do
    basename=$(echo "$file" | sed 's/.fastq//')
    echo $file
    echo $basename

    bwa mem -t 6 \
    solenopsis_genome_index/solenopsis_invicta_genome.fa.gz \
    ${basename}.fastq \
    | samtools view -bSh \
    | samtools sort \
    -@ 3 -m 4G \
    -o ${basename}_sorted.bam
done
```

Next, let's use for loop to map paired end data to the genome. Change into the `paired_end` directory and write the following script.

```bash
cd ../paired_end
nano bwa_pe.sh
```

Let's see an example:

```bash
spack load bwa
spack load samtools@1.9%gcc@8.4.1

for file in *_1.fastq
do
    basename=$(echo "$file" | sed 's/_1.fastq//') 
    echo $basename #you will get SRR### instead of SRR###_1 like in the previous example

    bwa mem -t 6 \
    solenopsis_genome_index/solenopsis_invicta_genome.fa.gz \
    ${basename}_1.fastq \ #hardcode in "_1" 
    ${basename}_2.fastq \ #hardcode in "_2" 
    | samtools view -bSh \
    | samtools sort \
    -@ 3 -m 4G \
    -o ${basename}_sorted.bam

done
```