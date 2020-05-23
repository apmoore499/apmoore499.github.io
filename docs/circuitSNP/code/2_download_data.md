---
layout: default
title: 2. Download data
nav_order: 2
parent: Python Implementation
---



In this step, the goal is to obtain the data for:

##### 1. SNP locations <http://genome.grid.wayne.edu/centisnps/compendium/>

##### 2. Motif locations <http://genome.grid.wayne.edu/centisnps/combo/>

##### 3. Open chromatin in ALL and LCL regions <http://genome.grid.wayne.edu/centisnps/test/>


### 1. SNP locations <http://genome.grid.wayne.edu/centisnps/compendium/>

##### Source Code: [./compendium/download_compendium.py](../compendium/download_compendium.py)

Execute following in bash terminal

```sh
python ./compendium/download_compendium.py
```
The python commands in this script are explained as follows:

List of files to download stored in ```file_names_combo.txt```
```python
f=open('file_names_combo.txt','r+')
file_lines=f.readlines()
```

Files take a long time to download, so if we don't have enough time to download all at once, save downloaded files
```python
sf_fn='saved_files.txt'
saved_files=open(sf_fn,'r+')
```
Compare both lists of files and obtain the remaining files to download
```python
remaining_to_download=[f for f in to_download if f not in saved_fn]
```
Now run thru the list of files in ```remaining_to_download``` using wget:
```python
for dl in remaining_to_download:
	print(home_directory_url+ dl)
	file_url=home_directory_url + dl
	wget.download(file_url, dl)
	saved_files.write(dl+'\n')
	num_already_dl+=1
	print('\n' + str(num_already_dl) + ' successfully downloaded of ' + str(len(to_download)))
```

Once we have all files, compile by calling the following script:

##### Source Code: [./compendium/compendium_data_compile.py](../compendium/compendium_data_compile.py)

```bash
python ./compendium/compendium_data_compile.py
```

### 2. Motif locations <http://genome.grid.wayne.edu/centisnps/combo/>

Combo reads are all pre-compiled into one file on the server:

<http://genome.grid.wayne.edu/centisnps/combo/footprints.all>

This file is 2.6gb and contains more records than all combo files downloaded together. 

Reconciliation of files: ...









##### Source Code: [./COMBO/download_combo.py](../COMBO/download_combo.py)


Execute following in bash terminal
```sh
python ./compendium/download_combo.py
```
This script works in a manner identical to compendium (explained above)


Once we have all files, compile by calling the following script:

##### Source Code: [./COMBO/combo_data_compile.py](../COMBO/combo_data_compile.py)

```bash
python ./compendium/compendium_data_compile.py
```

### 3. Open chromatin in ALL and LCL regions <http://genome.grid.wayne.edu/centisnps/test/>


##### Source Code: [./rawdata/compendium/download_compendium.py](../rawdata/download_bulk_data.py)


Execute following in bash terminal
```sh
python ./rawdata/download_bulk_data.py
```
This retrieves following files:
```python
#ALL reads
ALL_reads_fn='wgEncodeAwgDnaseUwdukeGm12878UniPk.narrowPeak'
wget.download(home_directory_url + ALL_reads_fn, ALL_reads_fn)
```
```python
wget.download(home_directory_url + LCL_reads_fn, LCL_reads_fn)
LCL_reads_fn='wgEncodeRegDnaseClusteredV3.bed'
```
