title: 'Evolution of Metric System Architecture'
date: 2015-03-26 14:31:12
tags:
---

In Yahoo!, about 70% of my time is spent on building data platform for Yahoo's recommendation system. After two years of continuous iteration, we currently have a data system with ability to:

* understand recommendation system with reports from different aspects and A/B test visualization tools
* identify metric abnormal / data pipeline failure
* collect user feedback data to improve system online in short cycle
* make it easy for PM/Dev/Scientist to play with data

During the iteration of the recommendation system, we follow and forecast what would be the actual use cases to understand or to improve the system itself. So for different stage, we focus on different aspect and use different tools / techniques to solve problems.

Stage 1: Prototype

At the early stage, recom system is just built from scratch. As data team, our top priority is to use data to identify whether the recom system work as expect, which means we care about our services more than our actual user at this stage. And the above graph show a scraper pattern.
We build a scraper to send multiple mock requests
Analysis the statistical result from response of mock requests
Write the statistical result to a local MySQL.
The front end of data product call MySQL directly to get data and render

Stage 2: Scalable Core

Now we start to care more about user, we want to know how user in different segments interact with our business product. Thus we collect a user behaviour log (which is not produced from our side), and compute metrics like Retention Rate and CTR(click through rate) to measure user’s engagement.

Storage
Starting from this point, some interesting changes happen. First of all, Hadoop become the computation platform since:
user behaviour logs are located on HDFS
they are commonly very large (TB level for daily log)
the best playground for such data by then is also Hadoop.

Then we start to think about the storage. Should we pull the aggregated data to non-Hadoop environment and put them to MySQL?

Previous lessons is asking us not to do that, because of the questions we are going to answer just changed:
How does user in different segments interact with our business product

Here different segments means a lot, and a lot. For example, the segment might based on age of the user, location of the user, login status of the user, the A/B test id, etc. the combination of dimensions will soon explode to a number you might not imagine before (In our projects, we have two layer of A/B test). So here we have to solve the problem of scalability. We have mainly two choices:
Use a MySQL cluster
Use other scalable Storage

And we finally choose HBase due to several reasons:
Friendly integration with Data Pipeline on HDFS
scalable as a KV database
Good (enough) latency on range query
No relation query use case in predictable future
Easy to adapt new combination of dimensions

