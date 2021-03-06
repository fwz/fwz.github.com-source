title: 'Apache Hive in Practice 1'
date: 2014-07-08 13:55:19
tags: [Hive]
---

Here are some useful settings when using Hive
## Hadoop related Settings
{% code lang:sql %}
-- define queue name
set mapred.job.queue.name = your_queue;

-- set Hive into strice mode, so query without "where" will not be executed.
set hive.mapred.mode=strict;

## Interact with Bash
${env:HOME} indicate the Home env variable

{% endcode %}

## INPUT / OUTPUT
### use INSERT OVERWRITE to redirect selection to HDFS/local
{% code lang:sql %}
-- load to HDFS
INSERT OVERWRITE DIRECTORY '/path/to/output/dir' SELECT * FROM table WHERE id > 100;
-- load to local
INSERT OVERWRITE LOCAL DIRECTORY '/path/to/output/dir' SELECT * FROM table WHERE id > 100;
{% endcode %}

For more explanation about `INSERT OVERWRITE`, [this answer](http://stackoverflow.com/questions/18129581/how-do-i-output-the-results-of-a-hiveql-query-to-csv/18142190#18142190) might helps.

## Hive UDF
[Python UDF](http://blog.spryinc.com/2013/09/a-guide-to-user-defined-functions-in.html)
