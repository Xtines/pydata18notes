


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
