
## Now we split data


#### In prev step, we created entire T matrix, now in this python file we create validation/test/train data from this. T_matrix is class, and we have to define N_OPEN or N_CLOSED, and we need to define validation / train percentage. We can set RANDOM_SEED to make our random selection deterministic

### First randomly select from open and closed windows. 

```python
self.sample_open(all_windows)
self.sample_closed(all_windows)
```

### Combine the open and closed selections 

```python
self.combine_T()
```

### Split into train val and test data

```python
self.split_data(train_pc,val_pc)
```

### Assign train / test / val categories to data
```python
self.assign_splits()
```

### The usage of this is that we call the class, and we can call it using a random seed, and then we can make the dataset deterministic