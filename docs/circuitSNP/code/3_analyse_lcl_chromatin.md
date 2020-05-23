---
layout: default
title: 3. Detect open / closed regions
nav_order: 3
parent: Python Implementation
---

##### [Back to contents](./..)


The previous step retrieved the locations of open chromatin windows in LCL and ALL tissues.

The goal of this step is to use these locations to derive regions of open and closed chromatin between both tissue types. 

Specifically, we want to see whether chromatin that was previously open in ALL tissue has been closed in LCL tissue, or has been kept open.

Open and closed regions of chromatin are classified as follows:

- Chromatin open: footprints in region of ALL tissues overlap with footprints in region of LCL tissues

- Chromatin closed: footprints in region of ALL tissues do not overlap with any footprint in LCL tissues


##### Source Code: [./analyse_footprint_overlap.py](../analyse_footprint_overlap.py)

The python script to analyse chromatin openness is run in bash terminal:

```sh
python ./analyse_footprint_overlap.py
```


This project uses pandas dataframes - this data structure is helpful for manipulating tables and extracting values (this data format was copied from R). Documentation on pandas dataframe here: <https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html>

We read csv files as dataframe objects with the following call:

```python
import pandas as pd
<dataframe_object> = pd.read_csv('file.csv')
```

So now let's load reads for ALL tissus and LCL tissues

Load ALL reads:
```python
#ALL reads
openALL = open("rawdata/wgEncodeRegDnaseClusteredV3.bed",'r')
#read in files as data frame usign pandas
all_reads = pd.read_csv(openALL, sep="\t")
```
Rename columns
```python
all_reads.colnames=['chrom','start','stop','1','2','3','4','5']
all_reads.columns = all_reads.columns.str.strip()
```
Do same for LCL
```python
#LCL reads
openLCL = open("rawdata/wgEncodeAwgDnaseUwdukeGm12878UniPk.narrowPeak",'r')
#read in files as data frame usign pandas
lcl_reads = pd.read_csv(openLCL, sep="\t")
```
Create column to store whether chromatin is open or closed
```python
#chromatin_open column refers to whether chromatin is open or closed
all_reads["chromatin_open"]=""
all_reads["chromatin_overlap"]=""
```
Comparing elements is very slow unless we use the GPU.

GPU functions can only work on arrays - they cannot work on dataframes directly.

So now extract relevant variables as arrays to be passed in.

Relevant variables are the start and stop locations of reads for ALL and LCL tissue types
```python
#extract arrays to pass into function
all_start=np.array(all_reads['start'],dtype=np.int32)
all_stop=np.array(all_reads['stop'],dtype=np.int32)

lcl_start=np.array(lcl_reads['start'],dtype=np.int32)
lcl_stop=np.array(lcl_reads['stop'],dtype=np.int32)
```
The chromosome identifier is given as a string, but we need to to convert this to an int, becuase GPU cannot work on string arrays.

So create a dictionary for unique chromosome names called 'chrom_dict'

```python
#get unique values for chromosome column in ALL and LCL data
all_chrom=set(all_reads['chrom'])
lcl_chrom=set(lcl_reads['chrom'])
#Create the union of all chromosomes in either ALL and LCL data
total_chrom=all_chrom.union(lcl_chrom)

#as we cannot pass string values into GPU parallel function,
#construct dictionary so that we can group by chromosome indexed by integer
chrom_dict={}
for i,chrom in enumerate(total_chrom):
	chrom_dict[chrom]=i

```
Now map chromosome dictionary values back to each read via a new column called 'chrom_dict'
```python
#now use this to replace in lcl and all (convert to integer)
all_reads['chrom_dict'] = all_reads['chrom'].map(chrom_dict)
lcl_reads['chrom_dict'] = lcl_reads['chrom'].map(chrom_dict)
```
And then extract chromosome arrays for ALL and LCL tissues so that we can use them to analyse overlapping regions
```python
#extract chromosomes as list to pass into function
all_chrom=np.array(all_reads['chrom_dict'],dtype=np.int32)
lcl_chrom=np.array(lcl_reads['chrom_dict'],dtype=np.int32)
```
Now setup input arrays on GPU cluster
```python
# move input data to the GPU cluster
d_all_start = cuda.to_device(all_start)
d_all_stop = cuda.to_device(all_stop)
d_lcl_start = cuda.to_device(lcl_start)
d_lcl_stop = cuda.to_device(lcl_stop)
d_all_chrom = cuda.to_device(all_chrom)
d_lcl_chrom = cuda.to_device(lcl_chrom)
```
Set up output arrays on GPU cluster
```python
# create arrays on GPU cluster to allow data to be retrieved from cluster
d_chromatin_open = cuda.device_array_like(d_all_start)
d_window_start = cuda.device_array_like(d_all_start)
d_window_end = cuda.device_array_like(d_all_start)
d_window_start_id = cuda.device_array_like(d_lcl_id)
d_window_end_id = cuda.device_array_like(d_lcl_id)

```
We also have to set window length. Here have used 300. The script will raise exception if length not divisible by 2.
```python
#set window length - must be divisible by 2
WINDOW_LENGTH=300
```

