---
layout: default
title: Anaconda device
nav_order: 3
parent: Spartan HPC Usage
---

## Anaconda

Two installed anaconda modules:

```sh
module load Anaconda2/5.3.0
module load Anaconda3/5.3.1
```

To configure anaconda for bioconda

```sh
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

To import Cython - sometimes this is necessary to install packages (running setup wheel for setuptools.py etc)
```sh
module install Cython
```

## Pybedtools

To import pybedtools:

```sh
module load Python/3.6.10-intel-2017.u2-GCC-6.2.0-CUDA10.1
module load pybedtools/0.7.10-intel-2017.u2-Python-3.6.4
source p6c10/bin/activate
```
