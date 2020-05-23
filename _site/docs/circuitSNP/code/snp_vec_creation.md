#####generate_SNP_vectors_6.py


In this stage, we wish to identify all SNPs in a region. We want to generate 3 outputs:

The aim:

1. Generate F vectors (both ref and alternate contained)
2. Generate reference vectors
3. Generate alternate vectors


Dependencies, functions, imports

```python 
from functions import *
```

Data downloaded in earlier steps:

SNP_READS is all SNPs

dsqtl consists of SNPs which are also dsQTLs


```python
#read in SNPs
SNP_READS=pd.read_csv(DOWN_DIR  + 'compendium/all_compendium_reads.csv')
SNP_READS['SNP_window_index']=-1

#read in dsQTLs
dsqtl=pd.read_csv(DOWN_DIR  + 'test/dsQTL.eval.txt',sep='\t')
dsqtl['SNP_window_index']=-1

```

rename for easier referencing


```python

SNP_READS.rename(columns = {'pos    (0-based)':'snp_start'}, inplace = True) 
SNP_READS.rename(columns = {'pos1   (1-based)':'snp_end'}, inplace = True) 
SNP_READS.rename(columns = {'motif  (motif ID, see factorNames.txt)':'motif'}, inplace = True) 

dsqtl.rename(columns = {'pos0':'snp_start'}, inplace = True) 
dsqtl.rename(columns = {'pos':'snp_end'}, inplace = True) 

```


before manipulating tables, number of rows in each is
```
>>> SNP_READS.shape
(5202041, 12)
>>> dsqtl.shape
(27702, 24)
```

remove sex chromosomes
```python
SNP_READS=SNP_READS[~SNP_READS.chr.isin(['chrX','chrY'])]
dsqtl=dsqtl[~dsqtl.chr.isin(['chrX','chrY'])]
```

new number of rows in each
```
>>> SNP_READS.shape
(5057359, 12)
>>> dsqtl.shape
(27702, 24)
```

Number of unique SNP is 3766427

```python
>>> SNP_READS[['chr','snp_start','snp_end']].drop_duplicates().shape
(3766427, 3)
```

But we should expect this number to be larger when we consider that duplicate SNP (same location and chromosome) will overlap with different SNPs

```python
SNP_READS[['chr','snp_start','snp_end','motif']].drop_duplicates().shape
(5057359, 4)
```
So exactly same as the number of rows in entire table 

Number of unique dsQTL in file is 

```python
>>> dsqtl[['chr','snp_start','snp_end']].drop_duplicates().shape
(27702, 3)
```


we want to find out how many dsQTL exist in SNP_READS (or how many SNP are dsQTL)

can use merge function

note that set indicator=True gives extra column `._merge` that counts outcome for particular row - whether row is common to both data frames, or only one

```python
common_dsqtl_snp = pd.merge(dsqtl,SNP_READS[['chr','snp_start','motif']], on=['chr','snp_start'],how='left',indicator=True)
#we are interested in the number of items that matched in both (indicates dsqtl in all SNP)
common_dsqtl_snp._merge.value_counts()

```

left_only indicates values that only exist in dsqtl table
so number of SNP exist in both is 8898 (shown below)
```sh
>>> common_dsqtl_snp._merge.value_counts()
left_only     21849
both           8898
right_only        0
Name: _merge, dtype: int64
```

verify this with a right join should get 8898 again

```python
common_dsqtl_snp_right = pd.merge(dsqtl,SNP_READS[['chr','snp_start','motif']], on=['chr','snp_start'],how='right',indicator=True)
#we are interested in the number of items that matched in both (indicates dsqtl in all SNP)
common_dsqtl_snp_right._merge.value_counts()
```

perfect - therefore we have 8898 SNP which are also dsQTL in SNP_READS
```sh
... common_dsqtl_snp_right._merge.value_counts()
right_only    5048461
both             8898
left_only           0
```

read in locations of motifs

```python
relevant_motifs=pd.read_feather(COMP_DIR + '2_motifs_in_lcl_windows.feather')
>>> relevant_motifs.head()
   index chrom    start     stop   motif       float sign  chrom_dict  d_motif_membership
0     21  chr1  2203656  2203666  M00140  326.956217   +           70                 918
1     38  chr1  4815881  4815891  M00140  111.470794   +           70                3329
2     42  chr1  5034577  5034587  M00140    8.410601   -           70                3579
3     50  chr1  5238635  5238645  M00140    4.346002   -           70                3783

```


convert chromosme to int for to use in CUDA func

```python


#convert chromsome to number...
all_chromosomes=list(set(SNP_READS.chr))
chr_dict={}
for i,c in enumerate(all_chromosomes):
	chr_dict[c]=i

#convert chromosome to int
SNP_READS['chr_int']=-1
SNP_READS=SNP_READS.assign(chr_int = lambda dataframe: dataframe['chr'].map(lambda chr: chr_dict[chr]))

dsqtl['chr_int']=-1
dsqtl=dsqtl.assign(chr_int = lambda dataframe: dataframe['chr'].map(lambda chr: chr_dict[chr]))

```

convert motif to int to use in CUDA func


```python

#convert motif to int
from functions import *
SNP_READS['motif_int']=SNP_READS['motif'].map(motif_dict)



```

subset to motif only under consideration

```python
#we only want to select motif <= M1372
SNP_READS[SNP_READS.motif_int<=1372]

SNP_READS=SNP_READS[SNP_READS.motif_int<=1372]
```

double check no NAN vals

```python
>>> SNP_READS.motif_int.apply(isnan).value_counts()
False    3134652
Name: motif_int, dtype: int64
```

So all values in motif_int are not NAN, so all are from motifs in 1-1372



Now see how many unique SNP. We can use snp_start as all starts correspond uniquely to each individual SNP
```python
SNP_READS[['chr_int','snp_start']].drop_duplicates().shape
(2388566, 2)
```

So we have 2388566 individual SNP





How many of those are dsQTL - 4165

```python
unique_SNP=SNP_READS[['chr_int','snp_start']].drop_duplicates()
reduced_dsqtl_snp = pd.merge(dsqtl,unique_SNP, on=['chr_int','snp_start'],how='left',indicator=True)

reduced_dsqtl_snp._merge.value_counts()
left_only     23537
both           4165
right_only        0
```

re-merge to map unique_SNP back
```python
SNP_READS=pd.merge(SNP_READS,unique_SNP, on=['chr_int','snp_start'],how='left',indicator=True)
```

check that all match up

```python
>>> SNP_READS._merge.value_counts()
both          3134652
right_only          0
left_only           0
Name: _merge, dtype: int64
```

move data onto CUDA array

```python
in_snp_index=np.array(SNP_READS.SNP_INDEX,dtype=np.int32)

in_snp_effect=np.array(SNP_READS.effect,dtype=np.int32)

in_snp_motif=np.array(SNP_READS.motif_int,dtype=np.int32)

in_snp_priorlodds=np.array(SNP_READS.ref_priorlodds,dtype=np.float32)

in_snp_altlodds=np.array(SNP_READS.alt_priorlodds,dtype=np.float32)

in_all_motifs=np.array(range(1,1373),dtype=np.int32)
```

```python
#T matrix constructed from number of unique snp, and number of motif
T_mat=np.empty([unique_SNP.shape[0],NUMBER_OF_MOTIFS],dtype=np.bool)

#now generate placeholders for output matrices on CUDA memory
out_T_matrix_F = cuda.device_array_like(T_mat)
out_T_matrix_Ref = cuda.device_array_like(T_mat)
out_T_matrix_Alt = cuda.device_array_like(T_mat)
#run function
```