Now, ready to set up function ```check_chromatin_open()``` 

```check_chromatin_open()``` performs analysis of overlapping regions, explained as follows

Define ```@cuda.jit``` to make run on GPU
```python
#Define function check_chromatin_open to perform analysis of overlap
@cuda.jit
def check_chromatin_open(
	al_start,
	al_end,
	lc_start,
	lc_end,
	al_chrom,
	lc_chrom,
	chromatin_open,
	window_start,
	window_end,
	window_start_id,
	window_end_id):
```
Loop thru every read for ALL and every read for LCL. If we get any overlapping regions then make a note of this in the chromatin open state for the corresponding row in ALL.

Reads are are overlapping if the open chromatin regions overlap like this:

```
Read 1:	  	 #-------------#
Read 2:	   #----------------#
```
`#` refers to a start or end point
`---` refers to a set of alleles

This depicts that read 1 terminates before read 2. Consider that read 1 might terminate after read 2:
```
Read 1:	  	 #-----------------#
Read 2:	   #----------------#
```
For this analysis, ignore where the Read 1 ends. Denote this as `/`:
```
Read 1:	  	 #-------------/
Read 2:	   #----------------#
```
Which means that read 1 might terminate before read 2 terminates, or after.  It doesn't matter - we're only looking at which read starts the overlapping region. 

Now, check for overlap with either LCL starting region, or ALL starting region.

We have two conditions to match:

##### LCL starts the overlapping region

First, check chromosome is the same: ```al_chrom[i1]==lc_chrom[i2]```

Then, check that LCL starts the region

```python
if al_chrom[i1]==lc_chrom[i2] and lc_start[i2]<=al_start[i1] and al_start[i1]<=lc_end[i2]:
	#this is case where we have:
	# ALL  	 #-------------/
	# LCL   #----------------#
	
	chromatin_open[i1] = 1
	window_start[i1]=lc_start[i2]
	window_start_id[i1]=i2+0.0003
```


##### ALL starts read

First, check chromosome is the same: ```al_chrom[i1]==lc_chrom[i2]```

Then, check that ALL starts the region

```python
elif al_chrom[i1]==lc_chrom[i2] and al_start[i1]<=lc_start[i2] and lc_start[i2]<=al_end[i1]:
	#this is case where we have:
	# ALL  #-------------/
	# LCL   #----------------#
```

Now, we have to make window 300bp long

```python
#now, extend window to be 300bp as defined in WINDOW_LENGTH
#borrowed from Denny Shin's code line 135 of dataprep.py
half_window_length=WINDOW_LENGTH/2 #need to have window for either side
centroid=window_start[i1]+window_end[i1]
resized_window_start[i1]=centroid-half_window_length-1
resized_window_end[i1]=centroid+half_window_length
```

Finally check that the new window encapsulates limits of old window

```python
if resized_window_start[i1]<=window_start[i1] and resized_window_end[i1]>=window_start[i1]:
	#so this corresponds to:
	#RESIZED WINDOW	        #------------------------#
	#WINDOW	 		    #------------------#
	window_encapsulates_limits[i1]=1
#if the window steps outside the original window, flag this - might come in handy later for troubleshooting
else:
	window_encapsulates_limits[i1]=0
```

In order to call function on GPU, we must specify number of blocks = 1000 and block size = 256

These values are chosen by trying out different values - they run ~ 6 seconds.

We must call function and then call cuda.synchronise() to wait for all cores to finish before proceeding

```python
check_chromatin_open[1000,256](
	d_all_start,
	d_all_stop,
	d_lcl_start,
	d_lcl_stop ,
	d_all_chrom,
	d_lcl_chrom,
	d_chromatin_open,
	d_window_start,
	d_window_end,
	d_window_start_id,
	d_window_end_id)
cuda.synchronize()
```
Now we can extract data from cluster using copy_to_host() syntax

```python
#now add results to all_reads and then save out...
all_reads['d_chromatin_open']=d_chromatin_open.copy_to_host()
all_reads['d_window_start']=d_window_start.copy_to_host()
all_reads['d_window_end']=d_window_end.copy_to_host()
all_reads['d_window_start_id']=d_window_start_id.copy_to_host()
all_reads['d_window_end_id']=d_window_end_id.copy_to_host()
```

Save results

```python
#now write out chromatin_open...
all_reads.to_csv('chromatin_reads.csv',index=False)
```

##### [Back to contents](./..)
