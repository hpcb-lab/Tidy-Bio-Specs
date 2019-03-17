Tidy Bio Specs
-------------------------------

## Motivation
During the third and fourth years of my PhD I've had the unexpected
task of designing file formats for bioinformatics programs.
I've also been involved in
many discussions around handling the
[Graphical Fragment Assembly]() format and designing
programs for parsing, manipulating, and emitting similar
files. Over the course of these adventures I have learned
some hard lessons. I think this knowledge is worth remembering,
so I wrote it down. I've decided it's also worth sharing, so I've
posted it here in the hopes that the next generation of graduate
students might be saved some pain!

Here, I discuss some basic principles for organizing 
data in ways that encourage good science and preserve
researcher sanity. I focus on aspects of "tidyness" (Wickham, 2014)
in driving data structure and format design. I describe the tidy
or tidy-translatable nature of widely-adopted bioinformatics formats
and my hypothesis that tidyness has contributed to their success.
I then briefly discuss other considerations beyond data structure
to consider when designing bioinformatics file formats.

## Tidy principles
The [tidy data principles](http://vita.had.co.nz/papers/tidy-data.pdf)
are guidelines set forth to help describe datasets that
can be easily processed. While originally designed with the R statistical
programming language in mind, the principles of tidy data are language
and data agnostic.

Some basics of tidy data, distilled from Wickham, 2014, are as follows:  
 1. Data is organized as a table, with rows and columns
 2. Each row represents a single observation
 3. Each column represents a single variable
 4. From (3) and (4), it follows that each value
 (a cell within the table) contains a single observation
 of a single variable.
 5. From (2) and (4), a single row describes all varaibles
 measured on an observation.
 6. From (3) and (4), a single column describes all values that
 measure the same attribute. For example, a column "Height" can
 never contain a value that is in units of temperature.
 7. Column headers are variable names, not groups or observations
 8. Each column stores one AND ONLY one variable.
 9. Variables are stored in columns, and not rows.
 10. Observations are stored in rows, and not columns.
 11. Each type of observation unit is stored in its own table. Do 
 not mix units of different types; give them their own tables.

From this, a basic structure is easy to define. Data should be
organized in tables, with a coherent and singular unit of observation
listed by row. Every variable should receive its own column, and each
cell should contain one and only one values (which may be NaN, missing,
"NA", etc. if no data was observed for that variable).

## Tidy-translatable data
Some data is not tidy, but is easily convertible to tidy format.
We will call such data *tidy-translatable*.
A FASTA file, for example, is not tidy: rows have no meaning, and
there is only one column containing both identifiers and sequences.
However, this data could easily be translated into a two-column tidy
format, where the first column contains the identifier and the second
contains the sequence. In fact, such a format could be easily extended to
encode FASTQ by simply appending two columns. Column-orienting 
the data would also make it simple to maintain the identifiers / sequences
and the annotations / qualities in separate files.

## Examples of existing tidy(-ish) genomic formats
Many functional genomic formats already in widespread use are tidy or
tidy-translatable in nature.
Examples include:  

**SAM/BAM**:
 - Every row is a single observation (a sequencing read).
 - Each column stores one and only one variable (e.g. read name, sequence, CIGAR, etc).
 - A header, which in the case of BAM is not tidy, but is tidy-translatable, defines
   each variable in the file.

**VCF/BCF**:
 - Every row is a position in the genome (chromosome + position in linear space on the
 reference).
 - Each column (at least for the core 7 mandatory VCF columns) is tidy in nature and
 defined in a tidy-translatable header.
 - Format + Sample fields are tidy-translatable.

 The tidy nature of VCF makes it easy to index. As each row is a position, and the columns
 are well-defined, Tabix can index the file in a sane and predictable way. This is indeed
 true for any chromosome- and position-oriented, row-based data format (see the Tabix help
 for details). The same is true of BAM as well.

**BED**:
 BED is a perfectly-tidy format, in all of its variants. 
 - Each observation is a genomic interval,
 and is represented by a row. 
 - Each column contains one and only one variable, which are defined by a specification
 which functions as an implicit header.

**GTF/GFF**:
 GTF/GFF are BED variants, and are strictly tidy in nature to the best of the author's knowledge.


One notable format that is not tidy is FASTQ/FASTQ.   

**FASTA/FASTQ**:
 - While FASTA and FASTQ are not tidy as they aren't tables,
 do not have individual row as for observations, and contain a
 single column for all variables, they are in fact tidy-translatable.
 A single format containing four columns with each row representing a 
 sequence could represent both FASTA and FASTQ without any fundamental
 changes to the format's backing data.

## Proposition: tidy data formats last for good reason
We have discussed above how many of the most widely adopted
genomics formats are tidy or tidy-translatable. We will now argue
the position that the tidyness of these formats has
contributed to their longevity and that specific properties of tidy
data contribute to the tendency of these formats to become tidy or
tidy-translatable over time.

### Tidy formats are easily indexed
One of the primary challenges of genomic data is scale. It is
not uncommon to encounter files with millions or billions of observations.
At the same time, it is desirable to have random access to these files,
as we may be interested in only a select observation. Indices (such as 
BAM's ".bai" index and VCF's ".tbi" index) enable random-access to these
large files. This indexability is derived from their tidy nature - because
each row is a single observation, we can easily build programs that
generate and process indices for such files. Such indices enable the
necessary performance for operating on subsets of genomic data
at scale.

### Tidy formats are human readable
Tidy structures are easily represented as comma-separated or tab-separated
text formats. These are easily legible to humans and excellent toolchains
exist for working with such files (e.g. Microsoft Excel and TAD can both easily
open a TSV/CSV file). With a proper header, the meanings of
the file's values can be quickly understood. These files are also viewable in the
terminal using the standard unix command `less`.

### Tidy formats are machine readable and representable
Formats should be easy to parse programatically if analysis is to scale beyond 
manual review. Tidy data is inherently easy to parse because of its underlying design.

- When a header is given, variable names and types can be known before the values
are parsed.
- Tables have a predictable structure, with a fixed number of columns. This means that
there is no need to program around observations that do not have a predictable length.
- Tidy formats are easily converted to binary representations by having a fixed number
of elements in each observation. Assuming a predefined header that lists the variables
and their data types, a natural pattern for parsing/emitting data emerges, which we
describe with the pseudocode in Figure 1.

Assume three functions are provided by system libraries:
- write_length(VALUE), which writes the amount of memory needed to store a given instance
of a specific data type (i.e. for a 64-bit integer, 64 bits are written);
- write_field(VALUE), which writes the value of a data type VALUE (i.e. write_field("X") writes
the binary form of the string "X" to output).
- read_length_and_field(INPUT, OBSERVATION, FIELD), which reads a field's length and value
from INPUT and stores it in the relevant FIELD of OBSERVATION.

```
# Persons are the atomic observations of our data
struct Person{
  String NAME
  Integer HEIGHT
  String FAVORITE_CHEESE
  Boolean LACTOSE_INTOLERANT
};

list(Person) people;

function emit_to_binary(Person p){
  write_length(NAME);
  write_field(NAME);
  write_length(HEIGHT);
  write_field(HEIGHT);
  write_length(FAVORITE_CHEESE);
  write_field(FAVORITE_CHEESE);
  write_length(LACTOSE_INTOLERANT);
  write_field(LACTOSE_INTOLERANT);
};

function read_from_binary(INPUT, PERSON){
  read_length_and_field(INPUT, PERSON, NAME);
  read_length_and_field(INPUT, PERSON, NAME);
  read_length_and_field(INPUT, PERSON, NAME);
  read_length_and_field(INPUT, PERSON, NAME);
};

# We can now write a list of Persons:
for p in people:
  emit_to_binary(p);

# And read in a file of persons with the following loop
WHILE open(file) as INPUT and not END_OF_FILE{
  Person p;
  read_from_binary(INPUT, p);
  
  # We could then choose where to store Person p, such as
  # in a list of Persons.
};

```
Figure 1: pseudocode for emitting and reading a simple tidy binary file format.
 

I will save a discussion of emitting/parsing tidy text formats for the reader, but 
I hope y'all would agree that languages such as python support raw parsing of such
data with commands such as `split`. Frameworks such as PANDAS and the R tidyverse
natively support tidy text formats very well.

### Tidy formats are extendable
Extending a tidy format with new observations simply requires adding rows to a file.
Extending the format to include new variables involves adding columns. While this
is somewhat difficult to code (as memory is row-oriented), the tidy verb `join()`
facilitates such operations. A discussion of tidy verbs is beyond the scope of this
document, but I encourage the reader to checkout the awesome [dplyr](https://dplyr.tidyverse.org) package and
the [data wrangling cheatsheet](https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf) for a discussion on the topic.

### Tidy observations are based on atomic entities
An important tenet of tidy data is that each row is atomic. This means 
that every observation is the smallest unit of the data that contains all variables.
If different groupings of observations are required, the tidy verbs encourage aggregation
of these atomic units rather than decomposition of more complex structures. For example,
a VCF file contains rows that represent positions, not genes or chromosomes (groups of positions).
Personally, I think this is easier to understand than rows containing multiple atomic entities.

### Tidy data is language-agnostic
While the tidy principles have gained the most traction in R,
they are not bound to a specific language. The binary serialization example of Figure 1
could be implemented in C++ or python (or R, or Julia, or Erlang, or Scala...). By having
a common format, users can interchange their files with others who might not be familiar
with their language of choice, with no need to worry about the availability of system libraries
(e.g. Boost, ProtocolBuffers, cap'n proto) or language-lockin (e.g. .RData, python pickle).


### Notes on pitfalls of tidy data
- Tidy data, when stored in raw files, are not easily appended to with new data.
Joining data frames in a row-rise manner is not difficult. Appending new columns, however,
is more challenging to program. Luckily, this has been addressed by the tidy verb `join()`
where it is available. The header must also be rewritten when a file is column-appended,
although this can be worked around by using a tidy-translatable external header.


## When designing future genomics formats, remember tidy principles
Many of the most persistent formats in genomics are tidy in nature. I think there are
good reasons for this tendency. I would encourage people designing new formats -
even if they are only for the configuration or output of their own tools - to consider
applying tidy design principles when doing so.

# Tidy genomic data should be paired with tidy phenotype data
I will discuss the utility of pairing tidy genotype data with
tidy phenotype data when performing analysis in another post.
I hope the reader
won't need much convincing to see that easy-to-read, easy-to-append,
atomic data (so long as the genomic and phenotypic data share units)
is easier than working with data that violates these principles.


# Beyond just tidy - what to consider when desigining file formats
In addition to tidyness, I think the following questions are important
to answer when designing a file format:
- Can my format be easily serialized to text?
- In my data human readable? 

These two questions are important for debugging, as it will (almost)
inevitably be useful to be able to examine data manually during the
developmet process. Data should be easily serializable to human-readable
format. An example of a format I find to be easily serialized to text
but not human-readable is XML. JSON slightly improve on this especially
when pretty-printed. Tab-separated files are even more legible.

- Would it be straightforward to serialize my format to binary?

For performance reasons it is useful to be able to serialize to and from
binary without intermediary conversion to strings.

- Does my format scale well with increasing data sizes?
- Are the fields in my format of sufficent bit size to hold realistic values?

Formats should grow linearly or sub-linearly with input size. Where possible,
repeated or constant fields should be reduced to reduce overall file size.
The smallest data type of sufficient size should be used to represent variables
(e.g. a number that is always less than 256 need not be represented by 64 bits).
However, fields should also be large enough to hold all reasonable values. For example,
while many genome sizes are <4 billion bases, and could fit in a 32-bit integer,
a 64-bit integer should be used as genomes larger than this do exist.

# Conclusions
I hope the arguments laid out above assist those who find themselves designing
output in new formats. While standard formats exist (and should be utilized!),
there will likely never be a one-size-fits-all solution. However, even one-off
solutions deserve good design. The tidy principles go a long way toward solving
many of the issues that format designers will encounter. In addition, they help
with other important considerations for large-scale data, such as indexability
and extendability. If you want to talk more about format design please find me
on twitter @erictdawson!



