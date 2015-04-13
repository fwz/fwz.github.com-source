title: 'Apache Pig in Practice 2'
date: 2014-07-04 10:12:07
tags: [Pig, Hadoop]
---

# Hadoop related tricks and settings

## manage output file count
If you do simply filter based on a huge data, reduce it to a very small data set, and store it onto HDFS, then you might end up with very large amount of file on HDFS, with most of them empty.

In HDFS, namespace(file and directory count) are resources we should care about. saving empty data file is expensive in this perspective.

The root cause of this situation is the MR job only contain mapper job, each mapper will create it's own mapper output. filename pattern `part-m-*` means it's output from mapper,not reducer (or it will looks like `part-r-*`).

if your ouput data are distinct, then one quick but dirty way is to do the DISTINCT operation and set the PARALLEL parameters to assign reducer count.

## Understand useful Hadoop/Pig settings
There are many useful settings that would help you better organize Pig script.

### job.priority
This settings could be set to "high" or "very_high" if you expect this job should be finished in time.

### splitCombination
Mapper count are mostly control by the split size and Hadoop itself. User could not control Mapper count directly, but we could impact it via settings.

if we have 10000 small file (each less than 10M), and supposingly we are going to launch 10000 mapper to handle them, if your splitsize are 10MB or more.

you dont want to waste the mapper resource since file I/O instead of computation are the main overhead, so you try to merge small data into a big trunk of data for each mapper. then you can use to combine file up to 2GB for a mapper:

{% code pig.settings lang:pig %}
SET pig.splitCombination true;
SET pig.maxCombinedSplitSize 2147483648;
{% endcode %}

<!-- more -->

# Using statistical tools

## illustrate

## explain

# Pig Use cases and patterns

* What is the recommended usage of projections to solve specific patterns?
* In which pattern is the usage of scalar projections ideal to access aggregates?
* For which patterns is it not recommended to use COUNT, SUM, and COUNT_STAR?
* How to effectively use sorting in patterns where key distributions are skewed?
* Which patterns are related to the correct usage of spill-able data types?
* When not to use multiple FLATTENS operators, which can result in CROSS on bags?
* What patterns depict the ideal usage of the nested FOREACH method?
* Which patterns to choose for a JOIN operation when one dataset can fit into memory?
  * replicate join helps. It could remove the sort/shuffle stage and all join could be finished in mapper.
* Which patterns to choose for a JOIN operation when one of the relations joined has a key that dominates?
  * skewed join. skewed join computes a histogram of the key space and uses this data to allocate reducers for a given key.
* Which patterns to choose for a JOIN operation when two datasets are already ordered?
  * Merge Join. Think of the merge sort. This also provides a significant performance improvement compared to passing all of the data through unneeded sort and shuffle phases.


# Writing more elegant Pig
## Project-Range Expressions
When there are more than ten columns in your data, writing a GENERATE clause is a little boring.

oroject-range ( .. ) expressions can be used to project a range of columns from input. For example:

```
.a $x : projects columns $0 through $x, inclusive
$x .. : projects columns through end, inclusive
$x .. $y : projects columns through $y, inclusive
```

Project-range can be used in the following statements: FOREACH, JOIN, GROUP, COGROUP, and ORDER BY (also when ORDER BY is used within a nested FOREACH block).