Data Pipeline
Most data pipelines are actually taking ETL operation against data. A more reasonable and natural choice on Hadoop is Pig instead of Hive. See more from Alan Gates’ summary. (https://developer.yahoo.com/blogs/hadoop/comparing-pig-latin-sql-constructing-data-processing-pipelines-444.html)
With the support of UDF (which could be written in Python/Ruby/JS), constructing an ETL data pipeline is effective. 

Scheduler
OK. the only thing left here is job scheduler. If we want to generate regular daily report, the simplest way is to start a cronjob to run regularly. Unfortunately a pipeline might have external dependencies. In our case, we have to wait the userlog data before we could start. Sure we can write a loop in the crontab to wait, or trigger this job with a reasonable delay? but how long should we wait? How could I start my pipeline once the dependency is ready? what if we have multiple dependencies?

Apache Oozie came out and it save us tons of efforts. On many cases, we use it as a data-trigger, when all dependency data is ready, trigger a series of jobs.

Data Product Serving
to secure our backend data and as a more regularized way to manage data, we implement a Serving layer using Tomcat. Since most operation happened in data products are READ operation, we mainly focus on RPS of Serving. We have achieved 180 QPS for most of our common data query.

Frontend
We made our FE more user-friendly by leveraging Bootstrap (For page Layout/CSS), Highchart (For the charting module). Node.js is used to communicate with Serving.



Stage 3: Expand Ecosystem
As the number of reporting metrics increase, some other problems / requirements emerged. 

Abnormal Detection

As number of pipelines increases, probability of a broken pipeline also increases.

First of all, data dependencies matters. Here data dependencies is actually a topology dependency, which mean pipeline A need pipeline B’s output as input. thus A depends on B. When pipeline B breaks or delays, pipeline A will also get blocked. Thus we get metrics delayed. For time sensitive metrics, detecting such issues become more important.

Secondly, we’d like to detect abnormal value in metrics. For example, if Ads CTR become 0 after a release, we definitely want to get some notifications immediately instead of knowing revenue of the past week are blank one week later.

So a detection system is implemented. For metric delay issue, we label each pipeline with expected running time, then detection system system build data dependencies graph according to configs and source code of data pipeline. Then we scan the intermediate data on HDFS to see whether each pipeline have produce output. 

In this way, we have enough information to get status of each pipelines. If delay occurs, we will be able to know why.

To detect metric abnormally, we also define our metric detectors, which use configurable parameters to control detect algorithm. With a series of metrics data as input, could apply numerous algorithm to detect metric abnormal.

One surprise finding is that, using a fixed threshold is one of our best friend if quality of source data could not be guaranteed. (To be illustrated)

Ad-hoc Query 

Now we have more and more data, people wish we could help answer their questions using our existing data, QUICKLY. With our previous architecture, such requirement / enquiry often require us to build and launch a new data pipeline (even not regularly), because the enquiry  could not be computed from our existing metrics. It is not cost-effective for following reasons: 1. it take human resources to finish such a task. 2. communication cost is un-imaginable big especially for people in remote office. 

To make good use of data and reduce engineering efforts, we decided to build a data-warehouse: Store atomic data for both metric computation and ad-hoc query. Since the enquiry are mostly relational (Such as “what are the top 100 publishers (get most clicks) among mid-age user?”), we use Hive(https://hive.apache.org/) to build our data warehouse. With Support of HCatalog(http://hortonworks.com/hadoop/hcatalog/), Pig and Other MapReduce could easily access / operate data in Hive.

Also an user interface is necessary for a data warehouse, currently we are using our own UI, and will switch to Hue(http://gethue.com/) in near future.

Logging

Now also think about the essence of our system, data. As we could see, we collect only the user feedback data from the UI layer. For a recommendation system, logging runtime intermediate results will be helpful to build ML model when combined with the user feedback data. But it’s not clever to return these results to front-end (User UI) because it will increase latency so the data should be logged at server side and send to somewhere (in our case Hadoop). Also the Server has it’s own duty, so this logging pipeline should be as light-weight as possible. With some performance test, 

We finally choose Flume (http://flume.apache.org/) as our logging framework. For each server, a Flume client is added and it help we send data to HDFS via memory & sockets. No Disk I/O is needed.

Once we have both logs from server and user, we are able to join them together and get labeled training data, thus we could run ML training pipelines to generate new model for server.

Also, a handy way to check error log is helpful for debugging issues. We deploy a Splunk(www.splunk.com/) instance to collect system log, and provide an real-time search UI to the team member.




## Lesson Learnt

Simplify the metrics. Complicated metrics make itself hard to understand, hard to compare, and error-prone in the computation stage. Also, try to limited the number of metrics to make important decision, we could seldom make a decision when some numbers are encouraging us to move forward when some are saying NO if there are too many metrics.

Manage metrics life cycles. It takes resource to maintain a metrics. When a metrics is no more used, retire it. It’s much easier to retire a metrics than to deprecate existing code: just stop the pipeline. With version control (or even better continuous integration) support, we could restart it very soon in with a simple click.

One way to identify which metric should be retired is “Don’t ask”, trust numbers. People are afraid of losing existing property. So when ask “May I retire these metrics”, we get “please don’t” for most cases. Write a script, parse the log to identify how many times the metrics have been reported, determine a threshold to filter some candidates, stop the pipeline with or without enquiry. Do it monthly.

I build, break, and fix data products in the past 2 years. This is a summary of my understanding of building the whole stuffs, and what kind of tool should be plugged as components.

The most simplest architecture might look like this
Let me brief a simple

# Lesson Learned
* Simplify the metrics. Complicated metrics make itself hard to understand, hard to compare, and error-prone in the computation stage. Also, try to limited the number of metrics to make important decision, we could seldom make a decision when some number is encouraging us to move forward when some saying NO if there is too many metrics.

## Define Project Goal Clearly

* Who will be our major user? Is it an internal tool? Or is it for business partner or real user? The answer greatly impact our future decision of design and resources allocation.
* How long this projects suppose to support?

## Understand and Clean data

* Equip Data expert
* Manage dimension explosion








* Always ask questions.
