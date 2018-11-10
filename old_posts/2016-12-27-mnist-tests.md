---
id: 216
title: Digit Recognition - 2
author: rahular
layout: post
excerpt: Tests I carried out on MNIST dataset. Includes a single layer fully connected neural network, a basic CNN with and without regularization. I ultimately end up with a validation accuracy of 99.55%
guid: http://rahular.com/?p=216
permalink: /digit-recognition-2/
categories:
  - Deep Learning
  - Machine Learning
  - Convolutional Neural Networks
  - CNN
  - Batch Normalization
  - Dropout
  - MNIST
  - Keras
  - Python
  - Tensorflow
---
If you are looking for a SVM based approach, see [here](http://rahular.com/digit-recognition-1). 

MNIST, by far is one of the most used toy datasets in machine learning. It contains 70,000 images of handwritten digits. The standard train-test split is 60,000-10,000. More details can be found [here](http://yann.lecun.com/exdb/mnist/ "MNIST homepage").

In this notebook, I show how applying very basic algorithms to this dataset gives surprisingly good results. It is always recommended to start off simple even on more complex datasets. Though simple models may not learn well if the data is complex, having a notebook like this as a starting point helps a lot in identifying the areas which need improvement. Also, the underlying process of experimentation remains the same.

Libraries used:
 
* Keras (with Tensorflow backend)
* IPython notebook
* Numpy (obviously!)

Here is a brief outline about the next sections:

1. Pre-processing of data: normalization and one-hot encoding the outputs
2. Augmenting the dataset with artificially generated data
3. Training a linear model: single layer fully connected neural network
4. Training a small CNN: A network similar to VGG16 in terms of architecture
5. Adding regularization: Dropout and Batch Normalization

As always, what follows is mostly code. I have tried to add some explanations along the way :)

## Pre-processing

Get MNIST test and train sets and see their dimensions


```python
from keras.datasets import mnist
(train_x, train_y), (test_x, test_y) = mnist.load_data()

print train_x.shape, train_y.shape
print test_x.shape, test_y.shape
```

    Using TensorFlow backend.


    (60000, 28, 28) (60000,)
    (10000, 28, 28) (10000,)


Add a `channel` dimension to this data as Keras expects it later when we reach CNNs. Since I use a `tf` backend, the `channel` is my 3rd dimension


```python
import numpy as np

train_x = np.expand_dims(train_x, 3)
test_x = np.expand_dims(test_x, 3)

print train_x.shape
print test_x.shape
```

    (60000, 28, 28, 1)
    (10000, 28, 28, 1)


We also need to convert the labels into one hot encodings


```python
from keras.utils.np_utils import to_categorical

print 'Before one hot encoding: ', train_y[5]

train_y = to_categorical(train_y)
test_y = to_categorical(test_y)

print 'After one hot encoding: ', train_y[5]
```

    Before one hot encoding:  2
    After one hot encoding:  [ 0.  0.  1.  0.  0.  0.  0.  0.  0.  0.]


Create a method which takes a data point as input and returns it's normalized value


```python
mean = train_x.mean().astype(np.float32)
std = train_x.std().astype(np.float32)

def norm_input(x):
    return (x - mean) / std
```

Create train and test batches **with** and **without** data augmentation. Note that we can perform normalization easily with the `ImageGenerator` itself, rather than doing it manually (as above)


```python
from keras.preprocessing import image

# without augmentation
generator = image.ImageDataGenerator()
train_batches = generator.flow(train_x, train_y, batch_size=64)
test_batches = generator.flow(test_x, test_y, batch_size=64)

# with augmentation
da_generator = image.ImageDataGenerator(rotation_range=8, width_shift_range=0.08, shear_range=0.3, 
                           height_shift_range=0.08, zoom_range=0.08)
da_train_batches = da_generator.flow(train_x, train_y, batch_size=64)
da_test_batches = da_generator.flow(test_x, test_y, batch_size=64)
```

Let us write a simple training method which does the following:

* runs a single epoch with the default learning rate (0.01)
* raise the learning rate to 0.1 and run for 5 epochs
* lower the learning rate back to 0.01 and run for 5 epochs


```python
def train(model, train, test):
    model.fit_generator(generator=train, samples_per_epoch=train.N, nb_epoch=1, 
                           validation_data=test, nb_val_samples=test.N)
    model.optimizer.lr = 0.1
    model.fit_generator(generator=train, samples_per_epoch=train.N, nb_epoch=5, 
                           validation_data=test, nb_val_samples=test.N)
    model.optimizer.lr = 0.01
    model.fit_generator(generator=train, samples_per_epoch=train.N, nb_epoch=5, 
                           validation_data=test, nb_val_samples=test.N) 
```

## Linear model

Create a simple fully connected network with a single hidden layer. The hidden layer has 10 neurons. We use standard `SGD` for optimization and `categorical crossentropy` loss function.

Lamda layer allows us to pass an arbitrary function which is applied on each data point.

Flatten layer just squashes the data into a single dimension.


```python
from keras.models import Sequential
from keras.layers import Dense, Lambda, Flatten
from keras.optimizers import sgd

def get_linear_model():
    model = Sequential()
    model.add(Lambda(norm_input, input_shape=(28, 28, 1)))
    model.add(Flatten())
    model.add(Dense(10, activation='softmax'))
    model.compile(sgd(), loss='categorical_crossentropy', metrics=['accuracy'])

    return model
```

### Linear model without augmentation


```python
linear_model = get_linear_model()
train(linear_model, train_batches, test_batches)
```

    Epoch 1/1
    60000/60000 [==============================] - 7s - loss: 0.5115 - acc: 0.8513 - val_loss: 0.3399 - val_acc: 0.9037
    Epoch 1/5
    60000/60000 [==============================] - 7s - loss: 0.3418 - acc: 0.9015 - val_loss: 0.3137 - val_acc: 0.9093
    Epoch 2/5
    60000/60000 [==============================] - 7s - loss: 0.3198 - acc: 0.9081 - val_loss: 0.2976 - val_acc: 0.9147
    Epoch 3/5
    60000/60000 [==============================] - 7s - loss: 0.3055 - acc: 0.9133 - val_loss: 0.2921 - val_acc: 0.9152
    Epoch 4/5
    60000/60000 [==============================] - 7s - loss: 0.2975 - acc: 0.9155 - val_loss: 0.2873 - val_acc: 0.9175
    Epoch 5/5
    60000/60000 [==============================] - 7s - loss: 0.2922 - acc: 0.9166 - val_loss: 0.2885 - val_acc: 0.9186
    Epoch 1/5
    60000/60000 [==============================] - 7s - loss: 0.2876 - acc: 0.9186 - val_loss: 0.2875 - val_acc: 0.9174
    Epoch 2/5
    60000/60000 [==============================] - 7s - loss: 0.2849 - acc: 0.9194 - val_loss: 0.2825 - val_acc: 0.9229
    Epoch 3/5
    60000/60000 [==============================] - 7s - loss: 0.2814 - acc: 0.9214 - val_loss: 0.2737 - val_acc: 0.9216
    Epoch 4/5
    60000/60000 [==============================] - 7s - loss: 0.2782 - acc: 0.9227 - val_loss: 0.2780 - val_acc: 0.9202
    Epoch 5/5
    60000/60000 [==============================] - 7s - loss: 0.2731 - acc: 0.9231 - val_loss: 0.2732 - val_acc: 0.9251


### Linear model with augmentation


```python
da_linear_model = get_linear_model()
train(da_linear_model, da_train_batches, da_test_batches)
```

    Epoch 1/1
    60000/60000 [==============================] - 7s - loss: 1.1433 - acc: 0.6333 - val_loss: 0.9586 - val_acc: 0.7045
    Epoch 1/5
    60000/60000 [==============================] - 8s - loss: 0.9645 - acc: 0.7021 - val_loss: 0.9101 - val_acc: 0.7168
    Epoch 2/5
    60000/60000 [==============================] - 7s - loss: 0.9418 - acc: 0.7100 - val_loss: 0.9046 - val_acc: 0.7242
    Epoch 3/5
    60000/60000 [==============================] - 7s - loss: 0.9307 - acc: 0.7167 - val_loss: 0.8987 - val_acc: 0.7314
    Epoch 4/5
    60000/60000 [==============================] - 7s - loss: 0.9301 - acc: 0.7180 - val_loss: 0.8909 - val_acc: 0.7285
    Epoch 5/5
    60000/60000 [==============================] - 7s - loss: 0.9183 - acc: 0.7222 - val_loss: 0.8600 - val_acc: 0.7375
    Epoch 1/5
    60000/60000 [==============================] - 7s - loss: 0.9138 - acc: 0.7227 - val_loss: 0.8701 - val_acc: 0.7423
    Epoch 2/5
    60000/60000 [==============================] - 7s - loss: 0.9080 - acc: 0.7246 - val_loss: 0.8778 - val_acc: 0.7394
    Epoch 3/5
    60000/60000 [==============================] - 7s - loss: 0.9091 - acc: 0.7271 - val_loss: 0.8683 - val_acc: 0.7401
    Epoch 4/5
    60000/60000 [==============================] - 7s - loss: 0.9122 - acc: 0.7251 - val_loss: 0.8657 - val_acc: 0.7360
    Epoch 5/5
    60000/60000 [==============================] - 7s - loss: 0.9075 - acc: 0.7256 - val_loss: 0.8810 - val_acc: 0.7361


## VGG-like CNN model

CNN, which specialize in image data, is expected to out-perform the previous model. Let's see by how much!


```python
from keras.models import Sequential
from keras.layers import Dense, Lambda, Flatten, Convolution2D, MaxPooling2D
from keras.optimizers import sgd

def get_vgg_model():
    model = Sequential()
    model.add(Lambda(norm_input, input_shape=(28, 28, 1)))
    model.add(Convolution2D(32, 3, 3, activation='relu'))
    model.add(Convolution2D(32, 3, 3, activation='relu'))
    model.add(MaxPooling2D())
    model.add(Convolution2D(64, 3, 3, activation='relu'))
    model.add(Convolution2D(64, 3, 3, activation='relu'))
    model.add(MaxPooling2D())
    model.add(Flatten())
    model.add(Dense(512, activation='relu'))
    model.add(Dense(10, activation='softmax'))
    model.compile(sgd(), loss='categorical_crossentropy', metrics=['accuracy'])

    return model
```

### CNN without augmentation


```python
vgg_model = get_vgg_model()
train(vgg_model, train_batches, test_batches)
```

    Epoch 1/1
    60000/60000 [==============================] - 84s - loss: 1.3413 - acc: 0.5329 - val_loss: 0.1907 - val_acc: 0.9452
    Epoch 1/5
    60000/60000 [==============================] - 85s - loss: 0.1524 - acc: 0.9533 - val_loss: 0.1001 - val_acc: 0.9685
    Epoch 2/5
    60000/60000 [==============================] - 85s - loss: 0.0948 - acc: 0.9714 - val_loss: 0.0678 - val_acc: 0.9776
    Epoch 3/5
    60000/60000 [==============================] - 86s - loss: 0.0725 - acc: 0.9774 - val_loss: 0.0665 - val_acc: 0.9790
    Epoch 4/5
    60000/60000 [==============================] - 87s - loss: 0.0575 - acc: 0.9819 - val_loss: 0.0563 - val_acc: 0.9819
    Epoch 5/5
    60000/60000 [==============================] - 84s - loss: 0.0486 - acc: 0.9849 - val_loss: 0.0470 - val_acc: 0.9841
    Epoch 1/5
    60000/60000 [==============================] - 86s - loss: 0.0435 - acc: 0.9864 - val_loss: 0.0442 - val_acc: 0.9853
    Epoch 2/5
    60000/60000 [==============================] - 85s - loss: 0.0361 - acc: 0.9887 - val_loss: 0.0441 - val_acc: 0.9858
    Epoch 3/5
    60000/60000 [==============================] - 85s - loss: 0.0317 - acc: 0.9899 - val_loss: 0.0399 - val_acc: 0.9857
    Epoch 4/5
    60000/60000 [==============================] - 83s - loss: 0.0283 - acc: 0.9911 - val_loss: 0.0438 - val_acc: 0.9856
    Epoch 5/5
    60000/60000 [==============================] - 85s - loss: 0.0248 - acc: 0.9919 - val_loss: 0.0464 - val_acc: 0.9860


By now, one should have observed that the training accuracy is slightly higher than the validation accuracy which is an indication of **over fitting**. We will address that in the next section

### CNN with augmentation


```python
da_vgg_model = get_vgg_model()
train(da_vgg_model, da_train_batches, da_test_batches)
```

    Epoch 1/1
    60000/60000 [==============================] - 84s - loss: 1.9180 - acc: 0.3261 - val_loss: 0.6366 - val_acc: 0.7977
    Epoch 1/5
    60000/60000 [==============================] - 85s - loss: 0.3559 - acc: 0.8900 - val_loss: 0.2062 - val_acc: 0.9375
    Epoch 2/5
    60000/60000 [==============================] - 84s - loss: 0.2029 - acc: 0.9371 - val_loss: 0.1989 - val_acc: 0.9349
    Epoch 3/5
    60000/60000 [==============================] - 82s - loss: 0.1558 - acc: 0.9524 - val_loss: 0.1327 - val_acc: 0.9571
    Epoch 4/5
    60000/60000 [==============================] - 82s - loss: 0.1317 - acc: 0.9584 - val_loss: 0.1093 - val_acc: 0.9648
    Epoch 5/5
    60000/60000 [==============================] - 84s - loss: 0.1167 - acc: 0.9645 - val_loss: 0.1002 - val_acc: 0.9672
    Epoch 1/5
    60000/60000 [==============================] - 86s - loss: 0.1051 - acc: 0.9671 - val_loss: 0.0934 - val_acc: 0.9708
    Epoch 2/5
    60000/60000 [==============================] - 87s - loss: 0.0964 - acc: 0.9698 - val_loss: 0.0999 - val_acc: 0.9692
    Epoch 3/5
    60000/60000 [==============================] - 86s - loss: 0.0885 - acc: 0.9728 - val_loss: 0.0906 - val_acc: 0.9716
    Epoch 4/5
    60000/60000 [==============================] - 86s - loss: 0.0824 - acc: 0.9738 - val_loss: 0.0717 - val_acc: 0.9771
    Epoch 5/5
    60000/60000 [==============================] - 87s - loss: 0.0789 - acc: 0.9745 - val_loss: 0.0738 - val_acc: 0.9756


## VGG-like CNN model with regularization

The purpose of Regularization is to avoid over-fitting. Here we apply:

* **Dropout**: randomly drop 50% (`p=0.5`) of the inputs coming in from the previous layer before sending it to the next layer
* **Batch Normalization**: Normalize the inputs coming from the previous layer by pushing their mean close to `0` and standard deviation close to `1`

For sake of demonstration, we will only consider un-augmented data from here on.

### Dropout


```python
from keras.models import Sequential
from keras.layers import Dense, Lambda, Flatten, Convolution2D, MaxPooling2D, Dropout
from keras.optimizers import sgd

def get_dropout_vgg_model():
    model = Sequential()
    model.add(Lambda(norm_input, input_shape=(28, 28, 1)))
    model.add(Convolution2D(32, 3, 3, activation='relu'))
    model.add(Convolution2D(32, 3, 3, activation='relu'))
    model.add(MaxPooling2D())
    model.add(Dropout(0.5))
    model.add(Convolution2D(64, 3, 3, activation='relu'))
    model.add(Convolution2D(64, 3, 3, activation='relu'))
    model.add(MaxPooling2D())
    model.add(Dropout(0.5))
    model.add(Flatten())
    model.add(Dense(512, activation='relu'))
    model.add(Dense(10, activation='softmax'))
    model.compile(sgd(), loss='categorical_crossentropy', metrics=['accuracy'])

    return model
```


```python
dropout_vgg_model = get_dropout_vgg_model()
train(dropout_vgg_model, train_batches, test_batches)
```

    Epoch 1/1
    60000/60000 [==============================] - 93s - loss: 2.2000 - acc: 0.1931 - val_loss: 0.8338 - val_acc: 0.7178
    Epoch 1/5
    60000/60000 [==============================] - 89s - loss: 0.4403 - acc: 0.8606 - val_loss: 0.1492 - val_acc: 0.9556
    Epoch 2/5
    60000/60000 [==============================] - 89s - loss: 0.2202 - acc: 0.9334 - val_loss: 0.1008 - val_acc: 0.9692
    Epoch 3/5
    60000/60000 [==============================] - 88s - loss: 0.1682 - acc: 0.9483 - val_loss: 0.0831 - val_acc: 0.9746
    Epoch 4/5
    60000/60000 [==============================] - 87s - loss: 0.1443 - acc: 0.9555 - val_loss: 0.0641 - val_acc: 0.9791
    Epoch 5/5
    60000/60000 [==============================] - 87s - loss: 0.1250 - acc: 0.9607 - val_loss: 0.0564 - val_acc: 0.9815
    Epoch 1/5
    60000/60000 [==============================] - 86s - loss: 0.1126 - acc: 0.9652 - val_loss: 0.0519 - val_acc: 0.9824
    Epoch 2/5
    60000/60000 [==============================] - 88s - loss: 0.1040 - acc: 0.9677 - val_loss: 0.0436 - val_acc: 0.9855
    Epoch 3/5
    60000/60000 [==============================] - 90s - loss: 0.0956 - acc: 0.9699 - val_loss: 0.0451 - val_acc: 0.9859
    Epoch 4/5
    60000/60000 [==============================] - 87s - loss: 0.0893 - acc: 0.9706 - val_loss: 0.0386 - val_acc: 0.9871
    Epoch 5/5
    60000/60000 [==============================] - 87s - loss: 0.0832 - acc: 0.9738 - val_loss: 0.0390 - val_acc: 0.9878


### Dropout + Batch Normalization


```python
from keras.models import Sequential
from keras.layers import Dense, Lambda, Flatten, Convolution2D, MaxPooling2D, Dropout, BatchNormalization
from keras.optimizers import sgd

def get_bn_dropout_vgg_model():
    model = Sequential()
    model.add(Lambda(norm_input, input_shape=(28, 28, 1)))
    model.add(Convolution2D(32, 3, 3, activation='relu'))
    model.add(BatchNormalization(axis=3))
    model.add(Convolution2D(32, 3, 3, activation='relu'))
    model.add(BatchNormalization(axis=3))
    model.add(MaxPooling2D())
    model.add(Convolution2D(64, 3, 3, activation='relu'))
    model.add(BatchNormalization(axis=3))
    model.add(Convolution2D(64, 3, 3, activation='relu'))
    model.add(BatchNormalization(axis=3))
    model.add(MaxPooling2D())
    model.add(Flatten())
    model.add(Dense(512, activation='relu'))
    model.add(BatchNormalization())
    model.add(Dropout(0.5))
    model.add(Dense(10, activation='softmax'))
    model.compile(sgd(), loss='categorical_crossentropy', metrics=['accuracy'])

    return model
```


```python
bn_dropout_vgg_model = get_bn_dropout_vgg_model()
train(bn_dropout_vgg_model, train_batches, test_batches)
```

    Epoch 1/1
    60000/60000 [==============================] - 177s - loss: 0.1690 - acc: 0.9485 - val_loss: 0.0366 - val_acc: 0.9871
    Epoch 1/5
    60000/60000 [==============================] - 180s - loss: 0.0581 - acc: 0.9820 - val_loss: 0.0292 - val_acc: 0.9897
    Epoch 2/5
    60000/60000 [==============================] - 185s - loss: 0.0424 - acc: 0.9867 - val_loss: 0.0220 - val_acc: 0.9926
    Epoch 3/5
    60000/60000 [==============================] - 184s - loss: 0.0365 - acc: 0.9886 - val_loss: 0.0206 - val_acc: 0.9935
    Epoch 4/5
    60000/60000 [==============================] - 182s - loss: 0.0281 - acc: 0.9910 - val_loss: 0.0201 - val_acc: 0.9932
    Epoch 5/5
    60000/60000 [==============================] - 177s - loss: 0.0258 - acc: 0.9919 - val_loss: 0.0189 - val_acc: 0.9938
    Epoch 1/5
    60000/60000 [==============================] - 177s - loss: 0.0222 - acc: 0.9932 - val_loss: 0.0181 - val_acc: 0.9937
    Epoch 2/5
    60000/60000 [==============================] - 177s - loss: 0.0197 - acc: 0.9939 - val_loss: 0.0162 - val_acc: 0.9944
    Epoch 3/5
    60000/60000 [==============================] - 177s - loss: 0.0165 - acc: 0.9949 - val_loss: 0.0195 - val_acc: 0.9929
    Epoch 4/5
    60000/60000 [==============================] - 178s - loss: 0.0154 - acc: 0.9955 - val_loss: 0.0182 - val_acc: 0.9939
    Epoch 5/5
    60000/60000 [==============================] - 178s - loss: 0.0150 - acc: 0.9952 - val_loss: 0.0216 - val_acc: 0.9928


We can see the 4th epoch went up to **0.9939** and came back down in the 5th. Therefore, it is always a good idea to save the weights after every epoch and use the one which gave the best validation accuracy.

In the hope of pushing it back up just a little bit more, let's decay the learning rate some more and train it for 5 additional epochs


```python
bn_dropout_vgg_model.optimizer.lr = 0.001
bn_dropout_vgg_model.fit_generator(generator=train_batches, samples_per_epoch=train_batches.N, nb_epoch=5, 
                       validation_data=test_batches, nb_val_samples=test_batches.N) 
```

    Epoch 1/5
    60000/60000 [==============================] - 175s - loss: 0.0125 - acc: 0.9963 - val_loss: 0.0187 - val_acc: 0.9931
    Epoch 2/5
    60000/60000 [==============================] - 175s - loss: 0.0116 - acc: 0.9962 - val_loss: 0.0136 - val_acc: 0.9955
    Epoch 3/5
    60000/60000 [==============================] - 175s - loss: 0.0104 - acc: 0.9968 - val_loss: 0.0190 - val_acc: 0.9938
    Epoch 4/5
    60000/60000 [==============================] - 180s - loss: 0.0090 - acc: 0.9976 - val_loss: 0.0173 - val_acc: 0.9947
    Epoch 5/5
    60000/60000 [==============================] - 177s - loss: 0.0087 - acc: 0.9972 - val_loss: 0.0188 - val_acc: 0.9943

    <keras.callbacks.History at 0x7fd21ecc1e10>



So with 20 minutes of coding, we were able to reach an accuracy of 0.9955 (epoch 2) which means an error of 0.45%! **This puts the model among the best models in the world!** Don't believe me? Have a look [here](http://rodrigob.github.io/are_we_there_yet/build/classification_datasets_results.html "Best MNIST accuracies").
