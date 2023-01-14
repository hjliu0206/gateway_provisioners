# Hadoop YARN deployments

To leverage the full distributed capabilities of Gateway Provisioners, there is a need to
provide additional configuration options in a cluster deployment.

Steps required to complete deployment on a Hadoop YARN cluster are:

1. [Install Gateway Provisioners](installing-gp.md) on the primary node of the Hadoop YARN cluster
   where the host server is located. Note, this location is not a hard-requirement, but recommended.
   If installed remotely, some extra configuration will be necessary relative to the Hadoop configuration.
2. [Install the desired kernels](installing-kernels.md)
3. Install and configure the server and desired kernel specifications (see below)
4. Launch the server

The distributed capabilities are currently based on an Apache Spark cluster utilizing Hadoop
YARN as the resource manager and thus require the following environment variables to be set
to facilitate the integration between Apache Spark and Hadoop YARN components:

- `SPARK_HOME` must point to the Apache Spark installation path

```
SPARK_HOME:/usr/hdp/current/spark2-client  # For HDP distribution
```

- GP_YARN_ENDPOINT: Must point to the YARN resource manager endpoint if remote from YARN cluster

```
GP_YARN_ENDPOINT=http://${YARN_RESOURCE_MANAGER_FQDN}:8088/ws/v1/cluster
```

```{note}
If the server is using an applicable `HADOOP_CONF_DIR` that contains a valid `yarn-site.xml` file,
then this config value can remain unset (default = None) and the YARN client library will locate
the appropriate resource manager from the configuration.  This is also true in cases where the
YARN cluster is configured for high availability.
```

If server is remote from the YARN cluster (i.e., no `HADOOP_CONF_DIR`) and the YARN cluster is
configured for high availability, then the alternate endpoint should also be specified...

```
GP_ALT_YARN_ENDPOINT=http://${ALT_YARN_RESOURCE_MANAGER_FQDN}:8088/ws/v1/cluster #Common to YARN deployment
```

## Configuring Kernels for YARN Cluster mode

For each supported kernel (IPyKernel for Python, Apache Toree for Scala, and IRKernel for R), we
have provided sample kernel configurations and launchers as assets associated with each [Enterprise Gateway release](https://github.com/jupyter-server/enterprise_gateway/releases). For Hadoop YARN configurations, you can access those specific kernel specifications within the `jupyter_enterprise_gateway_kernelspecs_yarn-VERSION.tar.gz` file. (Replace `VERSION` with the desired release number.)

FIXME - add example commands for creating kernel specs

```{tip}
We recommend installing kernel specifications into a shared folder like
`/usr/local/share/jupyter/kernels`.  This is the location in which they reside within
container images and where many of the document references assume they'll be located.
```

### Python Kernel (IPython kernel)

Considering we would like to enable the IPython kernel to run on YARN Cluster and Client mode
we would have to copy the sample configuration folder **spark_python_yarn_cluster** to where the
Jupyter kernels are installed (e.g. jupyter kernelspec list)

For more information about the IPython kernel, please visit the [IPython kernel](https://ipython.readthedocs.io/en/stable/) page.

### Scala Kernel (Apache Toree)

Considering we would like to enable the Scala Kernel to run on YARN Cluster and Client mode
we would have to copy the sample configuration folder **spark_scala_yarn_cluster** to where the
Jupyter kernels are installed (e.g. jupyter kernelspec list)

For more information about the Scala kernel, please visit the [Apache Toree](https://toree.apache.org/) page.

### R Kernel (IRkernel)

Considering we would like to enable the IRkernel to run on YARN Cluster and Client mode
we would have to copy the sample configuration folder **spark_R_yarn_cluster** to where the
Jupyter kernels are installed (e.g. jupyter kernelspec list)

For more information about the iR kernel, please visit the [IRkernel](https://irkernel.github.io/) page.

### Adjusting the kernel specifications

After installing the kernel specifications, you should have a `kernel.json` that resembles the
following (this one is relative to the Python kernel):

FIXME - update spec content to be provisioner based

```json
{
  "language": "python",
  "display_name": "Spark - Python (YARN Cluster Mode)",
  "metadata": {
    "kernel_provisioner": {
      "class_name": "enterprise_gateway.services.processproxies.yarn.YarnClusterProcessProxy"
    }
  },
  "env": {
    "SPARK_HOME": "/usr/hdp/current/spark2-client",
    "PYSPARK_PYTHON": "/opt/conda/bin/python",
    "PYTHONPATH": "${HOME}/.local/lib/python3.6/site-packages:/usr/hdp/current/spark2-client/python:/usr/hdp/current/spark2-client/python/lib/py4j-0.10.6-src.zip",
    "SPARK_YARN_USER_ENV": "PYTHONUSERBASE=/home/yarn/.local,PYTHONPATH=${HOME}/.local/lib/python3.6/site-packages:/usr/hdp/current/spark2-client/python:/usr/hdp/current/spark2-client/python/lib/py4j-0.10.6-src.zip,PATH=/opt/conda/bin:$PATH",
    "SPARK_OPTS": "--master yarn --deploy-mode cluster --name ${KERNEL_ID:-ERROR__NO__KERNEL_ID} --conf spark.yarn.submit.waitAppCompletion=false",
    "LAUNCH_OPTS": ""
  },
  "argv": [
    "/usr/local/share/jupyter/kernels/spark_python_yarn_cluster/bin/run.sh",
    "--kernel-id",
    "{kernel_id}",
    "--response-address",
    "{response_address}",
    "--public-key",
    "{public_key}"
  ]
}
```

The `metadata` and `argv` entries for each kernel specification should be nearly identical and
not require changes. You will need to adjust the `env` entries to apply to your specific
configuration.

You should also check the same kinds of environment and path settings in the corresponding
`bin/run.sh` file - although changes are not typically necessary.

After making any necessary adjustments such as updating `SPARK_HOME` or other environment
specific configuration and paths, you now should have a new kernel available to execute your
notebook cell code distributed on a Hadoop YARN Spark Cluster.