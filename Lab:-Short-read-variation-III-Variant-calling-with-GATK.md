# Calling Variants on Illumina Data with GATK

## Adding Read Groups

We need to add read group (RG) information to perform our downstream analysis. We're going to add it to our alignment files now, but RGs can also be added at the alignment stage (if you're curious, see the [BWA manual](https://bio-bwa.sourceforge.net/bwa.shtml) and check out the `-R` option for `bwa mem`).

Read group data is metadata that gives more context to the alignments for downstream tools so they can do things like distinguish samples and detect errors.

We're going to use `AddOrReplaceReadGroups` via GATK, but it's a program that's part of [Picard Tools](https://broadinstitute.github.io/picard/), which is bundled with GATK.

Navigate to your `3_bwa` folder where you have your sorted BAM data.

``` bash
/pickett_shared/software/gatk-4.2.6.1/gatk \
	AddOrReplaceReadGroups \
	--INPUT <your_read_set>_sorted.bam \
	--OUTPUT <your_read_set>_sorted.RG.bam \
	--RGSM <your_read_set> \
	--RGLB <your_read_set> \
	--RGPL ILLUMINA \
	--RGPU <your_read_set>
```

> [!NOTE]
> `--INPUT` is your sorted BAM file
> `--OUTPUT` is the name of your new file that will have RG data
> `--RGSM` is the name of the sample the input belongs to 
> `--RGLB` is the name of the library this sample belongs to.
> `--RGPL` is the name of the platform that was used to create these reads (e.g. Illumina, PacBio, etc.)
> `--RGPU` is (intended to be) the the platform unit that describes exactly which flowcell, lane, and sample multiplex barcode was used when the reads were created.
>
> In our case, we are just setting the `RGPU` field to the name of the sample, which will be fine for our purposes. If our experiment was more complex, we would be using more specific information here.

See [this page](https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups?id=6472) for more information on read groups.

If you want to save space, you can delete your old sorted BAM file because your new `[...].RG.bam` is the file you'll work with from here on.

## Samtools Index

Now that we've got a sorted BAM with read groups, we need to create an index with `samtools index`.

``` bash
spack load samtools@1.15.1

samtools index <your_read_set>_sorted.RG.bam
```

The resulting index file will be named something like `[...]_sorted.RG.bam.bai`.

Other tools will often automatically look for this file in the same directory as the BAM when you run them. Typically, if a tool can't find this file, it will raise an error to tell you.

### Example Samtools Index Loop

> [!NOTE]
> Here's an example of what processing all of your samples with a for loop might look like. This isn't likely to be the best solution if you have a lot of data, but it would get the job done.
> _You don't need to run this, this is just an example_

``` bash
spack load openjdk@11.0.8_10
spack load samtools@1.15.1

for BAM in *_sorted.bam; do
	SAMPLENAME=$(basename "$BAM" | sed 's/_sorted.bam$//')
	/pickett_shared/software/gatk-4.2.6.1/gatk AddOrReplaceReadGroups \
			--INPUT $BAM \
			--OUTPUT ${SAMPLENAME}_sorted.RG.bam \
			--RGSM $SAMPLENAME \
			--RGLB $SAMPLENAME \
			--RGPL ILLUMINA \
			--RGPU $SAMPLENAME

	samtools index ${SAMPLENAME}_sorted.RG.bam
done
```

## GATK

[GATK Homepage](https://gatk.broadinstitute.org)

GATK is a large toolkit of programs for a variety of bioinformatic analyses. We're following the best practices workflow outlined by the Broad Institute [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035535932-Germline-short-variant-discovery-SNPs-Indels).

In addition to GATK, we'll be using some tools from [Picard](https://broadinstitute.github.io/picard/) that are bundled with GATK.

#### Supporting files

> [!Warning]
> _You do not need to run this step, these files are already in the reference folder_

We use [samtools faidx](http://www.htslib.org/doc/samtools-faidx.html) and [Picard CreateSequenceDictionary](https://gatk.broadinstitute.org/hc/en-us/articles/5358872471963-CreateSequenceDictionary-Picard) to make a an index (`.fai`) and sequence dictionary (`.dict`) for our reference.

``` bash
spack load samtools@1.15.1

samtools faidx /pickett_sphinx/teaching/EPP622_2024/raw_data/genome/solenopsis_invicta_genome.fa.gz

/pickett_shared/software/gatk-4.2.6.1/gatk CreateSequenceDictionary -R /pickett_sphinx/teaching/EPP622_2024/raw_data/genome/solenopsis_invicta_genome.fa.gz
```

### Prep
Make a new folder for this step, and create symlinks to the files we're going to use.

``` bash
mkdir 4_gatk
cd 4_gatk

# link the sorted RG BAM and its index
ln -s $(readlink -e ../3_bwa/*_sorted.RG.*) ./

# link some relevant reference genome files
ln -s /pickett_sphinx/teaching/EPP622_2024/raw_data/genome/solenopsis_invicta_genome.fa.gz .
ln -s /pickett_sphinx/teaching/EPP622_2024/raw_data/genome/solenopsis_invicta_genome.fa.gz.gzi .
ln -s /pickett_sphinx/teaching/EPP622_2024/raw_data/genome/solenopsis_invicta_genome.fa.gz.fai .
ln -s /pickett_sphinx/teaching/EPP622_2024/raw_data/genome/solenopsis_invicta_genome.dict .

```

> [!NOTE]
> `readlink` resolves a relative path into a canonical path from root, so you don't have to worry if you relocate your symbolic link file, it will still point to the same file (as long as that other file doesn't move!)
> See `man readlink` for more on how to use it.
### Variant Calling

#### On a Single Sample

Using GATK's [HaplotypeCaller](https://gatk.broadinstitute.org/hc/en-us/articles/5358864757787-HaplotypeCaller), we'll call variants on a single sample. We're also going to call variants on only one chromosome for this example to same some time.

``` bash
/pickett_shared/software/gatk-4.2.6.1/gatk \
	--java-options "-Xmx4G" \
	HaplotypeCaller \
	-R solenopsis_invicta_genome.fa.gz \
	-I <your_read_set>_sorted.RG.bam \
	-O <your_read_set>.vcf \
	-bamout <your_read_set>_sorted.RG.realigned.bam \
	-L NC_052664.1
```

> [!NOTE]
> `-R` is the reference file used to create the alignments.
> `-I` is the input BAM you want to call variants for.
> `-O` is what file you want to write the output to.
> `-bamout` outputs the reads and pseudo-haplotypes that were realigned when calling variants. (We will take a look at this file in a little while)
> `-L` is the interval where you would like to call variants.
> If you do not provide an `-L` argument, `HaplotypeCaller` will call variants on the entire genome.

If you use something like `less` to browse through the resulting VCF (Variant Call Format) file, you'll see a large amount of header information on the lines that start with `##`. That information is relevant to downstream tools, but isn't very interesting to us right now. Let's look at the first few lines after that header.

``` bash
grep -v '^##' <your_read_set>.vcf | head -n 3
```

You should see something like the following:

``` txt
#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  SRR6922447
NC_052664.1     54393   .       T       C       176.97  .       AC=2;AF=1.00;AN=2;DP=6;ExcessHet=0.0000;FS=0.000;MLEAC=2;MLEAF=1.00;MQ=27.00;QD=29.50;SOR=3.912     GT:AD:DP:GQ:PL  1/1:0,6:6:18:191,18,0
NC_052664.1     59958   .       C       CACCG   526.02  .       AC=2;AF=1.00;AN=2;DP=12;ExcessHet=0.0000;FS=0.000;MLEAC=2;MLEAF=1.00;MQ=50.20;QD=25.36;SOR=5.136    GT:AD:DP:GQ:PL  1/1:0,12:12:36:540,36,0
```

Now, this is good information on a couple of loci in _this sample_, but if we want to compare several samples' variants, we would have to match up the sites where they all have variants. Furthermore, we could _only_ compare samples at sites where they _all_ had variants unless we went back to the original data and checked those same sites over again.

We could run all of our samples at the same time and get a single VCF that describes the sites in all of the samples, but we wouldn't be able to add any more samples to that analysis afterwards, and if it were to fail midway (for example if the system crashed, or ran out of memory or storage), then we'd have to wait on an entire new run from scratch.

Especially if your study involves a large number of samples, those issues can be very troublesome. This is where the GVCF (Genomic Variant Call Format) approach comes in.

#### Genomic VCF

Let's generate a GVCF file with `HaplotypeCaller` this time instead.

We'll also use the `-bamout` option to generate a BAM that contains the reads as realigned by `HaplotypeCaller` while it was calculating variants, which we'll take a look at a little later.

``` bash
/pickett_shared/software/gatk-4.2.6.1/gatk \
	--java-options "-Xmx4G" \
	HaplotypeCaller \
	-R solenopsis_invicta_genome.fa.gz \
	-I <your_read_set>_sorted.RG.bam \
	-O <your_read_set>.g.vcf \
	-ERC GVCF
```

> [!Note]
> `-ERC GVCF` (or `--emit-ref-confidence GVCF`) instructs the tool to output the GVCF format.
> This time, we named the output `[...].g.vcf` so we know this is a genomic VCF.
> We did _not_ use `-L` this time, so we called variants for the whole genome.
> `-R` and `-I` are the same as before.

The GVCF (Genomic Variant Call Format) is similar to the VCF, but it reports _every_ site. Try taking a look at the first few entries like we did with the VCF above. In the GVCF, there are also lines that don't describe variants but instead describe stretches of the genome and the quality of the base calls in the BAM. These are used later to compare different samples to each other without having to dive back in to every sample's BAM to check if it was reference type or had missing/low quality data at that site.

The tradeoff with this approach it that we can't use the GVCF directly for analysis. For now, please copy your GVCF file into a shared folder so we can use them all to create a single VCF that contains the variants for all of the samples.

``` bash
cp <your_read_set>.g.vcf /pickett_sphinx/teaching/EPP622_2024/analyses/GVCFs/
```

## Re-Calling SNPs with GVCFs and Multiple Samples

Now that we've got several samples' GVCFs, we can call variants on the group of them.

### CombineGVCFs

First, we need to gather our GVCFs together into one file using [GATK CombineGVCFs](https://gatk.broadinstitute.org/hc/en-us/articles/5358891439771-CombineGVCFs).

First, let's create a new folder for this step and link our GVCF files.

``` bash
mkdir 5_gatk_combine
cd 5_gatk_combine

# symlink our reference genome and required other supporting files
ln -s $(readlink -e ../4_gatk/solenopsis_invicta_genome.fa.gz) .
ln -s $(readlink -e ../4_gatk/solenopsis_invicta_genome.fa.gz.gzi) .
ln -s $(readlink -e ../4_gatk/solenopsis_invicta_genome.fa.gz.fai) .
ln -s $(readlink -e ../4_gatk/solenopsis_invicta_genome.dict) .

# create symlinks to the GVCFs
ln -s /pickett_sphinx/teaching/EPP622_2024/analyses/GVCFs/*.g.vcf .
```

> [!NOTE]
> Here, `readlink` 

Now we need to tell GATK which samples to use when calling SNPs. One way to do that is to supply every GVCF as an argument. We can make a script that can build a script with those arguments automatically with a for loop like this:

``` bash
echo '/pickett_shared/software/gatk-4.2.6.1/gatk CombineGVCFs -R solenopsis_invicta_genome.fa.gz \'

for f in *.g.vcf; do
	echo "--variant $f \\"
done

echo "-O solenopsis_combined.g.vcf.gz"
```

If you create that script and run it, you will get an output like this:

``` txt
/pickett_shared/software/gatk-4.2.6.1/gatk CombineGVCFs -R solenopsis_invicta_genome.fa.gz \
--variant <sample1>.g.vcf \
--variant <sample2>.g.vcf \
--variant <sampleEtc>.g.vcf \
-O solenopsis_combined.g.vcf.gz
```

Where every line with `-variant` lists one of the GVCF files. If you create a script with that and run it, then `CombineGVCFs` will run with all of your samples.

Another way to provide all of the files to GATK is by creating a file that lists all of them, and giving that file to `CombineGVCFs`.

``` bash
# create the sample list
ls *.g.vcf > sample.list

/pickett_shared/software/gatk-4.2.6.1/gatk \
	CombineGVCFs \
	-R solenopsis_invicta_genome.fa.gz \
	--variant sample.list \
	-O solenopsis_combined.g.vcf.gz
```

Choose one of the above methods, and run it to generate your combined GVCF.

### GenotypeGVCFs

Now we'll calculate the genotypes from the combined GVCF file using [GATK GenotypeGVCFs](https://gatk.broadinstitute.org/hc/en-us/articles/5358906861083-GenotypeGVCFs).

``` bash
/pickett_shared/software/gatk-4.2.6.1/gatk \
	--java-options "-Xmx10g" \
	GenotypeGVCFs \
	-R solenopsis_invicta_genome.fa.gz  \
	-V solenopsis_combined.g.vcf.gz \
	-O solenopsis_combined.vcf.gz
```

Let's use `bcftools` to see how many SNPs we got.

``` bash
spack load bcftools

bcftools stats solenopsis_combined.vcf.gz > solenopsis_combined.vcf.stats.txt

less solenopsis_combined.vcf.stats.txt
```

Not bad, however, this VCF probably contains a lot of sites that may not be real, so we need to filter them to select the sites that are more likely to be real to the best of our ability.

For filtering variants, Broad Institute recommends using their variant quality score recalibration (VQSR) technique when possible. However, that would require a large set of high-quality previously known variants, which we don't have. The next best thing is "hard filtering" our data.

### Hard Filtering

We'll use [GATK VariantFiltration](https://gatk.broadinstitute.org/hc/en-us/articles/5358873312539-VariantFiltration) to filter out reads that we think aren't real.

#### Basic filters

First, let's filter with a very basic set of parameters. We'll require that our sites are over 20.0 quality, and have more than 5x depth. We'll compare the results to another set of filters later.

``` bash
/pickett_shared/software/gatk-4.2.6.1/gatk \
	VariantFiltration \
	-R solenopsis_invicta_genome.fa.gz  \
	-V solenopsis_combined.vcf.gz \
	-O solenopsis_combined.Basicfilters.vcf \
	--filter-name "basic_filter" --filter-expression "QUAL > 20.0 && DP > 5"
```

#### Recommended filters

Now, let's use the [suggested filters from Broad Institute](https://gatk.broadinstitute.org/hc/en-us/articles/5358906861083-GenotypeGVCFs). More detail on these is available [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035890471-Hard-filtering-germline-short-variants) as well.

Additionally, they recommend to that we filter the SNPs and INDELs separately, but we're just going to filter our data with the recommended SNP parameters to save some time.

``` bash
/pickett_shared/software/gatk-4.2.6.1/gatk \
	VariantFiltration \
	-R solenopsis_invicta_genome.fa.gz  \
	-V solenopsis_combined.vcf.gz \
	-O solenopsis_combined.GATKfilters.vcf \
	--filter-name "QD_filter" --filter-expression "QD < 2.0" \
	--filter-name "FS_filter" --filter-expression "FS > 60.0" \
	--filter-name "MQ_filter" --filter-expression "MQ < 40.0" \
	--filter-name "SOR_filter" --filter-expression "SOR > 4.0" \
	--filter-name "MQRankSum" --filter-expression "MQRankSum < -12.5" \
	--filter-name "ReadPosRankSum" --filter-expression "ReadPosRankSum < -8.0" 
```


### Comparing the Filtered VCFs

`VariantFiltration` only marks the sites if they don't pass, it doesn't remove them, so we need to provide an extra argument to `bcftools stats` to look at only those sites that passed our filtered.

``` bash
spack load bcftools

bcftools stats -f "PASS,." solenopsis_combined.vcf.gz > solenopsis_combined.vcf.stats.txt

bcftools stats -f "PASS,." solenopsis_combined.Basicfilters.vcf > solenopsis_combined.Basicfilters.vcf.stats.txt

bcftools stats -f "PASS,." solenopsis_combined.GATKfilters.vcf > solenopsis_combined.GATKfilters.vcf.stats.txt

grep 'number of SNPs:' *stats.txt
```

You can see that with the filters from Broad Institute we kept many more SNPs than with our simple filter, and we can be relatively confident that most of them are high enough quality for meaningful further analysis.

### Selecting Sites

While some tools automatically detect and skip sites that have anything other than `.` or `PASS` in the filter column of the VCF, not every tool does. We can use GATK's [SelectVariants](https://gatk.broadinstitute.org/hc/en-us/articles/5358856605339-SelectVariants) to extract a subset of data from our VCF if we want to focus on a particular kind of variant, subset of samples, region, or passing sites.

This is an example of how to create a new VCF with only biallelic SNP sites that passed all filters.

``` bash
/pickett_shared/software/gatk-4.2.6.1/gatk \
	SelectVariants \
	-R solenopsis_invicta_genome.fa.gz \
	-V solenopsis_combined.vcf.gz \
	-O s_inv_only_passing_snps.vcf.gz \
	--select-type-to-include SNP \
	--restrict-alleles-to BIALLELIC \
	--exclude-filtered
```
