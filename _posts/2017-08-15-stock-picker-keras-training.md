---
layout:     post
title:      Stock Picker Keras Training
date:       2017-08-15 09:32:19
summary:    To train the stock picker, I'll used a deep learning neural network model.  There are many machine learning frameworks that support Nvidia's GPUs and I settled on using Keras with Tensorflow.
categories: machine-learning
---

### Using Keras to Train a Stock Picker

To train the stock picker, I'll used a deep learning neural network model.  There are many machine learning frameworks that support Nvidia's GPUs (see [Deep Learning Frameworks](https://developer.nvidia.com/deep-learning-frameworks)), but I settled on using Keras with Tensorflow.  I tried stand-alone Tensorflow, CNTK and Caffe, but found Keras the easiest to use without sacrificing much performance.  Keras also allowed me to easily adjust the batch size and training sample randomization.

In my [previous post](/machine-learning/stock-fundamentals-training-data/), I talked about building the stock fundamentals data training and test set.  Below, I'll describe a bit how I used the data set for training using Keras and Tensorflow.  Note that you can get started with Keras using one of their excellent [tutorials](https://blog.keras.io/category/tutorials.html).

First, I needed to define the columns from the fundamental data set I would be creating.  Note that I added some features related to short interest from another [data source](http://shortsqueeze.com/). I also added a column called `beat_sp500` that was set to `1` if it beat the S&P 500 by 2 percentage points for a 6 month time period.  

```python
COLUMNS = ["symbol","sector","market_cap","pb","de_ratio",
           "div_yield","ebitda_margin","pe","ev_ebitda",
           "p_fc","net_margin","ps","roa","roic",
           "p_cashneq","div_payout","sortino",
           "market_cap_n","short_ratio_of_float",
           "short_interest_period_change","days_to_cover",
           "turnover_ratio""total_return","beat_sp500"
           ]

LABEL_COLUMN = "beat_sp500"

```

Next, the training and test sets needed to be loaded into a Pandas dataframe:

```python
  df_train = pd.read_csv(
      train_file_name,
      names=COLUMNS,
      skipinitialspace=True,
      skiprows=1,      
      engine="python")
  df_test = pd.read_csv(
      test_file_name,
      names=COLUMNS,
      skipinitialspace=True,
      skiprows=1,
      engine="python")
```

After dataframes are created, the labels and columns need to be extracted:

```python
train_x, train_y = get_features_labels(df_train, features)
test_x, test_y = get_features_labels(df_test, features)
```

Next, a three layer dense 30-60-30 neural network was created.  I found that I needed to tweak the size to find the best performance to percision balance.

```
# define baseline model
def baseline_model(num_features):
  # create model
  model = Sequential()
  model.add(Dense(30, input_dim=num_features, 
            activation='relu'))
  model.add(Dense(60, activation='relu'))
  model.add(Dense(30,activation='relu'))
  model.add(Dense(1, activation='sigmoid'))
  # Compile model
  model.compile(loss='binary_crossentropy', 
                optimizer='adam', 
                metrics=['accuracy', precision])
  return model


model = baseline_model(num_features)
```

Finally, train the model and evaluate it:

```python
model.fit(train_x, train_y, epochs=train_steps, 
          batch_size=batch_size, shuffle=True, verbose=0)
score = model.evaluate(test_x, test_y, 
                       batch_size=test_batch_size, verbose=0)
```

Note that if the data set is purely a time series set shuffle to `False`.

I plan to discuss the results in a future post in detail.  For now, I can say that the stock picker was able to choose some stocks that outperformed the S&P 500 in the test set, however the performance gain wasn't significant enough to justify the risk of holding individual stocks.  Also it's challenging to evaluate how the model would work for future stock picks due to the long time horizon.