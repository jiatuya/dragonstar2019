### Preparation

1. To use installed softwares, you need to run 
```
. "/shared/miniconda3/etc/profile.d/conda.sh"
conda activate base
```

You will see that your command line prompt is now changed to something that look like `(base) [biouser@test-0010 ]$ `. This indicates that you are already in the base environment in conda, and we can proceed with the exercise.

2. Prepare folders

2.1 Go to your home folder (you can type `cd` to automatically go to your home folder, and `mkdir -p ~/project/startup/` folder, and then `cd ~/project/startup/`. All our tests will be done in this directory.

2.2 `ln -s /shared/data/practice_data data` to link the fq, bam and vcf files for practice.

### Practice basic commands for fq and fa files

Examine the first two reads in a fastq file.
```
head -n 8 data/sr.chr1.2mb_1.fq
```

The output is below:

```
@HISEQ1:9:H8962ADXX:1:1101:1283:41924
TTATAGTTTTTAGTGTACAGGTGCTATTCTTCTTTTGTTAATCTTGTTCCCAAGAATTTTTTTTTAATTTACTGCTATTGTAATTGTTGTAATTGGAATTGGATTTTTTATTTTTATTTTTTTATTTTTATTCTATTATTATTATTAT
+
CCCFFFDEHHHHHEGGHGFHJEFHHIJJIJJJJJJJJJIIGGIJIHHCIIDDB?DGFIHJEFHIFFEFFFCCEA>CCCDC@C;@C>CCB?@@CDDDDACCCCCCDDDDD?CDEED<CDEEDDB8ACEEC8AC:B:>@4@C:@>4@B>C
@HISEQ1:9:H8962ADXX:1:1101:1403:46126
AAGTAGCTGGGATTACAGGTGTATGCCACCACGCCTGGCTAATTTTTGTATTTTTAGTACAGACTGGGTTACGCCAGGTCTTGAACTCCTGGCCTCAAGTGATCCGCCCACTCTGGCCTCCCAAAGTCCTGGGATTACAGGCGTGAGC
+
@??DDBDDHFF;FGHGFH@ICEFEBHFHIIIIBDBH?@;?DHEHBBG6B@FHIIG=FFHBAA7CCEHD@AAA;?<8>A6@>@>5;CCCCCCC?A:5:AA4:+4>A18?9.8?AC@::8:<A??CA?(::@@C(8@C4>>C9?<))++9
```

Count the number of reads in a FASTQ file (divide the line number by 4):

```
[biouser@main-lx startup]$ wc -l data/sr.chr1.2mb_1.fq
```

Convert fq to fa
```
cat data/sr.chr1.2mb_1.fq | paste - - - - | cut -f 1,2 | sed 's/^@/>/' | tr "\t" "\n" > sr.chr1.2mb_1.fa
```

The above command represents an easy and fast way to convert FASTQ file to FASTA file with standard Linux commands. Note that `sed` is used to replace the `@` to `>` character.


### Practice basic commands for bam/sam files

Get basic statistics from a bam
```
samtools stats data/chr1.2mb.mp2.bam | grep ^SN | cut -f 2-
```

The expected results are given below:

```
raw total sequences:    1210338
filtered sequences:     0
sequences:      1210338
is sorted:      1
1st fragments:  605169
last fragments: 605169
reads mapped:   1197138
reads mapped and paired:        1183962 # paired-end technology bit set + both mates mapped
reads unmapped: 13200
reads properly paired:  1106334 # proper-pair bit set
reads paired:   1210338 # paired-end technology bit set
reads duplicated:       0       # PCR or optical duplicate bit set
reads MQ0:      1978    # mapped and MQ=0
reads QC failed:        0
non-primary alignments: 0
total length:   179130024       # ignores clipping
total first fragment length:    89565012        # ignores clipping
total last fragment length:     89565012        # ignores clipping
bases mapped:   177176424       # ignores clipping
bases mapped (cigar):   175426403       # more accurate
bases trimmed:  0
bases duplicated:       0
mismatches:     771150  # from NM fields
error rate:     4.395861e-03    # mismatches / bases mapped (cigar)
average length: 148
average first fragment length:  148
average last fragment length:   148
maximum length: 148
maximum first fragment length:  148
maximum last fragment length:   148
average quality:        34.1
insert size average:    547.5
insert size standard deviation: 158.5
inward oriented pairs:  587439
outward oriented pairs: 2517
pairs with other orientation:   3154
pairs on different chromosomes: 0
percentage of properly paired reads (%):        91.4
```

You can also use `flagstat` to get a quick statistics on the flags in the file:

```
samtools flagstat data/chr1.2mb.mp2.bam
```

The expected output is below:

```
1213010 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
2672 + 0 supplementary
0 + 0 duplicates
1199810 + 0 mapped (98.91% : N/A)
1210338 + 0 paired in sequencing
605169 + 0 read1
605169 + 0 read2
1106334 + 0 properly paired (91.41% : N/A)
1183962 + 0 with itself and mate mapped
13176 + 0 singletons (1.09% : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

Check a few reads which failed to align in a bam
```
samtools view data/chr1.2mb.mp2.bam | awk '{if (and($2,4)) print NR" : "$0}' | less
```

Find FLAG distribution in a bam
```
samtools view data/chr1.2mb.mp2.bam | cut -f 2 | sort | uniq -c
```

Check those alignment whose mapping quality > 30.
```
samtools view data/chr1.2mb.mp2.bam | awk '{if ($5>30) print $0}' | less
```

Check the read alignment distribution according to chromosomes.
```
samtools view data/chr1.2mb.mp2.bam | cut -f 3 | sort | uniq -c
```


### Practice basic commands for bcf/vcf files

Get only snps or indels
```
bcftools view -v snps data/mp2.bcftools.call.bcf | less
bcftools view -v indels data/mp2.bcftools.call.bcf | less
```

Get snps and indels whose coverage > 30
```
bcftools view -i 'MIN(DP)>30' data/mp2.bcftools.call.bcf | less
```

Get snps and indels whose coverage > 30 and quality > 10
```
bcftools view -i 'QUAL>10 & MIN(DP)>30' data/mp2.bcftools.call.bcf | less
```

Get genotype not '1/1'
```
bcftools view -i 'GT!="1/1"' data/mp2.bcftools.call.bcf | less
```

## Advanced practice of processing VCF files

You can use tabix to extract subsets of the vcf files from the 1000genomes websites. Thanks to the fact that tabix uses a index file, you will be able to download only portions of the files, without having to download everything in local.

```
tabix -h ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp/release/20130502/ALL.chr1.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz 1:155259084-155271225 > 1000G_PKLR.vcf
```

This command takes the genomic region "1:155259084-155271225" (in hg19 coordinate) from the 1000 Genomes Project website and save it as the 1000G_PKLR.vcf file.

The input file is hosted in the FTP server contains the 1000 Genomes Project phase3 release of variant calls. This variant set contains 2504 individuals from 26 populations.

Sometimes, due to network problems, the program above does not return a result within a few minutes. In that case, do not worry about it, just stop the program (pressing "Ctrl+C" will stop the program). We already saved a copy of the output file in `/shared/data/VCF/1000G_PKLR.vcf`, and we can just use this file in the following steps.

Next, we want to extract a few samples from the VCF file.
```
bcftools view -s HG00376,NA11933,NA12282 /shared/data/VCF/1000G_PKLR.vcf > test1.vcf
```

In the output VCF file, the original INFO field from the original VCF file is still present. We want to remove the INFO field:

```
bcftools annotate -x INFO test1.vcf > test2.vcf
```

Now examine the new vcf file, you can see that the INFO field is no longer there.


```
bcftools view --min-ac=1 test2.vcf > test3.vcf
```

Now examine the new vcf file, you can see that the INFO field is updated with the AC and AN.

```
#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  HG00376 NA11933 NA12282
1       155260382       rs61755431      C       T       100     PASS    AC=3;AN=6       GT      1|0     1|0     0|1
1       155260780       rs576493870     AAAT    A       100     PASS    AC=3;AN=6       GT      0|1     0|1     0|1
1       155265177       rs2071053       A       G       100     PASS    AC=1;AN=6       GT      0|0     0|1     0|0
1       155266335       rs8177968       C       T       100     PASS    AC=1;AN=6       GT      0|0     0|1     0|0
1       155268120       rs12067675      T       C       100     PASS    AC=1;AN=6       GT      0|0     0|1     0|0
1       155268425       rs12741350      C       T       100     PASS    AC=1;AN=6       GT      0|0     0|1     0|0
1       155268958       rs11264357      C       T       100     PASS    AC=1;AN=6       GT      0|0     0|1     0|0
1       155269776       rs3020781       A       G       100     PASS    AC=1;AN=6       GT      0|0     0|1     0|0
```

So now we have 8 sites that are polymorphic in the VCF file (in other words, we found eight SNPs that have mutations in at least one of the three subjects.

You can make a statistics on the test3.vcf file:

```
bcftools stats test3.vcf
```

We will see the information below:

```
# SN    [2]id   [3]key  [4]value
SN      0       number of samples:      3
SN      0       number of records:      8
SN      0       number of no-ALTs:      0
SN      0       number of SNPs: 7
SN      0       number of MNPs: 0
SN      0       number of indels:       1
SN      0       number of others:       0
SN      0       number of multiallelic sites:   0
SN      0       number of multiallelic SNP sites:       0
```


### For other practice

To do other tutorial, you might need to run `conda deactivate` to get out of the base environment. You might have errors if you do not deactivate the environment.
