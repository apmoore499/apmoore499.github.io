---
layout: default
title: 1. Environment Configuration
nav_order: 1
parent: Python Implementation
---




## Machine info setup - home

Code executed on ubuntu machine with nVidia GTX1080-Ti for graphics-accelerated machine learning. 

To set up environment with GPU support (required to run python scripts), need to call the following commands in order in bash terminal. 

2 commands below from instructions at <https://towardsdatascience.com/tensorflow-gpu-installation-made-easy-use-conda-instead-of-pip-52e5249374bc>

Install Conda to enable graphics card support
```sh
sudo apt-get install conda
```
Install GPU support
```sh
conda create --name tf_gpu tensorflow-gpu 
```
Check that python version is as required (3.7 or greater) 
```sh
python -v
```
Output on this machine satisfies this requirement:
```sh
Python 3.7.4 (default, Aug 13 2019, 20:35:49) 
```
Install pip package manager for python
```sh
sudo apt install python3-venv python3-pip
```
Install requirements for python
```sh
pip3 install -r ./requirements.txt
```
