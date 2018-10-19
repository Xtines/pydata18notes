


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

