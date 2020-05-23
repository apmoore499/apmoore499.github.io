---
layout: default
title: Slurm example script
nav_order: 3
parent: Spartan HPC Usage
---

## Example Slurm script:

Save this w file extension '.slurm'

```sh
#job specifications (time, cpus, etc)

#!/bin/bash
#SBATCH --nodes 1
#SBATCH --account=punim0614
#SBATCH --qos=gpgpumdhs
#SBATCH --partition gpgpu
#SBATCH --gres=gpu:p100:1
#SBATCH --time 01:00:00
#SBATCH --cpus-per-task=1
#SBATCH --job-name=circuitSNP_4


#load modules
module load Python/3.6.10-intel-2017.u2-GCC-6.2.0-CUDA10.1
module load Tensorflow/2.1.0-intel-2017.u2-GCC-6.2.0-CUDA10.1-Python-3.6.10-GPU
source ./p6/bin/activate

#execute relevant python script
python m4_d.py
```


## Run using:

```sh
[apmoore@spartan-login1 circuitSNP]$ sbatch tf.slurm
Submitted batch job 16346235
```

job stdout will be written to slurm-16346235.out