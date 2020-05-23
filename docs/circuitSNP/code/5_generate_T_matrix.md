---
layout: default
title: 5. Generate T matrix
nav_order: 5
parent: Python Implementation
---


##### [Back to contents](./..)


In the previous step, we analysed which motifs existed in which windows. 

so now, we have open / closed regions and motif membership / absence in each window. This is the data used for current step.

In the current step, we wish to generate T matrix which denotes which motifs exist or do not exist at any window, and whether that window has open or closed chromatin.

##### Source Code: [./generate_T_matrix.py](../generate_T_matrix.py)

```sh
python ./generate_T_matrix.py
```

Read results from previous script to analyse chromatin open or closed
```python
windows = pd.read_csv('chromatin_reads.csv') #already formatted as data frame
```

Set how many open / closed regions we want by changing these two variables
```python
TOTAL_OPEN_CHROMATIN=100
TOTAL_CLOSED_CHROMATIN=100
```

This makes no difference, since we want to include all windows.

If we want to include all windows, we have to set SUBSET_OPEN_CLOSED=0. Otherwise set to 1 if we want to subset. 

So set to 0

```python
SUBSET_OPEN_CLOSED=0 #if this set to 0, we include all windows
```

Number of motif given to us by centiSNP study (Moyerbrailean et al., 2016)
```python
NUMBER_OF_MOTIFS=1372
```


If we are not using all windows, then we have to sample from open and closed windows separately.

##### open regions subsampling

note we have to use 'sample' from random.sample to generate permuted sample (randomly selected from all open chromatin regions)

```python
#open
open_window_indices=open_windows['window_index']
#use 'sample' to generate permutation
open_window_sample_indices=sample(set(open_window_indices),100)
samp_chromatin_open=open_windows[open_windows.window_index.isin(open_window_sample_indices)]
```
##### closed regions subsampling

Now we load in previous results regarding presence of motifs in windows, stored in `motif_memberships.csv`

```python
motif_memberships=pd.read_csv('motif_memberships.csv')
unique_motifs=set(motif_memberships['motif'])
...
motif_memberships['motif_index'] = motif_memberships['motif'].map(motifs_dict)
```

Note that `motif_memberships` is a very large table with 3045603 rows.

```
>>> motif_memberships
        chrom      start       stop   motif       float sign  chrom_dict  d_motif_membership
0        chr1    2203656    2203666  M00140  326.956217   +           13                 918
1        chr1    4815881    4815891  M00140  111.470794   +           13                3329
2        chr1    5034577    5034587  M00140    8.410601   -           13                3579
3        chr1    5238635    5238645  M00140    4.346002   -           13                3783
4        chr1    5600926    5600936  M00140    4.346002   +           13                4146
...       ...        ...        ...     ...         ...  ...         ...                 ...
3045598  chrX  154944162  154944172  M00141    5.723951   -            4             1850169
3045599  chrY   19791318   19791328  M00141    9.169588   -           36             1866533
3045600  chrY   19791342   19791352  M00141    9.169588   -           36             1866533
3045601  chrY   23790632   23790642  M00141    5.723951   -           36             1866946
3045602  chrY   28430492   28430502  M00141    9.169588   +           36             1867233

[3045603 rows x 8 columns]

```

When we generate the T matrix, we will loop through rows of `motif_memberships`, and if the motif is present in a window, we will set the corresponding T matrix entry to 1. 

So to save computation time, we want to avoid looping through rows which do not indicate the presence of a motif.  

`d_motif_membership` stores the index of the window (indexed to `windows`) where a motif is present.  It will be empty otherwise

`all_present_windows` is a vector containing the indices of all windows, starting from 1. 

So, to remove the rows where a motif does not exist inside any applicable window, we just have to filter the `motif_memberships` table and remove all rows where `d_motif_membership` is empty, or does not exist in `all_present_windows`

```
>>> motifs.head()
  chrom    start     stop   motif       float sign  chrom_dict  d_motif_membership
0  chr1  2203656  2203666  M00140  326.956217   +           13                 918
1  chr1  4815881  4815891  M00140  111.470794   +           13                3329
2  chr1  5034577  5034587  M00140    8.410601   -           13                3579
3  chr1  5238635  5238645  M00140    4.346002   -           13                3783
4  chr1  5600926  5600936  M00140    4.346002   +           13                4146

```



```python
all_present_windows=chromatin_samples.index
applicable_motifs=motif_memberships[motif_memberships.d_motif_membership.isin(all_present_windows)]
```

Now set up input arrays for GPU function

```python
#1.
#apply function to convert motif name to int
motif_int = lambda x: int(x[1:])
start_time=time.time()

#map motif name to integer to be used as column index for T matrix
#eg 'M00001' corresponds to column 1, 'M00002' to column 2, etc
a_motifs_list=list(applicable_motifs.motif)
applicable_motifs_int= list(map(motif_int, a_motifs_list))
motif_name_instances = np.array(applicable_motifs_int,dtype=np.int32)
```

We only loop through windows where we know motifs are present (to save time)

```python
#2. 
#motif_membership_windows is all window indices
motif_membership_windows=np.array(applicable_motifs.d_motif_membership,dtype=np.int32)
```


Store all window indices in an array so that we can loop through them

```python
#3. in_t_window_index
#in_t_window_index is relevant window index (row of T matrix)
t_window_index= np.array(chromatin_samples.index,dtype=np.int32)
```

Initialise the empty T matrix. 

Note that for speed of computation, data type of Boolean has been used (0,1) only permissible values

These will be converted to 1 and 0 later during NN stage

```python
#4. 
#out_t_matrix is empty array with:
# columns corresponding to motif names (1372 of them)
# rows corresponding to instance of training data (equivalent to nrow in t matrix)
# boolean cos true/false
in_t_matrix = np.empty(shape=(chromatin_samples.shape[0],1372),dtype=bool)
in_t_matrix.fill(False)
```


Function definition. See earlier documents for illustration of how to initialise function on GPU array.


```python
@cuda.jit
def set_motifs(motif_name_instances, motif_membership_windows, t_window_index, t_matrix):

	#we are looping through two variables (motif number, window number), so we initialise a 2x2 grid on the GPU array

	T_row, M_row = cuda.grid(2) #motif row
	d1, d2 = cuda.gridsize(2)

	#t_rows is the number of windows we are checking
	t_rows=t_matrix.shape[0]

	#number of rows in the motif lookup table
	motif_rows=len(motif_name_instances)

	for _motif in range(M_row, motif_rows, d2):

		#each motif corresponds to relevant column of T matrix
		relevant_motif = motif_name_instances[_motif]

		#each window corresponds to row of T matrix
		relevant_window=motif_membership_windows[_motif]

		#now loop thru each window, and then each motif to check presence of motif in window
		for _row in range(T_row, t_rows, d1):
			
			if t_window_index[_row]==relevant_window:
				#	set:
				# 	row = window index
				#	column = motif index / name
				# 	motif exists here, so set to True

				t_matrix[_row,relevant_motif-1]=True
```


Assign partitions of training, validation and test data (more code related to sampling indices is in source code)


```python
TRAINING_PERCENTAGE=0.75
VALIDATION_PERCENTAGE=0.125
TEST_PERCENTAGE=1-TRAINING_PERCENTAGE-VALIDATION_PERCENTAGE
```


Now we save out as `feather` data format which is lightweight (way better than CSV)

```python
#write out as 'T_MATRIX.csv'
post_t_df.to_feather('T_MATRIX.feather',index=False)
```


##### [Back to contents](./..)
