_Visit the [Performance Benchmarks main page](Performance.md)_<br>
_Visit the [TSV Utilities main page](../README.md)_

# April 2018 Comparative Benchmarks Update

* [Overview](#overview)
* [Benchmark Tests](#benchmark-tests)
* [TSV Utilities performance improvements post the March 2017 study](#tsv-utilities-performance-improvements-post-the-march-2017-study)
* [Benchmark results](#benchmark-results)
* [Test details](#test-details)

## Overview

This is an update to the [March 2017 Comparative Benchmarks Study](comparative-benchmarks-2017.md). The 2017 study compared eBay's TSV Utilities to a number of other tools as a way to gauge the performance of the D programming language. This page contains updates to those benchmarks.

The goal of the 2017 study was to evaluate performance of an application written in a straightforward fashion, without going to unusual lengths optimize performance. The 2018 version of eBay's TSV Utilities are still written in this style, but have had performance improvements since. The most notable is the use of Link Time Optimization (LTO) and Profile Guided Optimization (PGO). These are compiler technologies available via the LDC compiler.

See the [main performance benchmarks page](Performance.md) for more information on the goals of the study and the approach used. See the [2017 study](comparative-benchmarks-2017.md) for details about the individual benchmarks. This page is primarily reporting the updated results. The 2017 study also includes several analyses not included in the 2018 study, for example, a comparison of DMD and LDC compilers.

The 2018 study adds one additional benchmark to the 2017 set. Benchmarks were run on MacOS and Linux. Only the faster tools from the 2017 study were included the 2018 update, the slower tools were dropped.

## Benchmark tests

Seven tasks were used as benchmarks. Two forms of row filtering: numeric comparisons and regular expression match. Join two files on a common key. Simple statistical calculations (e.g. mean of column values). Convert CSV files to TSV. Two column selection tests (aka 'cut'), one with medium width lines, the other with short lines. The second column selection test was added in the 2018 study.

Reasonably large files were used. One 4.8 GB, 7 million rows, another 2.7 GB, 14 million rows, and a third file 1.7 GB, 86 million lines. Tests against smaller files gave results consistent with the larger file tests.

The MacOS benchmarks were run on a Mac mini, 16 GB RAM, SSD drives, 3 GHz Intel i7 (2 cores). This machine is not quite as fast as the MacBook Pro used for the 2017 study, but the performance is not far off.

The Linux benchmarks were run using a commodity cloud machine with a 16 CPU Intel (Haswell) 2095 MHz processor and 32 GB RAM. The 2017 benchmarks did not include Linux.

The MacOS benchmarks were run 25 times, the Linux benchmarks 35 times. The 10th percentile numbers are reported here. Median and quartile numbers were consistent and could have been used as well.

## TSV Utilities performance improvements post the March 2017 Study

eBay's TSV Utilities became materially faster between the March 2017 and April 2018 studies. The tables below show these changes. The pre-built binary for release v1.1.11 was used as a proxy for the March 2017 versions. v1.1.11 was released after the March 2017 study, but is largely unchanged.

| Benchmark | MacOS<br>March 2017<br>(v1.1.11) | MacOS<br>April 2018<br>(v1.1.19) | MacOS<br>Delta | Linux<br>March 2017<br>(v1.1.11) | Linux<br>April 2018<br>(v1.1.19) | Linux<br>Delta |
| ----------------------------- | -------: | -------: | -------: | -------: | -------: | -------: |
| **Numeric row filter**        |     4.25 |     3.35 |      21% |     6.32 |     5.48 |      13% |
| **Regex row filter**          |    10.11 |     8.28 |      18% |     9.72 |     8.80 |       9% |
| **Column selection**          |     4.26 |     2.93 |      31% |     5.52 |     4.79 |      13% |
| **Column selection (narrow)** |    25.12 |    10.18 |      59% |    15.41 |     8.26 |      46% |
| **Join two files**            |    23.94 |    21.17 |      12% |    28.77 |    26.68 |       7% |
| **Summary statistics**        |    16.97 |     9.82 |      42% |    22.68 |    15.78 |      30% |
| **CSV-to-TSV**                |    30.70 |    10.91 |      64% |    34.65 |    20.30 |      41% |

There were three main sources of performance improvements between the 2017 and 2018 studies:
* Using Link Time Optimization (LTO) and Profile Guided Optimization (PGO) in the builds.
* Performance improvements in D, the standard libraries, and compilers.
* The use of output buffering in the tools. This had the largest impact on the narrow file column selection test.

## Benchmark results

Updates to the benchmarks from the 2017 study are provided below. The top four tools in each benchmark are shown.

### A note about comparisons between individual tools

The intent of the study is to gain an overall picture of the performance of typical D programs. Comparisons across a range of tests and tools helps with this. However, any one test or comparison between individual tools is less meaningful. Each tool will have its own design and functionality goals. These goals will affect performance, often in ways that are a deliberate design trade-off on the part of the developer.

A few specific considerations:
* Tools accepting CSV data must handle escape characters. This is computationally more expensive than a strict delimited format like TSV. Supporting both CSV and TSV makes optimizing the TSV case challenging.
* Some CSV implementations support embedded newlines, others do not. Embedded newline support is more challenging because highly optimized "readline" routines cannot be used to find record boundaries.
* Handling arbitrary expression trees (ala `Awk`) is more computationally complex than handling a single conjunctive or disjunctive expression list as `tsv-filter` does.
* Some tools use multi-threading, others do not. (eBay's TSV Utilities do not.) This is a design trade-off. Running multiple threads can improve run-times of individual tools. However, this may reduce overall throughput when several tools are chained in a command pipeline.

### Top four in each benchmark

The tables below show fastest times for each benchmark. One table each for MacOS and Linux. Times are in seconds. A description of the individual tests can be found in the [March 2017 study](comparative-benchmarks-2017.md). eBay's TSV Utilities are shown in italics. Reference information for all tools is in the [Test Details](#test-details) section.

#### MacOS: Top-4 in each benchmark

| Benchmark                     |       Tool/Time |    Tool/Time | Tool/Time |    Tool/Time |
| ----------------------------- | --------------: | -----------: | --------: | -----------: |
| **Numeric row filter**        |    _tsv-filter_ |         mawk |   GNU awk |        csvtk |
| (4.8 GB, 7M lines)            |            3.35 |        15.06 |     24.25 |        39.10 |
| **Regex row filter**          |             xsv | _tsv-filter_ |   GNU awk |         mawk |
| (2.7 GB, 14M lines)           |            7.03 |         8.28 |     16.47 |        19.40 |
| **Column selection**          |    _tsv-select_ |          xsv |     csvtk |         mawk |
| (4.8 GB, 7M lines)            |            2.93 |         7.67 |     11.00 |        12.37 |
| **Column selection (narrow)** |             xsv | _tsv-select_ |   GNU cut |        csvtk |
| (1.7 GB, 86M lines)           |            9.22 |        10.18 |     10.65 |        23.01 |
| **Join two files**            |      _tsv-join_ |          xsv |     csvtk |              |
| (4.8 GB, 7M lines)            |           21.78 |        60.03 |     82.43 |              |
| **Summary statistics**        | _tsv-summarize_ |          xsv |     csvtk | GNU Datamash |
| (4.8 GB, 7M lines)            |            9.82 |        35.32 |     45.59 |        71.60 |
| **CSV-to-TSV**                |       _csv2tsv_ |          xsv |     csvtk |              |
| (2.7 GB, 14M lines)           |           10.91 |        14.38 |     32.49 |              |

#### Linux: Top-4 in each benchmark

| Benchmark                     |       Tool/Time |    Tool/Time |    Tool/Time | Tool/Time |
| ----------------------------- | --------------: | -----------: | -----------: | --------: |
| **Numeric row filter**        |    _tsv-filter_ |         mawk |      GNU awk |     csvtk |
| (4.8 GB, 7M lines)            |            5.48 |        11.31 |        42.80 |     53.36 |
| **Regex row filter**          |             xsv | _tsv-filter_ |         mawk |   GNU awk |
| (2.7 GB, 14M lines)           |            7.97 |         8.80 |        17.74 |     29.02 |
| **Column selection**          |    _tsv-select_ |         mawk |          xsv |   GNU cut |
| (4.8 GB, 7M lines)            |            4.79 |         9.51 |         9.74 |     14.46 |
| **Column selection (narrow)** |         GNU cut | _tsv-select_ |          xsv |      mawk |
| (1.7 GB, 86M lines)           |            5.60 |         8.26 |        13.60 |     23.88 |
| **Join two files**            |      _tsv-join_ |          xsv |        csvtk |           |
| (4.8 GB, 7M lines)            |           26.68 |        68.02 |        98.51 |           |
| **Summary statistics**        | _tsv-summarize_ |          xsv | GNU Datamash |     csvtk |
| (4.8 GB, 7M lines)            |           15.78 |        44.38 |        48.51 |     59.71 |
| **CSV-to-TSV**                |       _csv2tsv_ |          xsv |        csvtk |           |
| (2.7 GB, 14M lines)           |           20.30 |        26.82 |        44.82 |           |

## Test details

Tests were run on April 14, 2018. The latest released version of each tool was used. Details needed to reproduce the tests are given below. The [March 2017 study](comparative-benchmarks-2017.md) has a more detailed description of the individual tests, however, everything need to reproduce the tests can be found here.

### Machines

* MacOS: Mac Mini; 16 GB RAM; SSD drives; MacOS High Sierra version 10.13.3
* Linux: 16 CPUs; Intel (Haswell) 2095 MHz; 32 GB RAM Ubuntu 16.04 (commodity cloud machine)

### Tools

GNU Awk, GNU cut and mawk are available on most platforms via standard package managers. The xsv and csvtk packages were downloaded from pre-built binaries on their respective GitHub releases pages. eBay's TSV Utilities releases are also from the pre-built binaries available from GitHub.

* [GNU Awk](https://www.gnu.org/software/gawk/) - Version 4.2.1. Written in C.
* [mawk](https://invisible-island.net/mawk/mawk.html) - MacOs: Version 1.3.4; Linux: Version 1.3.3. Written in C.
* [GNU cut](https://www.gnu.org/software/coreutils/coreutils.html) - MacOS: Version 8.29; Linux: Version 8.25. Written in C.
* [GNU Datamash](https://www.gnu.org/software/datamash/) - Version 1.3. Written in C.
* [xsv](https://github.com/BurntSushi/xsv) - Version 0.12.2. Written in Rust.
* [csvtk](https://github.com/shenwei356/csvtk) - Version v0.14.0-dev. Written in Go.
* [eBay's TSV Utilities](https://github.com/eBay/tsv-utils) - Version v1.1.19 (this toolkit). Written in D.

### Data files

* `hepmass_all_train.tsv` - 7 million lines, 4.8 GB. The HEPMASS training set from the UCI Machine Learning repository, available [here](https://archive.ics.uci.edu/ml/datasets/HEPMASS). (The `all_train.csv.gz` file. Use `csv2tsv` to convert to TSV.)
* `TREE_GRM_ESTN_14mil.[csv\|tsv]` - 14 million lines, 2.7 GB. From the Forest Inventory and Analysis Database, U.S. Department of Agriculture. The first 14 million lines from the TREE_GRM_ESTN_.csv file, available [here](https://apps.fs.usda.gov/fia/datamart/CSV/datamart_csv.html).
* `googlebooks-eng-all-1gram-20120701-a.tsv` - 86 million lines, 1.7 GB. The google one-gram file for the letter 'a', available from the [Google Books ngram datasets](http://storage.googleapis.com/books/ngrams/books/datasetsv2.html).

To create the left and right data files used in the join test:
```
$ number-lines -s line hepmass_all_train.tsv > hepmass_numbered.tsv
$ tsv-select -f 1-16 hepmass_numbered.tsv | tsv-sample -H -v 111 > hepmass_left.shuf.tsv
$ tsv-select -f 1,17-30 hepmass_numbered.tsv | tsv-sample -H -v 333 > hepmass_right.shuf.tsv
$ rm hepmass_numbered.tsv
```

### Command lines

Command lines used in the tests are given below. The command lines are the same for Linux and MacOS. Both `awk` and `gawk` refer to GNU awk.

Numeric filter test:
```
$ [awk|mawk|gawk] -F $'\t' -v OFS='\t' '{ if ($4 > 0.000025 && $16 > 0.3) print $0 }' hepmass_all_train.tsv > /dev/null
$ csvtk filter2 -t -l -f '$4 > 0.000025 && $16 > 0.3' hepmass_all_train.tsv > /dev/null
$ tsv-filter -H --gt 4:0.000025 --gt 16:0.3 hepmass_all_train.tsv > /dev/null
```

Regular expression filter command lines:
```
$ [awk|gawk|mawk] -F $'\t' -v OFS='\t' '$10 ~ /[RD].*ION[0-2]/' TREE_GRM_ESTN_14mil.tsv > /dev/null
$ csvtk grep -t -l -f 10 -r -p '[RD].*ION[0-2]' TREE_GRM_ESTN_14mil.tsv > /dev/null
$ xsv search -s COMPONENT '[RD].*ION[0-2]' TREE_GRM_ESTN_14mil.tsv > /dev/null
$ tsv-filter -H --regex 10:'[RD].*ION[0-2]' TREE_GRM_ESTN_14mil.tsv > /dev/null
```

Column selection command lines:
```
$ [awk|gawk|mawk] -F $'\t' -v OFS='\t' '{ print $1,$8,$19 }' hepmass_all_train.tsv > /dev/null
$ csvtk cut -t -l -f 1,8,19 hepmass_all_train.tsv > /dev/null
$ cut -f 1,8,19 hepmass_all_train.tsv > /dev/null
$ tsv-select -f 1,8,19 hepmass_all_train.tsv > /dev/null
$ xsv select 1,8,19 hepmass_all_train.tsv > /dev/null
```

Column selection (narrow) command lines:
```
$ [awk|gawk|mawk] -F $'\t' -v OFS='\t' '{ print $1,$3 }' googlebooks-eng-all-1gram-20120701-a.tsv > /dev/null
$ csvtk cut -t -l -f 1,3 googlebooks-eng-all-1gram-20120701-a.tsv > /dev/null
$ cut -f 1,3 googlebooks-eng-all-1gram-20120701-a.tsv > /dev/null
$ tsv-select -f 1,3 googlebooks-eng-all-1gram-20120701-a.tsv > /dev/null
$ xsv select 1,3 googlebooks-eng-all-1gram-20120701-a.tsv > /dev/null
```

Join files command lines:
```
$ csvtk join -t -l -f 1 hepmass_left.shuf.tsv hepmass_right.shuf.tsv > /dev/null
$ tsv-join -H -f hepmass_right.shuf.tsv -k 1 hepmass_left.shuf.tsv -a 2,3,4,5,6,7,8,9,10,11,12,13,14,15 > /dev/null
$ xsv join 1 -d $'\t' hepmass_left.shuf.tsv 1 hepmass_right.shuf.tsv > /dev/null
```

Summary statistics command lines:
```
$ csvtk stat2 -t -l -f 3,5,20 hepmass_all_train.tsv > /dev/null
$ cat hepmass_all_train.tsv | datamash -H count 3 sum 3,5,20 min 3,5,20 max 3,5,20 mean 3,5,20 sstdev 3,5,20 > /dev/null
$ tsv-summarize -H --count --sum 3,5,20 --min 3,5,20 --max 3,5,20 --mean 3,5,20 --stdev 3,5,20 hepmass_all_train.tsv > /dev/null
$ xsv stats -s 3,5,20 hepmass_all_train.tsv > /dev/null
```

CSV-to-TSV command lines:
```
$ csvtk csv2tab TREE_GRM_ESTN_14mil.csv > /dev/null
$ csv2tsv TREE_GRM_ESTN_14mil.csv > /dev/null
$ xsv fmt -t '\t' TREE_GRM_ESTN_14mil.csv > /dev/null
```

Note: For `xsv`, the more correct way to operate on unescaped TSV files is to first pipe the data through `xsv input --no-quoting` first. e.g.
```
$ xsv input --no-quoting hepmass_all_train.tsv | xsv select 1,8,19 > /dev/null
```

However, adding an additional command invocation runs counter to the goals of the benchmark exercise, so it was not done in these tests.
