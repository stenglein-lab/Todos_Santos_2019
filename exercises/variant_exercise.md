## Intrahost variation exercise

## In this exercise, we will go through a variant calling pipeline. 

The data we'll use was generated from a pool of _Anopheles gambiae_ mosquitoes collected in Liberia.  The dataset contains reads mapping to a virus (Anopheles flavivirus).  We'll map the reads to the viral genome and use a variant caller to identify variants and determine their frequencies.

We will:

- download the files we need
- make a bowtie index from the viral genome (reference) sequence
- map reads to the reference sequence using bowtie2
- use lofreq to call variants
- visualize and inspect variants in vcf output file and in Tablet


### Download files needed for exercise from github

First, we need to download some files.

Open the terminal and change directory to a directory of your choosing or stay where you are.

The files for this exercise are in the TodosSantos directory in your home directory.  Transfer them to a new directory

```
cd
mkdir variant_exercise
cd variant_exercise
cp ../TodosSantos/variant_exercise_files.tar.gz .
```

The file extension `.tar.gz` is similar to .zip.  It means this is a compressed set of files.  

You'll often run across `.tar.gz` files when you're downloading data or software.  You can uncompress and extract this file using the tar command, as follows:

```
tar xvzf variant_exercise_files.tar.gz
```

You should now see 4 new file (run the `ls` command):
```
Pool_filtered_R1.fastq   # paired read files containing already trimmed reads in FASTQ format
Pool_filtered_R2.fastq
viral_genome.fasta       # viral genome in FASTA format
viral_genome.gb          # viral genome in GenBank (annotated) format
```

### Create a bowtie index from the viral genome sequence

We're going to use bowtie2 to map reads in the dataset to the viral genome.  First, we need to create a bowtie2 index.  Remember how to do that?

```
bowtie2-build viral_genome.fasta viral_genome_bt_index 
```


### Mapping reads in the dataset to the viral genome sequence 

Now that we've created the index, we can map reads to it.  We'll use [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) to do this, as follows

```
bowtie2 -x viral_genome_bt_index \
   -q -1 Pool_filtered_R1.fastq  -2 Pool_filtered_R2.fastq \
   --no-unal --threads 4 -S Pool_reads_aligned_to_viral_genome.sam
```

### Using lofreq to call variants

Now we have mapped reads.  There are a variety of variant callers (see review articles cited in lecture).  We'll use the [lofreq](http://csb5.github.io/lofreq/) variant caller, because it is pretty easy to use and has performed reasonably well in benchmarking comparisons.

Most variant callers take as input sorted .bam format files, which are a compressed, sorted version of sam files.  We'll use the samtools program to convert our .sam output file from bowtie to .bam format:

```
# convert SAM -> BAM
samtools view -b Pool_reads_aligned_to_viral_genome.sam > Pool_reads_aligned_to_viral_genome.bam

# sort BAM 
samtools sort -T tmp -O 'bam' Pool_reads_aligned_to_viral_genome.bam  > Pool_reads_aligned_to_viral_genome.sorted.bam
```

Now, we'll run lofreq.  To learn more about how you could run lofreq, run:
```
lofreq         # show general usage info
lofreq call    # show general usage info for the call command in lofreq (actually does variant calling)
```

To call variants, you tell lofreq the name of the reference sequence (-f option), the name of the sorted bam file, and the name of the output file that will be created (-o option).  The output is in [VCF format](https://samtools.github.io/hts-specs/VCFv4.3.pdf):
```
lofreq call -f viral_genome.fasta Pool_reads_aligned_to_viral_genome.sorted.bam -o Pool_reads_aligned_to_viral_genome.vcf
```

Use less or cat to inspect the contents of your vcf file.  
```
less Pool_reads_aligned_to_viral_genome.vcf
```
Questions to consider:
- Does the VCF file make sense?  
- How many variants were identified?
- What are their allele frequencies?
- Is linkage between variants described?
- Are the variants SNPs, or InDels?  Would you expect to see InDel variants here?

### Time permitting: Visualize and inspect mapped data supporting called variants in Tablet

First, you need to get your reference sequence into Tablet.  Then you need to bring in the mapped reads (can bring in .sam or .bam format)

1. Transfer Pool_reads_aligned_to_viral_genome.sam and viral_genome.fasta to your laptop using sftp or Cyberduck
2. Open Tablet and click Open Assembly.  Open these files.

**Some questions to consider:**
- Can you identify variants called in your VCF file?  (*Note:* numbering differs between the consensus sequence and the reference sequence in Tablet.  #s in the VCF file correspond to reference sequence positions).
- Do variant frequencies seem to match between Tablet and the VCF file?
- Can you identify linked variants?  How far apart can you identify linked variants?
- Are these intrahost or interhost variants? (Hint: these reads come from a _pool_ of mosquitoes


