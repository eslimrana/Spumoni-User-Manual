# Spumoni-User-Manual
# SPUMONI

SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).** 

SPUMONI is based on another software tool called [MONI](https://github.com/maxrossi91/moni) which is a MEM-finder and aligner for pan-genomes. `MONI` uses prefix-free parsing of the text to build the Burrows-Wheeler Transform (BWT) of the reference collection, the suffix array (SA) samples at the beginning and end of each run, and the threshold positions. 

## Step 1: Building an Index

After installing SPUMONI on your machine, the first step would be to build an index over the reference you want to use for your experiment. This reference will be a FASTA file (and it can be a multi-FASTA for pan-genomes). SPUMONI allows you to either build it over a single FASTA file, or you can specify a list of genomes that you want to include in the index. 

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

# Detailed Wiki Information

## 1. Building SPUMONI Indexes

### Overview

There are two main approaches to build a SPUMONI index: (1) using a single FASTA file, or (2) providing a list of files. Using the second approach allows users to specify groups of genomes in order to perform multi-class classification. Before discussing how to build the index in these two approaches, there are a handful of command line options that are worth mentioning:

***

### **What value do you want to compute?**

SPUMONI is capable of computing matching statistics (MS) and pseudo-matching lengths (PML) which can be thought of as an approximation of MS but faster to compute. Generally speaking, if you are interested in just classifying reads using PMLs should be sufficient and preferred. The options below are required when building an index, and you can use both if you would like.

- `-M`: builds an index that can be used for **computing matching statistics (MS)**. It will result in three files with the following extensions: `*.ms`, `*.slp`, and `*.msnulldb`.
- `-P`: builds an index that can be used for **computing pseudo-matching lengths (PML)**. It will result in two files with the following extensions: `*.spumoni`, and `*.pmlnulldb`.

***

### **Should I use minimizer digestion when building the index?**

For larger references (>2GB), it would be recommended to use minimizer digestion to speed up index building and query time. When building the index, you should specify one and only one of these options below.

- `-n`: tells `spumoni` to not apply a minimizer digestion to the input sequence. It will build an index for the FASTA file directly.
- `-m`: tells `spumoni` to use minimizer digestion where each minimizer is a unique character
- `-t`: tells `spumoni` to use minimizer digestion where each minimizer is a sequence of k DNA-characters (ACGT)

***

### **Other relevant options for classification:**

- `-d`: builds a data-structure called the document array which is needed if you would like to perform multi-class classification.

***

### Building from single FASTA file

This approach is pretty straight forward. To build both a MS and PML indexes, you can run the following command:

```sh
./spumoni build -r <ref_file> -o <index_prefix> -n -M -P
```

If you would only like one of the indexes, you can remove the relevant option. Also, if you would like to turn on a form of minimizer digestion, you can replace `-n` with `-m` or `-t`.

***

### Building from a list of files

This approach is convenient if you have a collection of genomes that you would like to build an index over and/or want to group genomes into groups and classify the reads into those groups. Firstly, we will go over how to organize your list of files for `spumoni` to understand how to build the index.

Here is example of a file-list. Each line starts with a path to a FASTA file you want to index, and it followed by space, then an id number corresponding to a class. The id numbers must start at 1, and increment by 1 as you move from group to group. If you do not want to perform multi-class classification, you do not need to specify the id number of each line, it can just be the path.

After the id number, SPUMONI will treat anything else as a comment so its possible to add descriptors as shown below like "E. coli" or "Human" if you want to store metadata labeling each group within your filelist.

```
/path/to/genomes/ecoli_1.fa 1 E. coli
/path/to/genomes/ecoli_2.fa 1 E. coli
/path/to/genomes/salmonella_1.fa 2 Salmonella
/path/to/genomes/salmonella_2.fa 2 Salmonella
/path/to/genomes/human_1.fa 3 Human
/path/to/genomes/human_2.fa 3 Human
```

Now, in terms of how to run `spumoni` to build the index using a file-list. You can refer to the command below, the main change is you will use the `-i` to provide the path to file-list using the same format as the example above.

```sh
./spumoni build -i <file_list> -o <index_prefix> -n -M -P
```

## 2. Running SPUMONI on Input Reads
### Overview

SPUMONI can classify sequencing reads either in binary or multi-class mode. We will discuss each of those modes below, but we will discuss some important points to understand.

**What are each of these output files?**
- `*.lengths`: contains the MS/PMLs for each read
- `*.pointers`: contains the pointers (indexes where each half-mem begins) for each read
- `*.doc_numbers`: contains the document ids encountered for each read 
- `*.report`: contains the overall classifications and statistics for each read

**Relevant command-line options:**
- `-M`: use **matching-statistics** to classify the reads
- `-P`: use **pseudo-matching length** to classify the reads
- `-d`: use document array data-structure to classify what group each read is from
- `-c`: writes out a classification report with all the reads
- `-t`: number of threads used
***

### Binary classification

In this mode, `spumoni` will just classify each read for whether it is present in the database or not. It can be run using the command below, it will use matching statistics to classify the reads. **Importantly**, it is crucial to make sure you are following the same options that you used when building the index, the command below is using `-n` to avoid minimizer digestion, and therefore the pre-build index should have used the `-n` command as well.

```sh
./spumoni run -r <index_prefix> -p <pattern_file> -M -n -c
```
***

### Multi-class classification

In this mode, `spumoni` will classify each read as being present or not in the database, as well as which group it is mostly likely from. In the command below, we add the `-d` flag to tell `spumoni` it should use the document array to classify the reads in a multi-class approach.

```sh
./spumoni run -r <index_prefix> -p <pattern_file> -n -M -c -d
```

## 3. Example Workflows with SPUMONI
### Overview

This page shows some example workflows to show how you would run `spumoni` in different scenarios.

#### Scenario 1:

- Using a single FASTA file, applying minimizer digestion, and classifying using matching statistics

```
./spumoni build -r ecoli_genome.fa -o index/ecoli_index -m -M
./spumoni run -r index/ecoli_index -p reads.fa -m -M
```

#### Scenario 2:

- Using a single FASTA file, not using minimizer digestion, and classifying with pseudo-matching lengths and writing to report.

```sh
./spumoni build -r ecoli_genome.fa -o index/ecoli_index -P -n
./spumoni run -r index/ecoli_index -p reads.fa -P -c -n 
```

#### Scenario 3:

- Using a list of FASTA files, using minimizer digestion, performing multi-class classification using matching statistics, and writing to report.

```sh
./spumoni build -i file_list.txt -o index/complete_index -M -m
./spumoni run -r index/complete_index -p reads.fa -d -M -c -m
```

#### Scenario 4:

- Using a list of FASTA files, not using minimizer digestion, performing multi-class classification using pseudo-matching lengths, and writing to report.

```sh
./spumoni build -i file_list.txt -o index/complete_index -P -n
./spumoni run -r index/complete_index -p reads.fa -d -P -c -n
```
