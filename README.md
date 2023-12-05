# Spumoni-User-Manual
# SPUMONI

SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).** 

SPUMONI is based on another software tool called [MONI](https://github.com/maxrossi91/moni) which is a MEM-finder and aligner for pan-genomes. `MONI` uses prefix-free parsing of the text [2][3] to build the Burrows-Wheeler Transform (BWT) of the reference collection, the suffix array (SA) samples at the beginning and end of each run, and the threshold positions[1]. 

**For more details on installation/using SPUMONI, please refer to the [wiki page](https://github.com/oma219/spumoni/wiki/1.-Home) which covers those areas in detail.**

## Step 1: Building an Index

After installing SPUMONI on your machine, the first step would be to build an index over the reference you want to use for your experiment. This reference will be a FASTA file (and it can be a multi-FASTA for pan-genomes). SPUMONI allows you to either build it over a single FASTA file, or you can specify a list of genomes that you want to include in the index. [See the wiki for more details.](https://github.com/oma219/spumoni/wiki/4.-Building-SPUMONI-Indexes) 

For example, if you would want to do host depletion, you could index a human genome and build both the matching statistic and pseudo-matching lengths index using the command below: 

```sh
./spumoni build -r human_genome.fa -M -P -m -o /path/to/index
```
The command above will build an index for the reference file you provide. This command uses `-M` and `-P` which means it will build an index for computing matching statistics (MSs), and another index for computing pseudo-matching statistics (PMLs). Our experiments show that using PMLs are more accurate at binary classification while also being ~3x faster, therefore that would be our recommendation. The `-o` flag provides an output prefix for SPUMONI to use so it can build the index and any intermediate files it needs.

## Step 2: Running Classification

Once you have an index for desired experiment, you can use the `spumoni run` command to generate either MSs or PMLs for each read against the reference file that you just indexed. Now at this step, you provide the output prefix that was used to build the index.

```sh
./spumoni run -r /path/to/index -p reads.fa -P -c
```

This command uses `-P` for computing PMLs (if you want MSs, use `-M` instead) which will be used to classify the reads. Additionally, the command uses the `-c` option to write out the classifications to a report file.
