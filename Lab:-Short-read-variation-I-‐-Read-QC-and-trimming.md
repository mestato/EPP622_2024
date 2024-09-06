### 1. Finding and assigning fastq data files

The Solenopsis invicta RAD data we are using for the labs comes from this [project](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA448217). We have already downloaded all the reads, which can be found here -
```bash
/pickett_sphinx/teaching/EPP622_2024/raw_data/solenopsis_invicta
```

Please view the below table, as every student has been assigned a read for the duration of the short read variation I unit.

|Student|Read Accession|Total # Reads|Location
|-------|--------------|------------|----------|
|Sanskriti Acharya|SRR6922148|1281077|Oglethorpe Co, GA
|Aditi|SRR6922294|1405770|Oglethorpe Co, GA
|Maria Caballero Aragon|SRR6922306|1541579|Oglethorpe Co, GA
|Charles Dawe|SRR6922308|2987905|Oglethorpe Co, GA
|Mengling He|SRR6922451|1731620|Oglethorpe Co, GA
|Rebecca Kraus|SRR6922454|2897067|Oglethorpe Co, GA
|Stefanie Menezes De Moura|SRR6922449 |3649330 | Oglethorpe Co, GA
|Marissa Nufer|SRR6922311 |258861 |Oglethorpe Co, GA
|Alina Pokhrel|SRR6922354|1344896|Pascagoula, MS
|Andrew Reed|SRR6922399|1091567|Pascagoula, MS
|Patrick Sisler|SRR6922194|1100979|Alejandra, Argentina
|Hannah Teddleton|SRR6922233|1216598|Alejandra, Argentina
|James Ulmer|SRR6922241|1148981|Alejandra, Argentina
|Erin Van Berkel|SRR6922315|1017635|Alejandra, Argentina
|Makhali Voss|SRR6922318|1592618|Alejandra, Argentina
|Katie Wood|SRR6922319|2199106|Alejandra, Argentina
|Meg Staton|SRR6922321|1696637|Alejandra, Argentina
|Alysson Dekovich|SRR6922446|991054|Alejandra, Argentina
|Beant Kapoor|SRR6922447|1957684|Alejandra, Argentina


## 2. Setting up a personal directory
Go to the analysis directory within the EPP 622 course directory:
```bash
/pickett_shared/teaching/EPP622_2024/analysis
```
...and make a personal analysis folder. For example:

```bash
mkdir <your user id goes here>
cd <your user id goes here>
```



## 3. Running fastqc
Now, let's make a directory where we will run`fastqc:
```
mkdir 1_fastqc
cd 1_fastqc
```

We can create a soft link (symbolic link) to the raw data
```
ln -s ../../../raw_data/solenopsis_invicta/<your subset>.fastq .
```

Let's load fastqc:
```
spack load fastqc
```

Let's run the program now. Since, we all are sharing the same computing resource, we will run `fastqc` on just one forward read fastq file -
```
fastqc <your subset>
```

This program outputs results in`.zip and .html formats. We can't inspect them on sphinx, so we'll need to copy them to our own devices.
```
scp '<your_username>@sphinx.ag.utk.edu:/pickett_shared/teaching/EPP622_Fall2022/analysis/<your_username>/1_fastqc/*html' ./
```