


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
And the type of each partition using the `map_partitions` method.

```
df.map_partitions(type).compute()
```

### Divisions and the Index

Just like Pandas, Dask Dataframe has an *index*, a special column that indexes the rows of our dataframe.  In Dask this index has an additional purpose, it serves as a sorted partitioning of our data.  This makes some algorithms more efficient.  In this section, we'll sort our data by time and dive into the index a bit more deeply.

First, notice that our index is not particularly informative.  This is common when you load a dataset from CSV data, which generally doesn't store index or sorting information.

Lets set a new index to be the pickup time.  Sorting in parallel is hard, so this is an expensive operation.

```python
#Sorting/indexing by time of pickup
df2 = df.set_index('tpep_pickup_datetime').persist() 
```


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


# Delayed Dask objects


```python
results = []

for x in data:
    y = dask.delayed(inc)(x)
    results.append(y)

total = dask.delayed(sum)(results)
print("Before computing:", total)  # Let's see what type of thing total is
result = total.compute()
print("After computing :", result)  # After it's computed
```

## Exercise: Parallelizing a for-loop code with control flow

Often we want to delay only *some* functions, running a few of them immediately.  This is especially helpful when those functions are fast and help us to determine what other slower functions we should call.  This decision, to delay or not to delay, is usually where we need to be thoughtful when using `dask.delayed`.

In the example below we iterate through a list of inputs.  If that input is even then we want to call `inc`.  If the input is odd then we want to call `double`.  This `is_even` decision to call `inc` or `double` has to be made immediately (not lazily) in order for our graph-building Python code to proceed.

```python
# %load solutions/01-delayed-control-flow.py
results = []
for x in data:
    if is_even(x):  # even
        y = dask.delayed(double)(x)
    else:          # odd
        y = dask.delayed(inc)(x)
    results.append(y)
    

total = dask.delayed(sum)(results)
```

```total.visualize()```


### Some questions to consider:

-  What are other examples of control flow where we can't use delayed?
-  What would have happened if we had delayed the evaluation of `is_even(x)` in the example above?
-  What are your thoughts on delaying `sum`?  This function is both computational but also fast to run.


## Rebuild Dataframe algorithms manually - if we didn't have Dask how would we parallel compute? 

In the last notebook we used Dask Dataframe to load CSV data from the cloud and then perform some basic analyses.  In these examples Dask dataframe automatically built our parallel algorithms for us.

In this section we'll do that same work, but now we'll use Dask delayed to construct these algorithms manually.  In practice you don't have to do this because Dask dataframe already exists, but doing it once, manually can help you understand both how Dask dataframe works, and how to parallelize your own code.

To make things a bit faster we've also decided to store data in the Parquet format.  We'll use Dask delayed along with Arrow to read this data in many small parts, convert those parts to Pandas dataframes, and then do a groupby-aggregation.


### Build aggregation piece by piece

We want to compute the following operation on all of our data:

```python
import dask.dataframe as dd
df = dd.read_parquet('gcs://anaconda-public-data/nyc-taxi/nyc.parquet/part.*.parquet')
df.passenger_count.mean().compute()
```

This actually works, but lets pretend that it didn't, and lets build this up, chunk by chunk, file by file.

(Dask would do this for you if using Dask)
We do this for you sequentially below with a for loop:

```python
sums = []
counts = []

def filename_to_dataframe(fn):
    gcs = gcsfs.GCSFileSystem()

    with gcs.open(fn) as f:
        pf = pq.ParquetFile(f)  # Arrow ParquetFile
        table = pf.read()       # Arrow Table
        df = table.to_pandas()  # Pandas DataFrame

    return df


for fn in filenames[:3]:

    # Read in parquet file to Pandas Dataframe
    df = filename_to_dataframe(fn)
    
    # Groupby origin airport
    total = df.passenger_count.sum()
    
    # Number of flights by origin
    count = df.passenger_count.count()
    
    # Save the intermediates
    sums.append(total)
    counts.append(count)

# Combine intermediates to get total mean-delay-per-origin
total_sums = sum(sums)
total_counts = sum(counts)
mean = total_sums / total_counts
mean
```

### Parallelize the code above

Use `dask.delayed` to parallelize the code above.  Some extra things you will need to know.

1.  Methods and attribute access on delayed objects work automatically, so if you have a delayed object you can perform normal arithmetic, slicing, and method calls on it and it will produce the correct delayed calls.

    ```python
    x = dask.delayed(np.arange)(10)
    y = (x + 1)[::2].sum()  # everything here was delayed
    ```

So your goal is to parallelize the code above (which has been copied below) using `dask.delayed`.  You may also want to visualize a bit of the computation to see if you're doing it correctly.


```%load solutions/01-delayed-dataframe.py


# Q & A

## Q: how to run on local computer instead of on cloud? 
```pip install dask

### Single Machine: dask.distributed
The dask.distributed scheduler works well on a single machine. It is sometimes preferred over the default scheduler for the following reasons:

It provides access to asynchronous API, notably Futures
It provides a diagnostic dashboard that can provide valuable insight on performance and progress
It handles data locality with more sophistication, and so can be more efficient than the multiprocessing scheduler on workloads that require multiple processes
You can create a dask.distributed scheduler by importing and creating a Client with no arguments. This overrides whatever default was previously set.

```
from dask.distributed import Client
client = Client()
```
This will create a Dash dashboard


### Running Dask on the cloud

Highly recommend taking Jupyter's documentation, takes about an hour to get Dask running. 

https://docs.dask.org/en/latest/setup/cloud.html



