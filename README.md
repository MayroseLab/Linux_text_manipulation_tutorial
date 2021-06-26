# Linux text manipulation tutorial
This repo contains a short tutorial about text manipulation using the Linux command line. Note that the tutorial does not attempt to be throrough or deep - it's just meant to be a nice introduction and demonstrate what one can do when they master the Linux command line.  
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
As you can see, this is a [BED](https://m.ensembl.org/info/website/upload/bed.html) file, which is basically a tab-separated values (TSV) file with genomic coordinates. The columns are:
* Chromosome
* Start position (1-based)
* End position (Inclusive)
* \<Target sequence\>_\<number of off-targets\>
* Score representing specificity
* Strand (+/-)
## Getting the list of chromosomes
Let's begin by making sure that the BED and FASTA files match in terms of the chromosome names they contain. To do that, we'll extract the list of chromosomes from each file.  
Fo the FASTA, we can do that using:
```
$ grep '>' Danio_rerio.GRCz11.dna.toplevel.fa
```
The `grep` command allows us to fetch lines matching a certain pattern from a file - in this case we look for ">", which indicates record headers in a FASTA file. If we only want the REF chromosomes (names ending with "REF"), we can use a regex pattern:
```
grep '>*REF$' Danio_rerio.GRCz11.dna.toplevel.fa
```
One other trick we can use `grep` for is to get the total genome size:
```
$ grep -v '>' Danio_rerio.GRCz11.dna.toplevel.fa | tr -d '\n' | wc
```
**Explanation**: `grep -v'` means "fetch lines **not** matching the pattern", so in this case this would be all sequence lines. We then **pipe** the result into another command, `tr -d '\n'`, which removes all the "newline" characters, and finally pipe the result again to `wc`, which counts the number of letters.  
Piping is a very useful feature in text manipulation, and we'll use it a lot in this tutorial. You can also pipe results into `less` if you want to view them - useful for checking large intermediate results. E.g.: `grep -v '>' Danio_rerio.GRCz11.dna.toplevel.fa | less`.

Now let's get the chromosomes list from the BED file. This can be done with:
```
$ cut -f1 danRer11.CRISPR.bed | sort | uniq
```
**Explanation**: `cut -f1` prints only the first column of the file (the chromosome). We only want unique (distinct) values, so we use the command `uniq`, however, this command only works for sorted data, so we must use `sort` first.  
Looks like the same chromosomes are present, but with slightly different naming. We'll take care of this later.  
We can also count how many targets are listed per chromosome with a very similar command:
```
$ cut -f1 danRer11.CRISPR.bed | sort | uniq -c
```
We can even sort the output by number of targets, like this:
```
$ cut -f1 danRer11.CRISPR.bed | sort | uniq -c | sort -k1 -n
```
**Explanation**: `sort -k1` means "sort by first column". `-n` tells the command to sort by numeric value rather than alphabetically.

## Using `awk` for working with tabular data
`awk` is a nice command line tool for working with tables. Let's see what it can do.  
Suppose we want to extract CRISPR targets in a certain range of genomic coordinates, e.g. chr1:1000000-2000000. We can do it using:
```
$ awk '$1 == "chr1" && $2 >= 100000 && $3 <= 200000' danRer11.CRISPR.bed | less
```
**Explanation**: $1 referes to the first column, $2 is the second etc. We can use any combination of boolean operators: `&&, ||, !, ()`...
We can also decide to print only specific fields using `{print $x}`. For example the following oneliner will count how many targets we have on the + and - strands in the above range:
```
$ awk '$1 == "chr1" && $2 >= 100000 && $3 <= 200000 {print $6}' danRer11.CRISPR.bed | sort | uniq -c
```

`awk` is also very useful for reformating text files. E.g.:
```
$ awk '{print $1":"$2"-"$3"("$6")""\t"$5}' danRer11.CRISPR.bed | head
chr1:1107-1127(-)       513.21
chr1:1111-1131(-)       706.302
chr1:1234-1254(-)       618.853
chr1:4188-4208(+)       769.012
chr1:4316-4336(+)       206.765
chr1:5007-5027(-)       838.946
chr1:5962-5982(-)       208.532
chr1:5965-5985(+)       907.262
chr1:5974-5994(-)       938.906
```
Of course you can always redirect the output to a new file using '>': `awk '{print $1":"$2"-"$3"("$6")""\t"$5}' danRer11.CRISPR.bed > danRer11.CRISPR.tsv`.

Finally, we can use `awk` for summing numbers. For example, let's calculate the mean score (column 5) in a given range:
```
$ awk '$1 == "chr1" && $2 >= 100000 && $3 <= 200000 {SUM+=$5; i+=1}END{print SUM/i}' danRer11.CRISPR.bed
555.81
```
**Explanation**: the first part (`$1 == "chr1" && $2 >= 100000 && $3 <= 200000`) just filters for the range of interest. `SUM` and `i` are variables - in `awk` there is no need to declare or initialize them, they just start from 0. For each row, we add the score ($5) to `SUM`, and also increment `i` by 1. At the end (`END`) - we print `SUM/i`.

## Substitutions using `sed`
`sed` is another great tool for manipulating text files (or "streams"). One nice feature is text substitution. The syntax is:
```
sed 's/<pattern>/<replace>/'
```
or:
```
sed 's/<pattern>/<replace>/**g**'
```
to replace multiple occurrences of the pattern in the same line.  
The pattern may be a simple text string. For example, the following command will remove the letters "chr" from each line, which will help us matching chromosome names to those found in the FASTA file:
```
$ sed 's/chr//' danRer11.CRISPR.bed | head
1       1107    1127    GTAGATGAGAGGTCACCGGC_553        513.21  -
1       1111    1131    AGCTGTAGATGAGAGGTCAC_350        706.302 -
1       1234    1254    GGAAAAATAACCTCCAAACC_436        618.853 -
1       4188    4208    GTAATAATCTCAGTTTATCG_292        769.012 +
1       4316    4336    GTAGTTGCCAGAATCACTAA_1138       206.765 +
1       5007    5027    AGATAAGTTACTTATAATCG_229        838.946 -
1       5962    5982    CTATCGAGAGGCATTACTGA_1132       208.532 -
1       5965    5985    GTAATGCCTCTCGATAGCTG_166        907.262 +
1       5974    5994    CGCACACCGCAGCTATCGAG_134        938.906 -
```
**Explanation**: we simply replace "chr" with "" (empty string.  
The pattern may also be a REGEX, using a specific [dialect](https://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html). As an example, let's get rid of the sequence in column 4:
```
$ sed 's/[ATGCN]*_//' danRer11.CRISPR.bed | head
chr1    1107    1127    553     513.21  -
chr1    1111    1131    350     706.302 -
chr1    1234    1254    436     618.853 -
chr1    4188    4208    292     769.012 +
chr1    4316    4336    1138    206.765 +
chr1    5007    5027    229     838.946 -
chr1    5962    5982    1132    208.532 -
chr1    5965    5985    166     907.262 +
chr1    5974    5994    134     938.906 -
```
We can even capture parts of the pattern and use them in the replacement value. Let's use this to separate the sequence from the number of off-targets, so instead of `GTAGATGAGAGGTCACCGGC_553` we'll get `GTAGATGAGAGGTCACCGGC  553`. This can be done like this:
```
$ sed 's/\([ATGCN]*\)_\([0-9]*\)/\1\t\2/' danRer11.CRISPR.bed | head
chr1    1107    1127    GTAGATGAGAGGTCACCGGC    553     513.21  -
chr1    1111    1131    AGCTGTAGATGAGAGGTCAC    350     706.302 -
chr1    1234    1254    GGAAAAATAACCTCCAAACC    436     618.853 -
chr1    4188    4208    GTAATAATCTCAGTTTATCG    292     769.012 +
chr1    4316    4336    GTAGTTGCCAGAATCACTAA    1138    206.765 +
chr1    5007    5027    AGATAAGTTACTTATAATCG    229     838.946 -
chr1    5962    5982    CTATCGAGAGGCATTACTGA    1132    208.532 -
chr1    5965    5985    GTAATGCCTCTCGATAGCTG    166     907.262 +
chr1    5974    5994    CGCACACCGCAGCTATCGAG    134     938.906 -
```
**Explanation**: we use `\(<something>\)` for indicating that we want to capture a part of the pattern, in this case, the sequence and the number (separately). Within the replace value, we use \1 to indicate the first captured value, \2 to indicate the second, and so on.  

**sed tip**: sometimes you find yourself in a situation where you need to replace Linux paths, which contain the character '/'. This may confuse `sed`, since we used '/' as part of the the command syntax. Turns out you can actually use any character instead, e.g.:
```
$ echo '/path/to/myFile.txt' | sed 's@/to/@/to/new/@'
/path/to/new/myFile.txt
```

## Using `sed` to extract line ranges
Another useful feature of the `sed` tool is its ability to fetch certain ranges of lines.  
Recall that we can use `head` to get the first lines of a file, e.g.:
```
$ head -100 danRer11.CRISPR.bed
```
will print the first 100 lines. The same can be achieved using `sed`:
```
$ sed -n '1,100p;100q' danRer11.CRISPR.bed
```
**Explanation**: we tell `sed` to print (p) lines 1-100, and then, at line 100, quit (q).  
Of course, we can do the same with any range of line numbers, e.g.:
```
$ sed -n '10000,20000p;20000q' danRer11.CRISPR.bed
```
We can even tell `sed` to only print every n-th line. Let's use that to sample 100k targets from the line range 1,000,000-2,000,000:
```
$ sed -n '1000000~10p;2000000q' danRer11.CRISPR.bed | less
```
**Explanation**: we tell `sed` to start at line 1000000, then print every 10th line, and quit at line 2000000.  

Two more useful `sed` tricks:
* To combine several `sed` commands, use `-e`: `sed -e 's@_@/@' -e 's/chr//' danRer11.CRISPR.bed | less`
* To modify a file in-place, use `sed -i` (but be careful - it's irreverssible).

