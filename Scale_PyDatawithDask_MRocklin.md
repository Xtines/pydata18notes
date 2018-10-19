


Scale PyData with Dask
=================================
Matthew Rocklin

Audience level:
Intermediate

### Description
This hands-on tutorial teaches students to scale Pandas and Scikit-Learn on a cluster with Dask. Students will be given on a cloud-hosted cluster running Jupyter notebooks and Kubernetes that allows them to scale out to moderate size. They will learn common workflows, how to understand and track performance, and some of the challenges of parallel computation.

Resources: 
1) 
https://github.com/mrocklin/pydata-nyc-2018-tutorial
2) 
http://examples.dask.org/


### Outline
Introduction (20 min): The size limitations of Pandas and Scikit-Learn. How Dask solves scalability issues, and how integrates into PyData.

Exercise: use Dask with Pandas on a cloud-hosted cluster

Controling Execution (20 min): We learn about laziness, triggering computation, and explicitly persisting data in memory.

At the same time we also learn about how to interpret Dask's diagnostic dashboard, and how our operations affect the state of the cluster.

Scikit-Learn on small data (15 min): We use Scikit-Learn in parallel on small datasets to accelerate hyper-parameter optimization and embarrassingly parallel estimators like RandomForests.

We also learn about parallel profiling to investigate performance

Scikit-Learn on large data (15 min): We use Scikit-Learn-style workflows on larger datasets using Dask-ML and investigate their performance.

Custom Computations (15 min): We use dask.delayed to build custom computations. This gives us a sense for how Dask parallelizes other libraries and how we can use it to paralellize our own work.

Other Applications (30 min): If time allows students will have thirty minutes to explore other notebooks with scalable computations in other topics like image processing, time series analysis, and more.


Workshop notes:


## Investigate laziness and use the `.compute()` method

Note that the `df.passenger_count.sum()` computation did not yet execute.  Dask dataframes are *lazy* by default, so they only evaluate when we tell them to.


There are two ways to trigger computation:

-  `result = result.compute()`: triggers computation and stores the result into local memory as a Pandas object.  

    You should use this with *small* results that will fit into memory.
-  `result = result.persist()`: triggers computation and stores the result into distributed memory, returning another Dask dataframe object.  

    You should use this with *large* results that you want to stage in distributed memory for repeated computation. This stores the results in "distributed memory". 
    


## Persist data in memory

When we started this notebook we ran the following lines to create our dataframe.

```python
df = dd.read_csv('gcs://anaconda-public-data/nyc-taxi/csv/2015/yellow_*.csv', 
                 parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime'])
df = df.persist()
```

In particular, we called `df = df.persist()` to load all of the CSV data into distributed memory.  Having this data in memory made our subsequent computations fast.  

In this section we're going to reset our cluster and run the same computations, but without persisting our data in memory.  What happens to our computation times?  Why?


#### *Exercise*: What did our workers spend their time doing?

To answer this question look at the Task Stream dashboard plot.  It will tell you the activity on each core of your cluster (y-axis) over time (x-axis).  You can hover over each rectangle of this plot to determine what kind of task it was.  What kinds of tasks are most common and take up the most time?

*Extra*: if you're ahead of the group you might also want to look at the Profile dashboard plot.  You can access this by selecting the orange Dask icon on the left side of your JupyterLab page.  The profile plot is an interactive [Flame graph](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html)



## Dask DataFrame Design

We briefly discuss the design of Dask dataframes.  Then we follow this section with exercises that dive into this design.

<img src="http://docs.dask.org/en/latest/_images/dask-dataframe.svg"
     width="50%">
     
Dask dataframes are composed of many *partitions*, split along the index.  Each partition is a Pandas dataframe or Series.  You can see the number of partitions in the rendering of a Dask Dataframe.

```df

And the type of each partition using the `map_partitions` method.

```df.map_partitions(type).compute()

### Divisions and the Index

Just like Pandas, Dask Dataframe has an *index*, a special column that indexes the rows of our dataframe.  In Dask this index has an additional purpose, it serves as a sorted partitioning of our data.  This makes some algorithms more efficient.  In this section, we'll sort our data by time and dive into the index a bit more deeply.

First, notice that our index is not particularly informative.  This is common when you load a dataset from CSV data, which generally doesn't store index or sorting information.

Lets set a new index to be the pickup time.  Sorting in parallel is hard, so this is an expensive operation.

```df2 = df.set_index('tpep_pickup_datetime').persist() # sorting/indexing by time of pickup


Q & A: We are writing data into pandas dataframes, not changing csv files. 
Note: see the plot Dask Memory Use under the left-hand column of Dask buttons - if columns for Bytes stored show a lot of orange versus blue, then it means we are running out of RAM. 


Our dataframe is split into roughly as many partitions as before, but now we know the time range of each partition.  Internally, the divisions between partitions is stored in the divisions attribute.

```df2.divisions


### Questions: 

What took up the most time in the operation above?
A: 

What colors are most prominent in the task stream plot?
A:

When you hover over some of these bars, what do they say?
A: 

### Fast operations along the index

Having a sorted dataframe allows for fast operations, like random access lookup and timeseries operations.

```python
df2.loc['2015-05-05'].compute()  # pick out one day of data

df2.passenger_count.resample('1h').mean().compute().plot()
```

