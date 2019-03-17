Simple Phenotype Data File Spec
---------------------------------

## Genomic data is largely tidy in nature
As discussed, previously, genomic data is largely tidy. Phenotype
data is often used in the analysis of such data. Having both in tidy
format facilitates co-analysis of these datasets.

## The sample is the atomic unit of the phenotype
While we have defined "sub-atomic" genomic units of the sample such
as genomic coordinates, phenotype data always comes at the sample level.
The sample may contain more than one entity (e.g. both tumor and normal
cells in a pathology slide, or many species in a metagenome sample).
However, we cannot observe phenotypic variables at a sub-sample level.

## Tidy phenotype data specification
Below, we outline some thoughts about designing a 
tidy phenotype data format.
- The sample is the atomic unit.
- All groups are built up from sample observations
- Every column contains only one variable. Variable
columns may, however, be lists, assuming they are
correctly formatted.