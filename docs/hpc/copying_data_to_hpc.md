

##### Copying data

In order to copy data, use following syntax:


```sh
scp ./3_train_data.feather apmoore@spartan.hpc.unimelb.edu.au:~/circuitSNP/data/3_train_data.feather
```

first argument: source file
second argument: destination file (ssh via personal acct on spartan hpc cluster)

##### Misc Other HPCs

An SSH config file will also make your life easier. It allows you to create alises (i.e. shortcuts) for a given hostname.

Create the text file in your ~/.ssh directory with your preferred text editor, for example, nano.
```sh
nano .ssh/config
```




Enter the following (replacing username with your actual username of course!):
```sh
Host *
ServerAliveInterval 120
Host spartan
       Hostname spartan.hpc.unimel
```

##### Load tflow - cpu

Module is called: 
````sh
Tensorflow-CPU/1.15.0-spartan_intel-2017.u2-Python-3.6.4
````

