# CRISPR-Genomics

## Introduction

For the Genomics analysis pipeline, we decided to implement the following steps:

1. Get list of names of bacterias know to contain CRISPR regions (from
crisprdb](http://crispr.u-psud.fr/crispr/) parsing it through script
(no API unfortunately)
2. Download (automatically?) the genomes of these bacterias from NCBI.
3. Run pilercr on each genome and produce report on spacers.
4. Go through the report and extract spacer sequences, for each
spacer, check in the corresponding genome if there is a sequence
(aside from the spacer sequence in the crispr region) that matches it. 
5. Go through matches and look at annotation for these regions 
6. If region not annotated, potential virus inserted in chromosomal
dna.

## 1 Retrieval of the information on the bacterias of interest

The python script *get-bacteria-links-from-cripsrdb.py* connects to
[crisprdb](http://crispr.u-psud.fr/crispr/) and retrieves all the
links to individual baxterias' webpage. The output of that script is a
text file: *bacteria_links_crispr.txt* which is fed to the next script

The python script *extract-refseq-from-crisprdb.py* connects to each
of the bacterias' individual pages and gets info on them, such as the
reference genome studied, the genbank id, the number of CRISPR cluster
found. The output of that script is a text file:
*bacteria_info_crisprdb.txt* which is fed to the next script in Part 2.

## 2 Downloading of the genomes of bacterias

The python script *download-genomes-from-ncbi.py* makes use of the
REST API of NCBI (Entrez) to download each genomes into a local
file. There are about 5200 of them, amounting to a total of 10 Gb of
genomic information. Make sure that you have enough room to save them.

### Note

You can avoid this step (and spare NCBI's servers) by downloading the
archive (compressed to 2.5gb) from [this
address](https://altersid.net/owncloud/index.php/s/d4GhPsbQ3a2wNfO). The
corresponding sha1sum is:

```
> sha1sum genomes_full.tar.bz2
acff7aa9d6fb29977f1471640a901c42a8d98237  genomes_full.tar.bz2
```

## 3 Generation of reports on CRISPR

The python script *generate-crispr-reports.py* makes use of the crispr
tool to produce one file/report per bacteria in the *genomes*
folder. The reports are output to the *crispr* folder.

  
## 4 Analysis of the reports

The python script *count-spacers-occurences-in-genomes.py* parses the
crispr reports to search for repeats of spacers in non-crispr regions
of the genomes. At the moment, it ignores spacers that are too short 
15 nucleotides). There is a lot of result, probably need to add a filtering
step.

The output of that analysis is the file *crispr-repeats.txt*

### Additional filtering step

As we discovered, there are many false positive that came out of the naive filtering step implemented in *count-spacers-occurences-in-genomes.py*. To narrow down the results further, another script was added to our pipeline *filter-spacers.py* which reduces the hits by about 75%. The script goes through all the spacers in the *crispr-repeats.txt* report, and removes the spacers that can be found in other CRISPR arrays, or in the same array as it was found in initially, but as a substring of another spacer. The output of that script is *filter_spacer_count.txt*

## 5 Annotations

The python script *retrieve-annotations.py* makes use again of the REST API of NCBI to download the annotations of the genomes where spacers of interest can be found in (see *filter_spacer_count.txt* for the list). The annotations are saved in the *annotations/* folder.


