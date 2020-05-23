---
layout: default
title: Imports and functions
nav_order: 8
parent: Python Implementation
---




##### All imports are here:

```python
import pandas as pd
import numpy as np
import io
import pickle
import timeit
import time
import sys
from numba import cuda, jit
import copy
from random import sample
import os
```

Save path must be edited for compatibility on diff systems

```python
#get save_path
save_path=open('save_path.txt','r').read().splitlines()[0]
#generate compiled and downloaded data directories
COMP_DIR=save_path+'compiled_data/'
DOWN_DIR=save_path+'downloaded_data/'
MODEL_DIR=save_path+'models/'
```


Following code is deprecated, but keep for now
```python

NUMBER_OF_MOTIFS=1372

motifs=['M' + str(motif_string).zfill(5) for motif_string in range(1,NUMBER_OF_MOTIFS+1)]
```
Chromosome dict for mapping

```python
#construct dictionary so that we can group by chromosome

total_chrom=['chr'+str(k+1) for k in range(22)]+['chrX','chrY']

chromosome_dict={}
for i,chrom in enumerate(total_chrom):
	chromosome_dict[chrom]=i

#create dictionary to map chromosome to integer
chr_dict_for_dsqtl={}
for chrom,chr_int in zip(chromosome_dict.values(),chromosome_dict.keys()):
	chr_dict_for_dsqtl[chrom]=chr_int
```

Following motif code is deprecated, keep for now
```python
#construct dictionary to convert motif to number 
#all_motif=['M' + str(mot_num).zfill(5) for mot_num in range(1,NUMBER_OF_MOTIFS+1)]
#motif_dict={}
#for k,motif in enumerate(all_motif):
#	motif_dict[motif]=k+1


```