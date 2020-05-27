---
layout: default
title: Anaconda device
nav_order: 3
parent: Spartan HPC Usage
---

## Anaconda

module load Anaconda2/5.3.0
module load Anaconda3/5.3.1

module load Python/3.7.1-GCC-6.2.0


module install Cython

py36 = name of conda environ

conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge


## Pybedtools

module load Python/3.6.10-intel-2017.u2-GCC-6.2.0-CUDA10.1
module load pybedtools/0.7.10-intel-2017.u2-Python-3.6.4


source p6c10/bin/activate


module load pybedtools/0.7.10-intel-2017.u2-Python-3.6.4

virtualenv p6c10
source p6c10/bin/activate
pip install absl-py
pip install scipy==1.4.1
pip install pandas==0.22.0
pip install matplotlib


Python/3.7.1-GCC-6.2.0










