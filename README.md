Connecting to Spark cluster example through a Jupyter notebook
==============================================================

To run the examples

* Rebuild the image (I've made some changes to adapt the version of tools) `docker-compose build`
* Launch the services `docker-compose up`
* Navigate to the notebook URL displayed in the logs, something like `http://127.0.0.1:8888/?token=<token>`
* Spark Web UI will be available at: `http://localhost:8080/`

# Connecting to a Spark Cluster in Standalone Mode

Connection to Spark Cluster on Standalone Mode requires the following set of steps:

0. Verify that the docker image (check the Dockerfile) and the Spark Cluster which is being
   deployed, run the same version of Spark.
1. [Deploy Spark in Standalone Mode](http://spark.apache.org/docs/latest/spark-standalone.html).
2. Run the Docker container with `--net=host` in a location that is network addressable by all of
   your Spark workers. (This is a [Spark networking
   requirement](http://spark.apache.org/docs/latest/cluster-overview.html#components).)
    * NOTE: When using `--net=host`, you must also use the flags `--pid=host -e
      TINI_SUBREAPER=true`. See https://github.com/jupyter/docker-stacks/issues/64 for details.

Note: In the following examples we are using the Spark master URL `spark://master:7077` that shall be replaced by the URL of the Spark master.

## In a Python Notebook

The **same Python version** need to be installed on the notebook (where the driver is located) and on the Spark workers.

```python
from pyspark.sql import SparkSession

# Spark session & context
spark = SparkSession.builder.master('spark://master:7077').getOrCreate()
sc = spark.sparkContext

# do something to prove it works
rdd = sc.parallelize(range(100))
rdd.sumApprox(3)
```

## In a R Notebook

Jupyter notebooks comes with two libraries permitting to interact with Spark: [SparkR](https://spark.apache.org/docs/latest/sparkr.html) and [sparklyr](https://spark.rstudio.com/).

### SparkR

R has to be installed on the Spark worker nodes.

```R
library(SparkR)

# Spark session
sc <- sparkR.session("spark://master:7077")

# do something to prove it works
data(iris)
df <- as.DataFrame(iris)
head(filter(df, df$Petal_Width > 0.2))
```

### Sparklyr

```R
library(sparklyr)
library(dplyr)

# Spark session
sc <- spark_connect(master = "spark://master:7077")

# do something to prove it works
iris_tbl <- copy_to(sc, iris)
iris_tbl %>% 
    filter(Petal_Width > 0.2) %>%
    head()
```

## In Scala

Jupyter notebooks comes with two ways to interact with Spark in scala: through [Spylon kernel](https://github.com/Valassis-Digital-Media/spylon-kernel) and [Apache Toree kernel](https://toree.apache.org/).

### In a Spylon Kernel Scala Notebook

```scala
%%init_spark
# Spark session
launcher.master = "spark://master:7077"
launcher.conf.spark.executor.cores = 1
```

Now run Scala code that uses the initialized `SparkContext` in `sc`

```scala
// do something to prove it works
val rdd = sc.parallelize(0 to 100)
rdd.takeSample(false, 5)
```

### In an Apache Toree Scala Notebook

The Apache Toree kernel automatically creates a `SparkContext` when it starts based on configuration information from its command line arguments and environment variables. You can pass information about your cluster via the `SPARK_OPTS` environment variable when you spawn a container.

For instance, to pass information about a standalone Spark master, you could start the container like so:

```bash
docker run -d -p 8888:8888 -e SPARK_OPTS='--master=spark://master:7077' \
       jupyter/all-spark-notebook
```

Note that this is the same information expressed in a notebook in the Python case above. Once the kernel spec has your cluster information, you can test your cluster in an Apache Toree notebook like so:

```scala
// Spark session already initialized
// should print the value of --master in the kernel spec
println(sc.master)

// do something to prove it works
val rdd = sc.parallelize(0 to 100)
rdd.sum()
```