---
title: "Apache Spark: Proceed with caution!"
date: 2022-06-15
categories:
  - Blog
tags:
  - Spark
  - Databricks
  - Azure
  - Performance
  - Cloud
  - Big data
  - Distributed Processing
---
What workloads make sense to run in Spark?

I recently got pulled into a project that had been underway for several months.  An agile team of developers had been
building a system that would take 3 flat files as input (.csv format), join data across the files, and output a set
of files in .json format that adhere to the CMS guidelines for price transparency: [Price Transparency](https://github.com/CMSgov/price-transparency-guide)

![Spark Diagram](/assets/images/spark-diagram.svg)

The size of the 3 files is expected to be small (50-100MB each).  However, when joined, the data will quickly grow.  This
data size problem was recognized early and the team chose to implement the processor in Java using Spark (Databricks).

The team had completed testing for the code and had deployed to cloud vendor and all was well.  The processing took some
time, but fulfilled the needs of the organization.  Then came the "real" data from production, and the process would no longer complete!  It
would run for several hours and then throw "out of memory".  The team proceeded to increase the processing resources in
the spark cluster.  When they reached cluster size of 1.5 TB memory and 192 CPUs, and still received out of memory
errors (after running for several days), the team got very worried.  With only a couple of weeks left before contracted
delivery dates, the team decided to try several things:

1. Change RDD to dataframes/datasets - did not help
2. Change dataframes to Spark SQL - did not help
3. Mayday, Mayday!

This is when I was asked if I could help.  I had never touched Spark or Databricks, but I really liked the "idea" of
distributed processing that happens "magically".  Without knowing anything about Spark I speculated that the nodes in
the cluster were having to communicate too much data between each other.  I later learned that Spark calls this "shuffling",
and it should be avoided if possible (it is the costliest operation in Spark).

At this point the Spark cluster was costing the organization tens of thousands of dollars per month, yet could not complete
the work.

With only 2 weeks left, I decided to take a crack at it, but instead of using an expensive Spark cluster, I decided to
write it in plain Java code - which has several advantages: unit tests, developer debuggability, low cost.  After analyzing
the data (with the help of a data SME), I found that the data could be sliced into smaller "buckets" of data, and each
bucket corresponded with one output file.  I knew that I could easily load the files into memory since the files themselves
are relatively small.  The problem is a combinatorial problem.  When you join across the 3 data sets you wind up with a
huge data set (multiple terabytes).  Also notice that what the workload is doing is "combining" data sets and returning
ALL the data.  The workload doesn't involve aggregating the data.

In Java this is a fairly simple problem to solve:

1. Load the files into memory.
2. Hash all 3 data sets wherever a join is required (e.g. index the foreign keys).
3. Take 1 bucket at a time, join one bucket's worth of data
4. Output a json file for that bucket (hint: for extra performance, don't use cpu hogging JSON serializers!).
5. Multithread steps 3-4 for speed.

This type of "partitioning" of the data is what was needed on the Spark side, but the developers thought that they could
throw all the data at Spark and it would figure it out.  It's the equivalent of running a SQL Query that joins 15 tables
and wondering why it takes 3 hours to run the SQL Query.

In the end, we wound up with a plain Java program running on cheap vm hardware (couple hundred dollars per month) and
is able to process the data in an hour or two.  It took a couple of weeks from start to finish rather than the months
that had been spent writing it for Spark and trying to "tune" the application and the cluster.

My opinion is that Spark is great for some applications and teams.  I would think twice about building a "product" on top
of it.  Data science/research work, sure.  But unless you have Spark experts who know how to keep the system tuned
over time as your data changes.  Back to the SQL analogy: most organizations have several DBAs who keep their database
systems running smoothly over time.  To embrace Spark, you need similar experts.  If the organization is not willing/able
to provide those experts...Proceed with caution!


