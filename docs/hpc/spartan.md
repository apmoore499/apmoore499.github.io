---
layout: default
title: Spartan HPC Usage
nav_order: 3
has_children: true
---

## Some notes on Spartan / HPC usage

### Logging in

Use ```ssh <username>@spartan.hpc.unimelb.edu.au``` to login via ssh terminal in linux/mac or else use PUTTY on windows machine

### Starting session or running whatever

When using spartan make sure to run code from a session - the default terminal interface runs on the login node and this is a common landing area for new users.

```sinteractive``` session very useful for debugging. Once verified to run on this node, can batch to run on hpc cluster via slurm

Start up the unit with following, for eg 20 mins. Change acct as necessary.
```sh
sinteractive --nodes 1 --account=punim0614 --partition gpgpu --qos=gpgpumdhs --gres=gpu:p100:1 --time 01:00:00 --cpus-per-task=1
```

For 1hr
```sh
sinteractive --nodes 1 --account=punim0614 --partition gpgpu --qos=gpgpumdhs --gres=gpu:p100:1 --time 01:00:00 --cpus-per-task=1
```

To specify more memory (40000MB in this case). If you don't have enough memory sometimes you will get 'process killed' error.
```sh
sinteractive --nodes 1 --account=punim0614 --partition gpgpu --qos=gpgpumdhs --mem 40000 --gres=gpu:p100:1 --time 01:00:00 --cpus-per-task=1
```


# Loading python modules


This is order of operation to set up environment for example relating to analysis for circuitSNP. The last command called in bash will overwrite any applicable libraries imported by other commands. For example, if importing python3.6.4 kernel, will backdate pandas to earlier version than that used in python3.6.10 kernel, but libraries not common to both will be left unaltered. This is particularly relevant for protobuf library which differs between various tensorflow versions, and the verison of pandas - as .pickle objects (serialised using pickle library in python) will not be compatible if the version of pandas used to save object is different from the verison used to load.




1. load python module (you would only load one python module)
```sh
#for python with CUDA9
module load Python/3.6.4-intel-2017.u2-GCC-6.2.0-CUDA9
#for python with CUDA10
module load Python/3.6.10-intel-2017.u2-GCC-6.2.0-CUDA10.1
```

2. then load tensorflow
```sh
#for cuda9
module load Tensorflow/2.1.0-intel-2017.u2-GCC-6.2.0-CUDA9-Python-3.6.4
#for cuda10
module load Tensorflow/2.1.0-intel-2017.u2-GCC-6.2.0-CUDA10.1-Python-3.6.10-GPU
```

3. then load virutalenv
```sh
cd circuitSNP
source ./p6/bin/activate  #in circuitSNP directory...
```

All togeth for cuda9
```sh
module load Python/3.6.4-intel-2017.u2-GCC-6.2.0-CUDA9
module load Tensorflow/2.1.0-intel-2017.u2-GCC-6.2.0-CUDA9-Python-3.6.4
cd circuitSNP
source ./p6/bin/activate  #in circuitSNP directory...
```

And for cuda10

```sh
module load Python/3.6.10-intel-2017.u2-GCC-6.2.0-CUDA10.1
module load Tensorflow/2.1.0-intel-2017.u2-GCC-6.2.0-CUDA10.1-Python-3.6.10-GPU
source ~/circuitSNP/p6c10/bin/activate  #in circuitSNP directory...
```

# Virtual environment

To create new virtual env do following (note change python version if need). Calling venv will attempt to load that python module.

It's important to call ```module load web_proxy``` so that the server can surf the web and download more modules

```sh
module load Python/3.6.4-intel-2017.u2-GCC-6.2.0-CUDA9
module load web_proxy
virtualenv <venv name>F
```

To install libraries for each cuda version: 

##### Cuda9 
```sh
module load Python/3.6.4-intel-2017.u2-GCC-6.2.0-CUDA9
module load web_proxy
virtualenv
source venv/bin/activate
module load web_proxy
pip install pandas==0.22.0
pip install protobuf==3.8.0
```
 
##### Cuda10
```sh
module load Python/3.6.10-intel-2017.u2-GCC-6.2.0-CUDA10.1
module load web_proxy
virtualenv p6c10
source p6c10/bin/activate
pip install absl-py
pip install scipy==1.4.1
pip install pandas==0.22.0
pip install matplotlib
```

To double check compatibility of various python modules, call following:

```sh
pip freeze
```

##### GPU usage w tensorflow

If using cuda10, we need to define model on the gpu device, otherwise tflow might run on CPU.

Force GPU usage using flag ```with tf.device('/device:GPU:0'):``` then that instance of ```tf``` refers to the GPU version of tflow kernel

```python

from tensorflow.python.client import device_lib
print(device_lib.list_local_devices())

#returns device as '/device:GPU:0'

NUMBER_OF_EPOCHS=500
with tf.device('/device:GPU:0'):
  Xtrain_v=tf.convert_to_tensor(Xtrain_v)
  Ytrain_v=tf.convert_to_tensor(Ytrain_v)
  Xval_v=tf.convert_to_tensor(Xval_v)
  Yval_v=tf.convert_to_tensor(Yval_v)
  Xtest_v=tf.convert_to_tensor(Xtest_v)
  Ytest_v=tf.convert_to_tensor(Ytest_v) 
  model1 = tf.keras.Sequential() #model defined on GPU
  model1.add(tf.keras.layers.Flatten(input_shape=(1, 1372)))
  model1.add(tf.keras.layers.BatchNormalization())
  model1.add(tf.keras.layers.Dense(5, activation='relu'))
  model1.add(tf.keras.layers.BatchNormalization())
  model1.add(tf.keras.layers.Dense(units=3, activation='relu'))
  model1.add(tf.keras.layers.BatchNormalization())
  model1.add(tf.keras.layers.Dense(units=1, activation='sigmoid'))
  model1.compile(optimizer='adagrad',loss='binary_crossentropy',metrics=['accuracy'])
  history1 = model1.fit(x=Xtrain_v,y=Ytrain_v, epochs=NUMBER_OF_EPOCHS, batch_size=128,validation_data=(Xval_v,Yval_v))
  model1.save('./models/model_batch_norm_every_5_3_128_.h5')
```





#####Misc / copying files

Copying files have following commands:


```sh
scp <from_dir> <to_dir>
```

Example on relevant data:

```
scp /home/doctorjonescore/Desktop/cdata/3_test_data.pickle apmoore@spartan.hpc.unimelb.edu.au:./circuitSNP/data/3_test_data.pickle
scp /home/doctorjonescore/Desktop/cdata/3_train_data.pickle apmoore@spartan.hpc.unimelb.edu.au:./circuitSNP/data/3_train_data.pickle
scp /home/doctorjonescore/Desktop/cdata/3_validation_data.pickle apmoore@spartan.hpc.unimelb.edu.au:./circuitSNP/data/3_validation_data.pickle
```


#####Slurm 

Batch files using slurm, here example

```
---------------
#!/bin/bash
#SBATCH --nodes 1
#SBATCH --account=punim0614
#SBATCH --qos=gpgpumdhs
#SBATCH --partition gpgpu
#SBATCH --gres=gpu:p100:1
#SBATCH --time 00:05:00
#SBATCH --cpus-per-task=1

module load Tensorflow/1.8.0-intel-2017.u2-GCC-6.2.0-CUDA9-Python-3.5.2-GPU
python tensor_flow.py
-------------
```


