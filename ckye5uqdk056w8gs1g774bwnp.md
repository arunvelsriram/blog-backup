## Adding Custom JARs to Spark Jobs on AWS EMR

I was working on a Spark job to process data and load it into a Database. The database provides support for reading/writing via Spark using a connector available as a JAR. I need to run this Spark job on AWS EMR, so the connector JAR should be available to the Spark cluster.

The immediate solution that came to mind is to use EMR bootstrap action to push the JAR to the nodes in the cluster and make it available to the Spark cluster by overriding the Spark configuration. The steps would be:

1. Place the JAR file in S3
2. Create a script to copy the JAR to the node. This script will be configured as a bootstrap action in EMR, so it runs on all the nodes after provisioning the nodes
3. Override Spark configurations to include the JAR in the classpath

It is explained in detail in this [SO answer](https://stackoverflow.com/a/56667997/4593654).

There are a few drawbacks with this approach:

* `spark.driver.extraClassPath` and `spark.executor.extraClassPath` configuration includes paths that are required for EMR to function properly. So we need to ensure that the paths are in sync with those used by EMR. For instance, if you planning to move to a higher EMR version, then these paths might change.
* The bootstrap action and Spark configuration override happen during the creation of the EMR. So if something is missed then we will have to re-create the EMR. (Otherwise, it's tedious to copy the JAR to all nodes manually and cumbersome to set `spark.driver.extraClassPath` and `spark.executor.extraClassPath` as part of `spark-submit`)
* Bootstrap actions incur some extra time during the creation of EMR. However, it's negligible.

### Alternative approach

There is an alternative approach without using bootstrap action and overriding Spark configuration. 

`spark-submit` has the `--jars` option for sending JARs to the nodes in the cluster and making them available in the classpath of all executors. 

From the [`spark-submit` documentation](https://spark.apache.org/docs/latest/submitting-applications.html#advanced-dependency-management):

> When using spark-submit, the application jar along with any jars included with the --jars option will be automatically transferred to the cluster. 

Since EMR can access S3 like a filesystem we can pass the S3 path itself in the `--jars` option. 

For example,

```bash
spark-submit --deploy-mode cluster --jars s3://my-bucket/my-jars/db-connector.jar ...
```

```bash
spark-submit --deploy-mode cluster --jars s3://my-bucket/my-jars/*.jar ...
```

In this way, we can make any JAR (in S3) available to the jobs as part of job submission and avoid dependency on additional steps at cluster creation.
