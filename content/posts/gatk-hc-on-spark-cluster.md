+++
title = 'Deploy GATK HaplotypeCaller on Spark Cluster'
date = 2024-04-02T20:25:42+08:00
categories = ["Infrasture", "Bioinformatics"]
draft = false
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
wget 
tar -zxvf ***
```
