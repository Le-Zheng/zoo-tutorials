
##### Copyright 2018 The TensorFlow Authors, 2019 Analytics Zoo Authors.

<img src="https://i.loli.net/2019/09/29/YLn5EVvemRHSaUZ.jpg" width="35" height="20" alt="" align=center>[Run in Google Colab](https://colab.research.google.com/drive/1fcxowPoT25-zP6bD34wVjzBNxm2GIu5G)                  <img src="https://i.loli.net/2019/09/29/bzk5NrSERhpsXQi.png" width="29" height="24" alt="" align=center> [View source on Github](https://github.com/intel-analytics/zoo-tutorials/blob/master/tensorflow/notebooks/basic_classification.ipynb)


```python
%env SPARK_DRIVER_MEMORY=4g
```

    env: SPARK_DRIVER_MEMORY=4g



```python
# !pip install tensorflow==1.10.0
# !pip install matplotlib
```


```python
#@title Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
```


```python
#@title MIT License
#
# Copyright (c) 2017 François Chollet, 2019 Analytics Zoo Authors
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
```

# Train your first neural network: basic classification

This guide trains a neural network model to classify images of clothing, like sneakers and shirts. It's okay if you don't understand all the details, this is a fast-paced overview of a complete TensorFlow program with the details explained as we go.

This guide uses [tf.keras](https://www.tensorflow.org/guide/keras), a high-level API to build and train models in TensorFlow.


```python
from __future__ import absolute_import, division, print_function, unicode_literals

# TensorFlow and tf.keras
import tensorflow as tf
from tensorflow import keras

# Helper libraries
import numpy as np
import matplotlib.pyplot as plt

# You MUST use tensorflow 1.10.0
print(tf.__version__)
```

    1.10.0


## Import the Fashion MNIST dataset

This guide uses the [Fashion MNIST](https://github.com/zalandoresearch/fashion-mnist) dataset which contains 70,000 grayscale images in 10 categories. The images show individual articles of clothing at low resolution (28 by 28 pixels), as seen here:

<table>
  <tr><td>
    <img src="https://tensorflow.org/images/fashion-mnist-sprite.png"
         alt="Fashion MNIST sprite"  width="600">
  </td></tr>
  <tr><td align="center">
    <b>Figure 1.</b> <a href="https://github.com/zalandoresearch/fashion-mnist">Fashion-MNIST samples</a> (by Zalando, MIT License).<br/>&nbsp;
  </td></tr>
</table>

Fashion MNIST is intended as a drop-in replacement for the classic [MNIST](http://yann.lecun.com/exdb/mnist/) dataset—often used as the "Hello, World" of machine learning programs for computer vision. The MNIST dataset contains images of handwritten digits (0, 1, 2, etc) in an identical format to the articles of clothing we'll use here.

This guide uses Fashion MNIST for variety, and because it's a slightly more challenging problem than regular MNIST. Both datasets are relatively small and are used to verify that an algorithm works as expected. They're good starting points to test and debug code.

We will use 60,000 images to train the network and 10,000 images to evaluate how accurately the network learned to classify images. You can access the Fashion MNIST directly from TensorFlow, just import and load the data:


```python
fashion_mnist = keras.datasets.fashion_mnist

(train_images, train_labels), (test_images, test_labels) = fashion_mnist.load_data()
```

Loading the dataset returns four NumPy arrays:

* The `train_images` and `train_labels` arrays are the *training set*—the data the model uses to learn.
* The model is tested against the *test set*, the `test_images`, and `test_labels` arrays.

The images are 28x28 NumPy arrays, with pixel values ranging between 0 and 255. The *labels* are an array of integers, ranging from 0 to 9. These correspond to the *class* of clothing the image represents:

<table>
  <tr>
    <th>Label</th>
    <th>Class</th>
  </tr>
  <tr>
    <td>0</td>
    <td>T-shirt/top</td>
  </tr>
  <tr>
    <td>1</td>
    <td>Trouser</td>
  </tr>
    <tr>
    <td>2</td>
    <td>Pullover</td>
  </tr>
    <tr>
    <td>3</td>
    <td>Dress</td>
  </tr>
    <tr>
    <td>4</td>
    <td>Coat</td>
  </tr>
    <tr>
    <td>5</td>
    <td>Sandal</td>
  </tr>
    <tr>
    <td>6</td>
    <td>Shirt</td>
  </tr>
    <tr>
    <td>7</td>
    <td>Sneaker</td>
  </tr>
    <tr>
    <td>8</td>
    <td>Bag</td>
  </tr>
    <tr>
    <td>9</td>
    <td>Ankle boot</td>
  </tr>
</table>

Each image is mapped to a single label. Since the *class names* are not included with the dataset, store them here to use later when plotting the images:


```python
class_names = ['T-shirt/top', 'Trouser', 'Pullover', 'Dress', 'Coat',
               'Sandal', 'Shirt', 'Sneaker', 'Bag', 'Ankle boot']
```

## Explore the data

Let's explore the format of the dataset before training the model. The following shows there are 60,000 images in the training set, with each image represented as 28 x 28 pixels:


```python
train_images.shape
```




    (60000, 28, 28)



Likewise, there are 60,000 labels in the training set:


```python
len(train_labels)
```




    60000



Each label is an integer between 0 and 9:


```python
train_labels
```




    array([9, 0, 0, ..., 3, 0, 5], dtype=uint8)



There are 10,000 images in the test set. Again, each image is represented as 28 x 28 pixels:


```python
test_images.shape
```




    (10000, 28, 28)



And the test set contains 10,000 images labels:


```python
len(test_labels)
```




    10000



## Preprocess the data

The data must be preprocessed before training the network. If you inspect the first image in the training set, you will see that the pixel values fall in the range of 0 to 255:


```python
plt.figure()
plt.imshow(train_images[0])
plt.colorbar()
plt.grid(False)
plt.show()
```


![png](basic_classification_files/basic_classification_24_0.png)


We scale these values to a range of 0 to 1 before feeding to the neural network model. For this, we divide the values by 255. It's important that the *training set* and the *testing set* are preprocessed in the same way:


```python
train_images = train_images / 255.0

test_images = test_images / 255.0
```

Display the first 25 images from the *training set* and display the class name below each image. Verify that the data is in the correct format and we're ready to build and train the network.


```python
plt.figure(figsize=(10,10))
for i in range(25):
    plt.subplot(5,5,i+1)
    plt.xticks([])
    plt.yticks([])
    plt.grid(False)
    plt.imshow(train_images[i], cmap=plt.cm.binary)
    plt.xlabel(class_names[train_labels[i]])
plt.show()
```


![png](basic_classification_files/basic_classification_28_0.png)


## Build the model

Building the neural network requires configuring the layers of the model, then compiling the model.

### Setup the layers

The basic building block of a neural network is the *layer*. Layers extract representations from the data fed into them. And, hopefully, these representations are more meaningful for the problem at hand.

Most of deep learning consists of chaining together simple layers. Most layers, like `tf.keras.layers.Dense`, have parameters that are learned during training.


```python
model = keras.Sequential([
    keras.layers.Flatten(input_shape=(28, 28)),
    keras.layers.Dense(128, activation=tf.nn.relu),
    keras.layers.Dense(10, activation=tf.nn.softmax)
])
```

The first layer in this network, `tf.keras.layers.Flatten`, transforms the format of the images from a 2d-array (of 28 by 28 pixels), to a 1d-array of 28 * 28 = 784 pixels. Think of this layer as unstacking rows of pixels in the image and lining them up. This layer has no parameters to learn; it only reformats the data.

After the pixels are flattened, the network consists of a sequence of two `tf.keras.layers.Dense` layers. These are densely-connected, or fully-connected, neural layers. The first `Dense` layer has 128 nodes (or neurons). The second (and last) layer is a 10-node *softmax* layer—this returns an array of 10 probability scores that sum to 1. Each node contains a score that indicates the probability that the current image belongs to one of the 10 classes.

### Compile the model

Before the model is ready for training, it needs a few more settings. These are added during the model's *compile* step:

* *Loss function* —This measures how accurate the model is during training. We want to minimize this function to "steer" the model in the right direction.
* *Optimizer* —This is how the model is updated based on the data it sees and its loss function.
* *Metrics* —Used to monitor the training and testing steps. The following example uses *accuracy*, the fraction of the images that are correctly classified.


```python
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])
```

Now to support distributted training, evaluation and prediction, we need to wrap our keras model with Analytics Zoo api


```python
from zoo.tfpark import KerasModel, TFDataset
from zoo import init_nncontext
# set up enviroment
_ = init_nncontext()
# wrap model as tfpark model for distributted training, evaluation and prediction
model = KerasModel(model)
```

    Prepending /home/yang/anaconda3/envs/py36/lib/python3.6/site-packages/bigdl/share/conf/spark-bigdl.conf to sys.path
    Adding /home/yang/anaconda3/envs/py36/lib/python3.6/site-packages/zoo/share/lib/analytics-zoo-bigdl_0.8.0-spark_2.3.2-0.5.0-SNAPSHOT-jar-with-dependencies.jar to BIGDL_JARS
    Prepending /home/yang/anaconda3/envs/py36/lib/python3.6/site-packages/zoo/share/conf/spark-analytics-zoo.conf to sys.path


## Train the model

Training the neural network model requires the following steps:

1. Feed the training data to the model—in this example, the `train_images` and `train_labels` arrays.
2. The model learns to associate images and labels.
3. We ask the model to make predictions about a test set—in this example, the `test_images` array. We verify that the predictions match the labels from the `test_labels` array.

To start training,  call the `model.fit` method—the model is "fit" to the training data:


```python
dataset = TFDataset.from_ndarrays((train_images, train_labels),
                                 batch_size=160,
                                 val_tensors=(test_images, test_labels))
model.fit(dataset, epochs=5)
```

    creating: createAdam
    creating: createZooKerasSparseCategoricalCrossEntropy
    creating: createLoss
    creating: createZooKerasSparseCategoricalAccuracy
    WARNING:tensorflow:From /home/yang/anaconda3/envs/py36/lib/python3.6/site-packages/zoo/util/tf.py:87: convert_variables_to_constants (from zoo.util.tf_graph_util) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.compat.v1.graph_util.convert_variables_to_constants`
    WARNING:tensorflow:From /home/yang/anaconda3/envs/py36/lib/python3.6/site-packages/zoo/util/tf_graph_util.py:283: extract_sub_graph (from zoo.util.tf_graph_util) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.compat.v1.graph_util.extract_sub_graph`
    INFO:tensorflow:Froze 4 variables.
    INFO:tensorflow:Converted 4 variables to const ops.
    creating: createTFTrainingHelper
    creating: createTFValidationMethod
    creating: createTFValidationMethod
    creating: createIdentityCriterion
    creating: createMaxEpoch
    creating: createDistriOptimizer
    creating: createEveryEpoch
    creating: createMaxEpoch


As the model trains, the loss and accuracy metrics are displayed. This model reaches an accuracy of about 0.86 (or 86%) on the training data.

## Evaluate accuracy

Next, compare how the model performs on the test dataset:


```python
test_loss, test_acc = model.evaluate(test_images, test_labels, batch_per_thread=280, distributed=True)

print('Test accuracy:', test_acc)
```

    INFO:tensorflow:Froze 4 variables.
    INFO:tensorflow:Converted 4 variables to const ops.
    creating: createTFNet
    creating: createZooKerasSparseCategoricalCrossEntropy
    creating: createLoss
    creating: createZooKerasSparseCategoricalAccuracy
    Test accuracy: 0.8608999848365784


## Make predictions

With the model trained, we can use it to make predictions about some images.


```python
predictions = model.predict(test_images, batch_per_thread=280, distributed=True)
```

    INFO:tensorflow:Froze 4 variables.
    INFO:tensorflow:Converted 4 variables to const ops.
    creating: createTFNet


Here, the model has predicted the label for each image in the testing set. Let's take a look at the first prediction:


```python
predictions[0]
```




    array([5.0464441e-06, 2.8245449e-08, 3.7241246e-06, 1.3102062e-06,
           3.4668458e-06, 7.1749002e-02, 3.8643364e-05, 1.4251211e-02,
           9.8233577e-05, 9.1384935e-01], dtype=float32)



A prediction is an array of 10 numbers. These describe the "confidence" of the model that the image corresponds to each of the 10 different articles of clothing. We can see which label has the highest confidence value:


```python
np.argmax(predictions[0])
```




    9



So the model is most confident that this image is an ankle boot, or `class_names[9]`. And we can check the test label to see this is correct:


```python
test_labels[0]
```




    9



We can graph this to look at the full set of 10 class predictions


```python
def plot_image(i, predictions_array, true_label, img):
  predictions_array, true_label, img = predictions_array[i], true_label[i], img[i]
  plt.grid(False)
  plt.xticks([])
  plt.yticks([])
  
  plt.imshow(img, cmap=plt.cm.binary)
  
  predicted_label = np.argmax(predictions_array)
  if predicted_label == true_label:
    color = 'blue'
  else:
    color = 'red'
  
  plt.xlabel("{} {:2.0f}% ({})".format(class_names[predicted_label],
                                100*np.max(predictions_array),
                                class_names[true_label]),
                                color=color)

def plot_value_array(i, predictions_array, true_label):
  predictions_array, true_label = predictions_array[i], true_label[i]
  plt.grid(False)
  plt.xticks([])
  plt.yticks([])
  thisplot = plt.bar(range(10), predictions_array, color="#777777")
  plt.ylim([0, 1])
  predicted_label = np.argmax(predictions_array)
  
  thisplot[predicted_label].set_color('red')
  thisplot[true_label].set_color('blue')
```

Let's look at the 0th image, predictions, and prediction array.


```python
i = 0
plt.figure(figsize=(6,3))
plt.subplot(1,2,1)
plot_image(i, predictions, test_labels, test_images)
plt.subplot(1,2,2)
plot_value_array(i, predictions,  test_labels)
plt.show()
```


![png](basic_classification_files/basic_classification_52_0.png)



```python
i = 12
plt.figure(figsize=(6,3))
plt.subplot(1,2,1)
plot_image(i, predictions, test_labels, test_images)
plt.subplot(1,2,2)
plot_value_array(i, predictions,  test_labels)
plt.show()
```


![png](basic_classification_files/basic_classification_53_0.png)


Let's plot several images with their predictions. Correct prediction labels are blue and incorrect prediction labels are red. The number gives the percent (out of 100) for the predicted label. Note that it can be wrong even when very confident.


```python
# Plot the first X test images, their predicted label, and the true label
# Color correct predictions in blue, incorrect predictions in red
num_rows = 5
num_cols = 3
num_images = num_rows*num_cols
plt.figure(figsize=(2*2*num_cols, 2*num_rows))
for i in range(num_images):
  plt.subplot(num_rows, 2*num_cols, 2*i+1)
  plot_image(i, predictions, test_labels, test_images)
  plt.subplot(num_rows, 2*num_cols, 2*i+2)
  plot_value_array(i, predictions, test_labels)
plt.show()
```


![png](basic_classification_files/basic_classification_55_0.png)


Finally, use the trained model to make a prediction about a single image.


```python
# Grab an image from the test dataset
img = test_images[0]

print(img.shape)
```

    (28, 28)


`tf.keras` models are optimized to make predictions on a *batch*, or collection, of examples at once. So even though we're using a single image, we need to add it to a list:


```python
# Add the image to a batch where it's the only member.
img = (np.expand_dims(img,0))

print(img.shape)
```

    (1, 28, 28)


Now predict the image:


```python
predictions_single = model.predict(img)

print(predictions_single)
```

    [[5.0464437e-06 2.8245445e-08 3.7241239e-06 1.3102086e-06 3.4668453e-06
      7.1749061e-02 3.8643360e-05 1.4251209e-02 9.8233468e-05 9.1384923e-01]]



```python
plot_value_array(0, predictions_single, test_labels)
plt.xticks(range(10), class_names, rotation=45)
plt.show()
```


![png](basic_classification_files/basic_classification_62_0.png)


`model.predict` returns a list of lists, one for each image in the batch of data. Grab the predictions for our (only) image in the batch:


```python
prediction_result = np.argmax(predictions_single[0])
print(prediction_result)
```

    9


And, as before, the model predicts a label of 9.


```python

```
