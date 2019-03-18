The Tidy FASTA (TFF) and Tidy FASTA Binary (TFB) formats
--------------------------

## Tidy data
The basics of tidy data are as follows:  
- Data is structured as a table
- Rows represent a single observation
- Columns represent a single variable

## FASTA/FASTQ files are not tidy

## The Tidy FASTA Format (TFF)
The TFF format is just a transform of the FASTA/FASTQ formats into
tidy format. Because it is column-oriented, it can represent both
FASTA and FASTQ in the same format (by simply placing a "NA" in the
third and fourth columns). It can also store paired-end FASTQ data
in the same file by appending the mate's info to the first read's row.
Missing mate's again receive "NA" in their fields.

### Definition
The TFF format is as follows:
- Each row is a read or sequence
- The first column is the sequence identifier
- The second column is the sequence
- The third column is the optional annotations of FASTQ (which may be a single "NA" value)
- The fourth column is the quality scores (which may be a single "NA" value)

A TFF file may have 8 columns in the case of paired-end FASTQ data, where the
last four columns are the mate's identifier, sequence, annotations and qualities.

## The Tidy FASTA Binary (TFB) format
TFB is a binary equivalent to TFF.
