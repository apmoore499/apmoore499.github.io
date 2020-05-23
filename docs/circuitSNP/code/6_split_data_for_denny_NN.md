---
layout: default
---

##### [Back to contents](./..)


In the previous step, we generated T matrix as input for NN

In this step, we will use the T matrix as input data

And the goal is to split the data into three types useful for the neural net: train, validation and test data

##### Source Code: [./split_data_for_Denny_NN.py](../split_data_for_Denny_NN.py)

Run python script in bash terminal:

```sh
python ./split_data_for_Denny_NN.py
```


First load data:

```python
t_matrix=pd.read_feather('T_matrix.feather')
```

Generate alternate column so that we can set target vector as 1-hot vector

```python
def alternate(val):
	if int(val)==1:
		return 0
	else:
		return 1

#append extra row for output
#so that target will be 1-hot vector eg:
# [1] = [1,0]
# [0] = [0,1]	
T_matrix['Y_alt']=T_matrix['Y']
T_matrix['Y_alt'] = T_matrix['Y_alt'].apply(alternate)
```

Now split into three different data types (validation, train, test) and save out
```python
train=T_matrix.loc[T_matrix['data_type']=='TRAINING_DATA'].reset_index().drop('data_type',axis=1)
test=T_matrix.loc[T_matrix['data_type']=='TEST_DATA'].reset_index().drop('data_type',axis=1)
validation=T_matrix.loc[T_matrix['data_type']=='VALIDATION_DATA'].reset_index().drop('data_type',axis=1)

validation.to_feather('validation_data.feather')
test.to_feather('test.feather')
train.to_feather('train.feather')
```

##### [Back to contents](./..)