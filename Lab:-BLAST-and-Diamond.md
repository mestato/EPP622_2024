## Set Up

If you haven't already, log into sphinx. Stay in your home directory.
```
ssh yourusername@sphinx.ag.utk.edu
```

## Finding software on Spack

Many software packages are available through [the spack system](https://spack.io/). Spack manages scientific software (usually pretty well but not always!) This is a way for a computer system to manage many software packages and make them available to users. For you to use spack, we need to add some information to a file in your home directory called `.bash_profile`. This is a hidden file that you normally don't have to worry about too much. It contains commands that run every time you log in, and it's usually for configurations.

Lets see what cool software we can play with! This is everything spack *can* install:

```
spack list
```

okay, but what is *actually* installed on sphinx?
```
spack find
```

We know we want blast, so we can ask if it is available directly

```
spack find blast
```

Okay, so it seems like it isn't installed, but that is misleading. We've chosen a difficult package to work with for your first package, but only because it is common and there are many versions. Let's do some digging.


First, lets search for "spack blast". Yes, internet searches are your best friend when it comes to Linux.

Okay now lets look for blastplus:
```
spack find blast-plus
```

This works, so why didn't the first search for blast find anything? The 'spack list' command used above returns many results, maybe we can somehow search the output of the list results? This is a great example of how passing (piping) the output of one command to another command can be fast and useful.

```
spack list | grep "blast"
```

---
> Grep?
> Grep is short hand for "globally search a regular expression and print". So what exactly is a regular expression? Well, it's a formal language to define search patterns. Regular expressions (also known as regex) are extremely powerful, and if you are going to be doing a lot of file manipulation and data science, worth learning more about!
---


Further, we can also see the installation information in spack using 'info'
```
spack info blast-plus
```

Finally, we actually load the existing software package:
```
spack load blast-plus
```

Is it working? 

```
blastn -version
blastp -help
```

## BLAST

BLAST is one of the most common bioinformatics tools out there, and many biology students learn to run BLAST for individual or small sets of sequences via a web interface: [you can find a quick tutorial of web BLAST here](https://www.tigm.org/updated-blast-tutorial/). The most popular is probably [NCBI BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi). However, you can perform many more BLAST computations on an HPC.  

BLAST has a number of possible programs to run depending on whether you have nucleotide or protein sequences:
* nucleotide query and nucleotide db - blastn
* nucleotide query and nucleotide db - tblastx (includes six frame translation of query and db sequences)
* nucleotide query and protein db - blastx (includes six frame translation of query sequences)
* protein query and nucleotide db - tblastn (includes six frame translation of db sequences)
* protein query and protein db - blastp

### Get Data

Make a directory to hold our blast lesson. 

```
mkdir blast_examples
cd blast_examples
```

Let's start by downloading some data. Grab some cow and human RefSeq proteins from NCBI:
```
wget ftp://ftp.ncbi.nih.gov/refseq/B_taurus/mRNA_Prot/cow.1.protein.faa.gz
wget ftp://ftp.ncbi.nih.gov/refseq/H_sapiens/mRNA_Prot/human.1.protein.faa.gz
```
	
This is only part of the human and cow protein files. check out how big they are:

```
ls -lh
```

### Running BLAST - Protein vs Protein example
Lets go back to our blast directory.  The database files are both gzipped, so lets unzip them
```
cd blast_examples
gunzip *gz
ls
```

Take a look at the head of each file:
```
head cow.1.protein.faa 
head human.1.protein.faa 
```	

Note that the files are in fasta format, even though they end if ".faa" instead of the usual ".fasta". This NCBI's way of denoting that this is a fasta file with amino acids instead of nucleotides.
	
How many sequences are in each one?
```
grep -c '^>' cow.1.protein.faa
grep -c '^>' human.1.protein.faa 
```

This grep command uses the c flag, which reports a count of lines with a match to the pattern. In this case, the pattern is a regular expression, meaning match only lines that begin with a >. In this case, we are asking to search for lines that begin with a greater than sign using the regex convention of the '^'symbol. 

This is a bit too big, let's take a smaller set for practice. Let's take the first two sequences of the cow proteins, which we can see are on the first 6 lines
```
head -n 16 cow.1.protein.faa > cow.small.faa
```	

Now we can blast these two cow sequences against the set of human sequences. First, we need to tell blast about our database. BLAST needs to do some pre-work on the database file prior to searching. This helps to make the software work a lot faster. Use the makeblastdb  command:

### Making a BLAST database
```
makeblastdb -in human.1.protein.faa -dbtype prot
ls
```	

Note that this makes a lot of extra files, with the same name as the database plus new extensions (.pin, .psq, etc). To make blast work, these files, called index files, must be in the same directory as the fasta file.

### Running blastp (protein query to protein database)

Now we can run the blast job. We will use blastp, which is appropriate for protein to protein comparisons.

```
blastp -query cow.small.faa -db human.1.protein.faa -out cow_vs_human_blast_results.txt
```	

Take a look at the results in nano. Note that there can be more than one match between the query and the same subject. These are referred to as high-scoring segment pairs (HSPs).

Lets also take a look at the help pages. Unfortunately, there are no man pages (those are usually reserved for shell commands, but some software authors will provide them as well), but there is a text help output
```
blastp -help
```	

To scroll through slowly

```
blastp -help | less
```	

To quit the less screen, press the q key.

Parameters of interest: `-evalue` ( Default is 10?!?) and `-outfmt`

Lets get only very meaningful matches with a different output format:
```
blastp \
-query cow.small.faa \
-db human.1.protein.faa \
-out cow_vs_human_blast_results.tsv \
-evalue .1 \
-outfmt 7
```

Whoa!? What are those slashes? In a practical sense, these tell the shell "I'm not done with this command yet". In a literal sense, the forward slash is an "escape" character, and it's telling the shell to interpret the newline as just a newline, not a "submit" of the command.

Check out the results with nano.

You MAY get an error message such as `Segmentation fault` when running nano. If so, load it from spack:
```
spack load nano
```

Tab-delimited has these default columns (-outfmt 7):
```
qseqid 		Query sequence ID
sseqid		Subject (ie DB) sequence ID
pident		Percent Identity across the alignment
length 		Alignment length
mismatch 	# of mismatches
gapopen 	Number of gap openings
qstart 		Start of alignment in query
qend 		End of alignment in query 
sstart 		Start of alignment in subject
send		End of alignment in subject
evalue 		E-value
bitscore	Bit score
```

I find it very useful to add the subject sequence description to the tabular output. If you go through help some more, you will find that you can decide which types of output you would like in the tab-delimited file and what order they should be in. For example:

```
blastp \
-query cow.small.faa \
-db human.1.protein.faa \
-out cow_vs_human_blast_results.tsv \
-evalue .1 \
-outfmt "7 std stitle" 
```

Lets try a medium sized data set next
```	
head -n 500 cow.1.protein.faa > cow.medium.faa
```	
 
What size is this db?
```
grep -c '^>' cow.medium.faa 
```	

Lets run the blast again, but this time lets not have the comment lines, and lets return only the best hit for each query. 
```
blastp \
-query cow.medium.faa \
-db human.1.protein.faa \
-out cow_vs_human_blast_results.tsv \
-evalue .1 \
-outfmt "6 std stitle" \
-max_target_seqs 1
```

I picked "max target seqs" for a reason! It's quite a misleading flag - it finds the number of target sequences but they may not be the best matches. [See this blog post for more details.](https://blastedbio.blogspot.com/2015/12/blast-max-target-sequences-bug.html).

How many results did we get?
```
wc -l cow_vs_human_blast_results.tsv
```

This is still very small compared to what we can do. Lets imagine needing to BLAST 50,000 sequences against a database of 500,000 proteins (a surprisingly common task!). A single command can do that.

## Diamond

An alternative basic alignment tool to BLAST is diamond. Currently, diamond only works for proteins and translated nucleic acid alignments. See the [diamond documentation](https://github.com/bbuchfink/diamond/wiki). Diamond runs *much* faster than BLAST, but hasn't been around as long.

Load up diamond using spack:
```
spack load diamond
```

If spack does not work, use the locally installed version
```
```

Similarly to BLAST, diamond also creates a database:
```
diamond makedb --in human.1.protein.faa --db human_db
```

Then, diamond runs... blastp. Note: here we are using the block flag 'b1' to denote amount of memory for diamond to use. According to diamond's documentation, a block size of 1 (b1) corresponds with 1 billion sequence letters to process at a time, and the ram usage in Gb is approximately 6X this value (e.g. using b2 would use 12Gb, and b3 would use 18gb).
```
diamond blastp -d human_db.dmnd -q cow.medium.faa -b1 -o cow_vs_human_diamond_results.tsv
```

Always read the manual and consider memory usage! Also, whenever using a newer software on GitHub, be sure to swing by the [issues](https://github.com/bbuchfink/diamond/issues) page to see if there are any concerns by fellow users than might influence your analysis.