title: 'Apache Pig in Practice 1'
date: 2014-07-04 01:27:43
tags: [Pig, Hadoop]
categories: 
- Engineering 
- Big Data
---

![](http://wenzhong.qiniudn.com/pig1.png)
I write many pig script in the past few months and have explored some tricks with my buddies. hopes it could help someone.

Let's focus on some interesting topics in this first article and get prepared for the later Pig rush.

## IDE & Environment
### Vim

I use Vim to write most script language and those are my favourite plugins to write Pig:
* [Pig Syntax Highlight](http://www.vim.org/scripts/script.php?script_id=2186). Latest update on Jun 2014, Pig 0.12 supported.
* [You complete me](https://github.com/Valloric/YouCompleteMe). Best auto-complete plugins ever. If you don't use a MAC, [Supertab](https://github.com/ervandew/supertab) is also a reasonable choice.
* [Tabularize](https://github.com/godlygeek/tabular) Align and keep cleaness of the Pig codelet. Most common usage is `:Tab/AS` to align `FOREACH ... GENERATE` clause.

To improve debug efficiency, I like to run pig with short cut. Here are my simple approach: add the following in `.vimrc` for quick run with `F5`
{% codeblock %}
map <F5> :call Compile_Run()<CR>
function Compile_Run()
    if &filetype=="coffee"
        :w
        !coffee % 2>&1
    elseif &filetype=="cpp"
        :w
        !g++ -g -o %< %; ./%<
    elseif &filetype=="python"
        :w
        !python %
    elseif &filetype=="pig"
        :w
        !./run_pig.sh %
{% endcodeblock %}

revise `run_pig.sh` as you like. General idea is reduce redundant work and typo.

<!-- more -->

Basic form would be:
{% codeblock %}
pig -x local $1
{% endcodeblock %}

or with default local debug settings
{% codeblock %}
intput=./input.txt
output=./output
rm -rf ${output?}
pig -x local -Dinput=${input} -Doutput=${output} $1
{% endcodeblock %}

## Modulize your Pig code using Marco
### Under standing Marco
We have mix feelings with Marco, still I love it better.

Marco could help organize and reuse your code. Marco in Pig is quite like Marco in C -- they do substitution. Think about you want to do the same series of operation with 3 dataset... Try refactor it into a marco, you will absolutely thank your mercy later.

Understanding the `$` sign is important when using Marco. `$` decorate those variable to be replaced

{% codeblock number1.txt %}
1
2
3
4
5
{% endcodeblock %}

{% codeblock filter.marco %}
DEFINE filter_small_number (events, threshold) RETURNS filtered_events {
    $filtered_events = FILTER $events BY a > $threshold;
};
{% endcodeblock %}

{% codeblock %}
IMPORT 'filter.marco';

events = LOAD './data/number1.txt' AS (a:int);

big_number = filter_small_number(events, 2);

DUMP big_number;
{% endcodeblock %}

{% codeblock %}
(3)
(4)
(5)
{% endcodeblock %}

Easy. 

However, you'd better not change input with in a Marco. they are just substitution, every change in input variables are global 

{% codeblock %}
IMPORT 'filter.marco';

events = LOAD './data/number1.txt' AS (a:int);

big_number = filter_small_number(events, 2);

DUMP big_number;
DUMP events;
{% endcodeblock %}

{% codeblock %}
DEFINE filter_small_number (events, threshold) RETURNS filtered_events {
    $filtered_events = FILTER $events BY a > $threshold;
    $events = FILTER $events BY a == 4;
};
{% endcodeblock %}
{% codeblock %}
big_number
(3)
(4)
(5)

events
(4)
{% endcodeblock %}

You can also return multiple data set in Marco
{% codeblock %}
DEFINE split_events (events, threshold) RETURNS big, small {
    $big = FILTER $events BY a >= $threshold;
    $small = FILTER $events BY a < $threshold;
};

events = LOAD './data/number1.txt' AS (a:int);

big_num, small_num = split_events(events, 3);

DUMP big_num;
DUMP small_num;

{% endcodeblock %}

{% codeblock %}
big
(3)
(4)
(5)
small
(1)
(2)
{% endcodeblock %}

### What's not so cool
One reason we love Marco less is that after marco is plugined into Pig then error message become a little difficult to read and resolve root cause, because line number would be reflecting the reassembled Pig scripts. However, it's still a great tool and a must have skill to use Pig.

# INPUT and OUTPUT
## INPUT
* You can almost load everything, HDFS / Avro / Protobuf / Hive / Elastic Search / MongoDB.

## OUTPUT
* Of course, there are Storage Function in Pair for above persistency / serialization tools.
* MultiStorage could help you store data hierarchily, which mean you could partition result when storing, absolutly must-know features.

# Third party Pig library
* [piggybank](https://cwiki.apache.org/confluence/display/PIG/PiggyBank)
* [DataFu](http://data.linkedin.com/opensource/datafu) from LinkedIn
* [ElephantBird](https://github.com/twitter/elephant-bird/) from twitter
* [Hcatalog](http://hortonworks.com/hadoop/hcatalog/)

# UDF
Once you know you could use Python/Ruby/JS to write UDF, I suppose nobody will try to use JAVA for common cases.
[Python UDF](http://pig.apache.org/docs/r0.9.1/udf.html#python-udfs)

# Unit test
## PigUnit
Write UT to be a good man. Of course, Pig could and should be unit-tested. The [PigUnit](http://pig.apache.org/docs/r0.8.1/pigunit.html) backbone are supported in Java. However docs are limited and you might run into many troubles.

## Unit test a python UDF
when using native `unittest` packages to test the python script，`outputSchema` will complains. One way is to add Pig support in Python script, the other one is to disable the outputSchema notation. Here we should the second tricks, put this codelet at the top of the UDF.

{% codeblock %}
if __name__ != '__lib__': 
    def outputSchema(dont_care): 
        def wrapper(func): 
            def inner(*args, **kwargs): 
                return func(*args, **kwargs) 
        return inner 
    return wrapper 
{% endcodeblock %}

This block is intended to test the UDF with the outputSchema notation. The `__name__` will be marked as 'lib' when script is call by Pig. So it will not take effect when the script is running as Pig UDF. 

# References
[Comparing Pig Latin and SQL for Constructing Data Processing Pipelines](https://developer.yahoo.com/blogs/hadoop/comparing-pig-latin-sql-constructing-data-processing-pipelines-444.html) By Alan Gates, Pig Architect in Yahoo.
[Programming Pig](http://shop.oreilly.com/product/0636920018087.do) also by Alan Gates.
[Pig Design Pattern](http://www.packtpub.com/pig-design-patterns/book)

