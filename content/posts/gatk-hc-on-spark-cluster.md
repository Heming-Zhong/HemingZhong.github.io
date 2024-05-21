+++
title = 'Deploy GATK HaplotypeCaller on Spark Cluster'
date = 2024-04-02T20:25:42+08:00
categories = ["Infrasture", "Bioinformatics"]
draft = true
ShowToc = true
TocOpen = true
+++

## Introduction

This post comes from my experience in my work to optimize GATK HC performance on HPC clusters. To evalute the relative speedup of my own work, I also need to setup GATK HC on Spark clusters as baseline.

## What's GATK HaplotypeCaller?

This post assumes that you have the knowledge about GATK and genomics variant calling process, so I will only introduce its background in brief. Genome Analysis ToolKit(GATK) is a widely-used tool set of genome analysis softwares. HaplotypeCaller(HC) is one of its variant calling tool which is widely used in DNA variant discovery. 

## Installation

As mentioned in title, we want to deploy GATK HC on a standalone Spark Cluster. So first of all we need to install and configure all environment we need on our machines. Here I use a HPC cluster managed by Slurm, so I first install jdk and GATK using spack package manager with following commands:

```shell
spack install gatk@4.5.0.0^openjdk@17.0.5_8
```

After that, I manually download the latest version of Spark package and uncompress it:

```shell
wget -c https://www.apache.org/dyn/closer.lua/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz
tar -zxvf spark-3.5.1-bin-hadoop3.tgz
```

## Spark Cluster Deployment

After all softwares needed are installed. We need to launch a spark cluster to submit our GATK jobs.

First, create configuration files `conf/spark-env.sh` and `conf/workers`:

```shell
cp conf/spark-env.sh.template conf/spark-env.sh
cp conf/workers.template conf/workers
```

Add java path and master node info in `conf/spark-env.sh`, and add worker hostnames in `conf/workers`:

```shell
# in conf/spark-env.sh:
export JAVA_HOME=<your java path>
export SPARK_MASTER_HOST=<your master node ip>

# in conf/workers:
<your worker ip 1>
<your worker ip 2>
<your worker ip 3>
...
```

For HPC clusters managed by slurm or other similar framework, we can also cooperate spark cluster deployment with these underlying frameworks. Taken slurm as an example, we can create following script to start a spark cluster using SBATCH:

```shell
#!/bin/bash
#SBATCH --job-name=spark-cluster-3
#SBATCH --output=spark3.%j.out
#SBATCH --error=spark3.%j.err
#SBATCH --ntasks-per-node=1
#SBATCH --time=04:00:00
#SBATCH --partition=ai

# load spark envrionment into PATH
export SPARK_HOME=<path to spark home dir>
export PATH="$SPARK_HOME/bin:$PATH"
export MASTER=<login node ip>:7077

# start master on login node


# start a worker on per allocated compute node
srun $SPARK_HOME/sbin/start-worker.sh $MASTER && echo "Worker started on node `hostname`"
```

execute the script with sbatch and the number of worker nodes:

```shell
$SPARK_HOME/sbin/start-master.sh
sbatch -n 10 start-spark-workers.sh
```

## Run GATK HC Spark

With a running spark cluster, we can submit a GATK HC Spark job on multi-nodes:

```shell
gatk HaplotypeCallerSpark \
    --spark-runner SPARK \
    --spark-master spark://ln206:7077 \
    -R <path to fasta> \
    -I <path to bam> \ 
    -O <path to vcf> \ 
    -- \
    --driver-cores=4 \
    --driver-memory=30g \
    --executor-memory 128g \
```