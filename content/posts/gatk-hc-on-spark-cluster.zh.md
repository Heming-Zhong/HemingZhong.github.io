+++
title = '在Spark集群上部署GATK HaplotypeCaller'
date = 2024-04-02T20:25:42+08:00
ZHcategories = ["体系结构", "生物信息"]
draft = true
ShowToc = true
TocOpen = true
+++

## 介绍

这篇文章来自我自己的一个工作，目标是对GATK HC在HPC集群上的性能进行优化。为了评估我自己工作的相对性能提升，我同样需要部署基于Spark集群的GATK HC作为一个比较的基准对象。

## GATK HaplotypeCaller介绍

这篇文章假设你已经对GATK和基因组变异检测流程较为熟悉。因此我将只简单过一下它的背景。GATK是一个广泛使用的基因组分析工具集。HaplotypeCaller（简称HC）是其中的一个变异检测工具，并且在实际DNA变异分析流程中被广泛应用。

## 安装相关依赖

正如文章标题，我们想要在一个独立的Spark集群上部署GATK HC。所以第一步我们应该在我们的实验机器上安装并配置所有的相关环境。这里我使用一个基于Slurm管理的HPC集群。这里我首先使用Spack包管理器安装JDK和GATK。下面是安装的具体命令：

```shell
spack install gatk@4.5.0.0^openjdk@17.0.5_8
```

然后，我再手动下载并解压最新版本的Spark：

```shell
wget -c https://www.apache.org/dyn/closer.lua/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz
tar -zxvf spark-3.5.1-bin-hadoop3.tgz
```

## Spark集群部署

在所有需要的软件都被安装之后，我们就需要启动一个Spark集群来提交我们的GATK HC任务。

首先，我们先配置一下Spark的相关运行环境和选项。编辑文件 `conf/spark-env.sh` and `conf/workers`:

```shell
cp conf/spark-env.sh.template conf/spark-env.sh
cp conf/workers.template conf/workers
```

其中，将JDK路径和主节点的信息添加到`conf/spark-env.sh`, 然后将工作节点的信息添加到`conf/workers`:

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

由于这里我们的HPC集群是基于Slurm作业调度系统的，因此我们也可以将Spark集群部署的过程与Slurm结合起来，实现更自动化的节点申请与释放。这里我以Slurm为例，我们可以创建如下的bash脚本，来使用SBATCH自动部署一个Spark集群：

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

使用sbatch执行该脚本，并通过运行参数来配置工作节点的数量。

```shell
$SPARK_HOME/sbin/start-master.sh
sbatch -n 10 start-spark-workers.sh
```

最后，当任务执行完成时，可以通过以下的脚本来自动化停止Spark集群并释放申请的全部节点：

[TODO: scripts]

## 运行GATK HC Spark

在Spark集群启动之后，我们就可以向其提交一个GATK HC任务，使其在多个节点上分布式运行：

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