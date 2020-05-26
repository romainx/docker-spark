Connecting to Spark cluster example through a Jupyter notebook
==============================================================

To run the examples

* Launch the services `docker-compose up`
* Navigate to the notebook URL displayed in the logs, something like `http://127.0.0.1:8888/?token=<token>`
* Spark Web UI will be available at: `http://localhost:8080/`

# Connecting to a Spark Cluster in Standalone Mode

In the following example we are using the Spark master URL `spark://master:7077` that shall be replaced by the URL of the Spark master.

## In a Python Notebook

The **same Python version** need to be installed on the notebook (where the driver is located) and on the workers.

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

R has to be installed on the worker nodes.

```python
library(SparkR)

# Spark session
sc <- sparkR.session("spark://master:7077")

# do something to prove it works
data(iris)
df <- as.DataFrame(iris)
head(filter(df, df$Petal_Width > 0.2))
```

### Sparklyr

```python
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

Jupyter notebooks comes with two ways to interact with Spark in scala: through [Spylon kernel](https://github.com/Valassis-Digital-Media/spylon-kernel) and [Apache Toree kernel](https://toree.apache.org/)

### In a Spylon Kernel Scala Notebook

```python
%%init_spark
# Spark session
launcher.master = "spark://master:7077"
launcher.conf.spark.executor.cores = 1

# do something to prove it works
val rdd = sc.parallelize(0 to 100)
rdd.takeSample(false, 5)
```

### In an Apache Toree Scala Notebook

```python
// Spark session already initialized
// should print the value of --master in the kernel spec
println(sc.master)

// do something to prove it works
val rdd = sc.parallelize(0 to 100)
rdd.sum()
```