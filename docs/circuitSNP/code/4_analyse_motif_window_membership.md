---
layout: default
title: 4. Motif membership
nav_order: 4
parent: Python Implementation
---

##### [Back to contents](./..)

In the previous step, reads were analysed to determine whether chromatin is open or closed in LCL reads, with a window size of 300bp.

The goal of the current step is to determine which motifs exist in these windows.

We use the result of chromatin open or closed as the data for current step


##### Source Code: [./analyse_motif_presence_in_windows.py](../analyse_motif_presence_in_windows.py)

We call the python script in bash terminal:

```sh
python ./analyse_motif_presence_in_windows.py
```

Read in files 

```python
#this is our open / closed chromatin
windows = pd.read_csv('chromatin_reads.csv') #already formatted as data frame
#read in all motifs..
motif_reads=pd.read_csv('COMBO/all_combo_reads.csv')
motif_reads.columns = ['chrom','start','stop','motif','float','sign','d_motif_membership']
```

Very similar to comparing LCL and ALL tissue types, we use a function to compare:

- the start and end positions for motif
- the start and end positions for the window 

So we loop through the table of all window reads (generated in the previous step), and if the motif exists in a window, we record the index of the window (which is its row number in the table of all window reads)


function defined here
```python
@cuda.jit
def check_motif_membership(
```
first case -  motif start precedes window
```python
if window_chrom[i2]==motif_chrom[i1] and window_start[i2]<=motif_start[i1] and motif_start[i1]<=window_end[i2]:
	#this is case where we have:
	# WINDOW  	 #-------------/
	# MOTIF   	#----------------#
	motif_membership[i1]=i2
```
second case - window start precedes motif
```python
elif window_chrom[i2]==motif_chrom[i1] and motif_start[i1]<=window_start[i2] and window_start[i2] <= motif_end[i1]:
	#this is case where we have:
	# WINDOW   #-------------/
	# MOTIF   	#----------------#
	motif_membership[i1]=i2

```
settings for cuda block size are [1000,256]
```python
check_motif_membership[1000,256](
	d_motif_start,
	d_motif_end,
	d_motif_chrom,
	d_window_start,
	d_window_end,
	d_window_chrom,
	d_motif_membership)
```
might need to fiddle with block size settings for cuda, time taken is very long
```python
completed in: 438.7458071708679 seconds.
```
Append the column designating motif membership, and write out file:
```python
motif_reads['d_motif_membership']=d_motif_membership.copy_to_host()
#write out
motif_reads.to_csv('motif_memberships.csv',index=False)
```


##### [Back to contents](./..)