# Linux text manipulation tutorial
This repo contains a short tutorial about text manipulation using the Linux command line.  
We will work with a data set of Zebrafish (_Danio rerio_) CRISPR/CAS9 annealing sites, accessible [here](https://research.nhgri.nih.gov/manuscripts/Burgess/zebrafish/download.shtml).
## Getting started
First, create a directory for the tutorail:
```
cd <wherever you like>
mkdir Linux_text_manipulation_tutorial
cd Linux_text_manipulation_tutorial
```
To download and unzip the data for the tutorial, run the following commads in your terminal:
```
wget https://research.nhgri.nih.gov/manuscripts/Burgess/zebrafish/downloads/NHGRI-1/danRer11/danRer11Tracks/danRer11.CRISPR.bed.gz
pigz -d danRer11.CRISPR.bed.gz
wget http://ftp.ensembl.org/pub/release-104/fasta/danio_rerio/dna/Danio_rerio.GRCz11.dna.toplevel.fa.gz
pigz -d Danio_rerio.GRCz11.dna.toplevel.fa.gz
```
This can take a few minutes.  
**Alternatively:**

## A first look at the data
Let's start by taking a quick look at the files we have. One way to do it is using the `less` command, e.g.: `less danRer11.CRISPR.bed` and `less Danio_rerio.GRCz11.dna.toplevel.fa`. Use up/down arrows to browse through the files. `less` is a very good pager for large files, since it doesn't try to load the entire file to RAM, so even huge files will not choke the machine.  
Two other useful commands are the `head`, which shows the first lines of the file, and `tail`, that shows the last ones:
```
$ head danRer11.CRISPR.bed
track name=crispr2246550 type=bed description="danRer11.spCas9.NGG.crisprs_run1" useScore=1
chr1    1107    1127    GTAGATGAGAGGTCACCGGC_553        513.21  -
chr1    1111    1131    AGCTGTAGATGAGAGGTCAC_350        706.302 -
chr1    1234    1254    GGAAAAATAACCTCCAAACC_436        618.853 -
chr1    4188    4208    GTAATAATCTCAGTTTATCG_292        769.012 +
chr1    4316    4336    GTAGTTGCCAGAATCACTAA_1138       206.765 +
chr1    5007    5027    AGATAAGTTACTTATAATCG_229        838.946 -
chr1    5962    5982    CTATCGAGAGGCATTACTGA_1132       208.532 -
chr1    5965    5985    GTAATGCCTCTCGATAGCTG_166        907.262 +
chr1    5974    5994    CGCACACCGCAGCTATCGAG_134        938.906 -
```
As you can see, this is a [BED](https://m.ensembl.org/info/website/upload/bed.html) file
