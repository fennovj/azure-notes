# Databricks

<https://docs.microsoft.com/en-us/azure/databricks/>

Note that this exam is not exactly about actually writing notebooks in Spark, but more about using Databricks in your design.

## Clusters

Clusters are sets of computation resources and configurations to run data stuff on. You can run them in notebooks or jobs. There are 'interactive clusters' and 'automated clusters'.

- Interactive clusters can be ... interacted with! With notebooks. You create them using the UI, or the Rest API. You can manually start/stop them.
- Automated clusters - Created by the Databricks Job Scheduler. Cannot be manually started. Cannot be restarted after they are done. They a bit faster and more robust.

### Configuring the cluster

When you create a cluster, you can choose:

- Cluster policy - the policy under which to create the cluster. Used to limit resources. By default, there is only the 'free form' policy.
- Cluster mode - Standard vs High Concurrency.
    - Standard are recommended for a single user. They also Auto-terminate after 120 minutes by default.
    - High concurrency is better at sharing resources between multiple users. It has:
        - preemption - preempt users from taking all resources at once. It ensures all users get a fair share.
        - fault isolation - When one user starts writing buggy code, it doesn't affect anyone else.

High concurrency does not support Scala (because of fault isolation, the user code runs in separate processes, which is not possible in Scala). Also, they do not auto-terminate by default.

- Pools: You can attach a cluster to a predefined pool of idle instances. Basically, this means clusters 'share' instances with each other. When one cluster is idle, other clusters can take those instances to start quicker.

- Databricks Runtime
- Python version: this one and the one above speak for themselves - usually pick the most recent one, unless you have some compatibility issues. From Databricks 6.0 onwards, Python 2 is unsupported. You can only have one databricks/python version per cluster.

- Cluster node Type: This defines the number of cores/memory. A cluster consists of one driver and multiple worker nodes. By default, the driver is the same as the worker, but you can assign the driver more resources if you plan to `collect()` a lot of data.
    - Also, there are GPU-instance types which are good for Image anaylsis and stuff. They come with CUDA, cuDNN and NCCL by default.

- Cluster size/autoscaling - define the number of workers. Either fixed or a minimum/maximum. There are two types of autoscaling:
    - Optimized scaling - used in Automated clusters, or in Azure Databricks Premium Plan.
        - Scales from min to max in 2 steps.
        - Can scale down even if cluster is not idle by checking utilization rate
    - Standard scaling
        - Starts with adding 8 nodes, then scales exponentially.
        - Scales down only it sees the cluster is completely idle for a certain percentage during the last 10 minutes.

Basically, Standard is fine for an Standard single-user interactive cluster. When you have MANY nodes, or use high-concurrency, optimized may be worth it. (but needs premium plan)

- Environment variables, init scripts - for code stuff.

## Delta Lake

<https://delta.io/> - An open source stoarge layer on top of an existing Azure Data Lake, that uses DataBricks to bring more reliability. Provides ACID transactions, and unifies batch and streaming. Runs on top of existing Data Lake. Not really a standard solution, but now you know what it is.

## Authentication

As you may have noticed, databricks requires an additional login when accessing the noteboks. Basically, you need a Personal Access Token to use Databricks (other than just managing it in Azure) If you use the UI, this all goes automatically, but if you want to use Databricks without logging into the UI, you need a PAT. For example, for logging into a databricks shell on a different machine.