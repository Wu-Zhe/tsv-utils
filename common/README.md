_Visit the eBay TSV utilities [main page](../README.md)_

# Utility functions

This directory contains utility functions shared by multiple TSV utility tools. A few that may be of more general interest:
* **InputFieldReordering** - A class that creates a reordered subset of fields from an input line. Used to operate on a subset of fields in the order specified on the command line. *File: utils.d*.
* **BufferedOutputRange** - An OutputRange with an internal buffer used to buffer output.
  Intended for use with stdout, it is a significant performance benefit. *File: utils.d*.
* **quantile** - Calculates a cummulative probability for values in a data set. Supports the same interpolation methods as the quantile function in R and many other statistical packages. *File: numerics.d*.
* **rangeMedian** - Finds the median in a range. Implements via the faster of `std.algorithm.topN` or `std.algorithm.sort` depending on the Phobos version. *File: numerics.d*.
* **formatNumber** - An alternate print format for numbers, especially useful when doubles are being used to represent integer and float values. *File: tsv_numerics.d*.
* **getoptInorder** - A cover for `std.getopt` that processes command line arguments in the order given on the command line. *File: getopt_inorder.d*.
