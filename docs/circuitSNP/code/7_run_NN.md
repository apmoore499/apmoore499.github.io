---
layout: default
title: 6. Run NN
nav_order: 6
parent: Python Implementation
---

##### [Back to contents](./..)


In the previous step, we split the data into train, validation and test

In this step, we will use the training and test data

And the goal is to build NN to predict whether chromatin open or closed based on which motifs are present in which window

Most of the code in this script was adapted from Denny's work:

<https://github.com/dennyshin/project_circuitSNP>

##### Source Code: [./run_DShin_NN.py](../run_DShin_NN.py)

Run python script in bash terminal

```sh
python ./run_DShin_NN.py
```

First load data and subset into X,Y partitions, then convert to numpy array

```python
#use this as lookup function for all present motifs
motif_names=['M' + str(index).zfill(5) for index in range(1,1373)]

#read in different data types
validation=pd.read_feather('validation_data.feather')
test=pd.read_feather('test.feather')
train=pd.read_feather('train.feather')

#training data
Xtrain=train[motif_names]
Xtrain=Xtrain.apply(pd.to_numeric).to_numpy()
Ytrain=train[['Y','Y_alt']].to_numpy()

#validation data
Xval=validation[motif_names]
Xval=Xval.apply(pd.to_numeric).to_numpy()
Yval=validation[['Y','Y_alt']].to_numpy()
```

Further conversion of data for torch processing

```python
#as we are using cuda to compute NN, set dtype as follows. This ensures it will run on GPU.
dtype = torch.cuda.FloatTensor
# convert data into torch tensors (so we can use pytorch to run)
Xtrain = torch.from_numpy(Xtrain).type(dtype)
Ytrain = torch.from_numpy(Ytrain).type(dtype)
Xval = torch.from_numpy(Xval).type(dtype)
Yval = torch.from_numpy(Yval).type(dtype)

```

Check proportions of open / closed chromatin are same for all data types

```python
#check proportions
for d in [train,validation,test]:
    print(str(d.Y.value_counts()[0]/d.Y.value_counts()[1]))
# all at ~11
```

```python
11.254522114599137
11.251305799364602
11.301038062283737
```

As expected, there is a good mixing of open / closed chromatin regions in all data types

Force torch to use GPU

```python
cuda = torch.device('cuda') 
```


Instantiate NN in Torch


```python
# build the neural net (D.Shin architecture)
class Net(nn.Module):
	def __init__(self, input_dim):
		super().__init__()
		self.fc1 = nn.Linear(input_dim, 5)
		self.fc2 = nn.Linear(5, 5)
		self.fc3 = nn.Linear(5, 2)
		# these are still random for each new model we create
		nn.init.xavier_uniform_(self.fc1.weight)
		nn.init.xavier_uniform_(self.fc2.weight)
		nn.init.xavier_uniform_(self.fc3.weight)
	def forward(self, x):
		x = F.relu(self.fc1(x))
		x = F.relu(self.fc2(x))
		# do not put relu on the last layer!
		return F.softmax(self.fc3(x), dim=1)


```


Below parameters copied from Denny

```python
# set model parameters (D.Shin)
# define my model
motif_num = Xtrain.shape[1] # this is now an int
model = Net(motif_num).to(cuda)

# choose optimizer
learning_rate = 0.0001
optimizer = optim.Adam(model.parameters(), lr=learning_rate)

# choose my criteria
criterion = nn.BCELoss()
```

Send data to GPU

```python
# send training data
Xtrain, Ytrain = Xtrain.to(cuda), Ytrain.to(cuda)
# send validation data
Xval, Yval = Xval.to(cuda), Yval.to(cuda)
```


Now train


```python
# train model (D.Shin)
# training and validation
n_epochs = 5000
epochs = list(range(1, n_epochs+1))
train_loss = []
val_loss = []
min_val_loss = 1
for epoch in epochs:
	model.train() # put the model in train mode
	optimizer.zero_grad() # null my gradients otherwise they will accumulate
	Ytrain_pred = model(Xtrain) # calculate my Y_hat
	loss = criterion(Ytrain_pred, Ytrain) # calculate my loss
	train_loss.append(loss)
	loss.backward(loss) # finds grad * loss (remember this is a weighted sum, where weight = loss)
	optimizer.step() # update my parameters
	model.eval()
	with torch.no_grad():
		Yval_pred = model(Xval)
	loss = criterion(Yval_pred, Yval)
	val_loss.append(loss)
	if epoch % 100 ==0 or epoch==1:
		print("epoch: ", epoch, f", val_loss: {loss.item(): f}")
	# save model with lowest validation loss
	if loss < min_val_loss:
		min_val_loss = loss
		torch.save(model, "trained_models/model.pt")
		opt_epoch = epoch
```

Results are validation loss ~0.28 (expected)
```python
epoch:  1 , val_loss:  0.734370
epoch:  100 , val_loss:  0.722462
epoch:  200 , val_loss:  0.710027
epoch:  300 , val_loss:  0.697230
epoch:  400 , val_loss:  0.684107
epoch:  500 , val_loss:  0.670686
epoch:  600 , val_loss:  0.656984
epoch:  700 , val_loss:  0.641809
epoch:  800 , val_loss:  0.626065
epoch:  900 , val_loss:  0.610111
epoch:  1000 , val_loss:  0.594016
epoch:  1100 , val_loss:  0.577875
epoch:  1200 , val_loss:  0.561785
epoch:  1300 , val_loss:  0.545837
epoch:  1400 , val_loss:  0.530124
epoch:  1500 , val_loss:  0.513478
epoch:  1600 , val_loss:  0.495988
epoch:  1700 , val_loss:  0.479464
epoch:  1800 , val_loss:  0.463758
epoch:  1900 , val_loss:  0.448854
epoch:  2000 , val_loss:  0.434746
epoch:  2100 , val_loss:  0.421437
epoch:  2200 , val_loss:  0.408921
epoch:  2300 , val_loss:  0.397194
epoch:  2400 , val_loss:  0.386238
epoch:  2500 , val_loss:  0.376032
epoch:  2600 , val_loss:  0.366554
epoch:  2700 , val_loss:  0.357778
epoch:  2800 , val_loss:  0.349675
epoch:  2900 , val_loss:  0.342214
epoch:  3000 , val_loss:  0.335366
epoch:  3100 , val_loss:  0.329100
epoch:  3200 , val_loss:  0.323383
epoch:  3300 , val_loss:  0.318180
epoch:  3400 , val_loss:  0.313462
epoch:  3500 , val_loss:  0.309198
epoch:  3600 , val_loss:  0.305361
epoch:  3700 , val_loss:  0.301920
epoch:  3800 , val_loss:  0.298849
epoch:  3900 , val_loss:  0.296123
epoch:  4000 , val_loss:  0.293713
epoch:  4100 , val_loss:  0.291595
epoch:  4200 , val_loss:  0.289744
epoch:  4300 , val_loss:  0.288132
epoch:  4400 , val_loss:  0.286742
epoch:  4500 , val_loss:  0.285551
epoch:  4600 , val_loss:  0.284538
epoch:  4700 , val_loss:  0.283684
epoch:  4800 , val_loss:  0.282968
epoch:  4900 , val_loss:  0.282376
epoch:  5000 , val_loss:  0.281892
```

So in summary after ~5000 epochs, validation loss at around 0.282
 

##### [Back to contents](./..)