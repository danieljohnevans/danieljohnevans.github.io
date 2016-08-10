---
layout: post
title:  "Tensorflow Notes"
date:   2016-08-05 17:09:50
categories: ml notes
published: false
---

This is my lab notebook as I work through Google's Tensorflow tutorial. [Scott Weingart](https://twitter.com/scott_bot) turned me onto the idea of maintaining notebooks so that I will be able to quickly reference and explain concepts later. Thanks Scott.


Deep Learning and Google's Tensorflow are two aspects of machine learning I've wanted to play around with for awhile now. I've understood neural networks from a conceptual level but never closely examined how they worked. I work through the [tutorial hosted on Google's site](https://www.tensorflow.org/versions/r0.9/tutorials/mnist/beginners/index.html#mnist-for-ml-beginners) below :

### The data

The Tensorflow tutorial draws from the [MNIST dataset](http://yann.lecun.com/exdb/mnist/). This database consists of 60,000 examples (and a training set of 10,000 examples) of handwritten numbers. These digits standardized as 28x28 pixel images. This data is popular because it requires minimal preprocessing and allows researchers a fairly standard dataset in which to tell their hypotheses. It was created by Yann LeCun, Corinna Cortes, and Christopher J.C. Burges.

The Tensorflow tutorial further divides this dataset into 55,000 for examples, 10,000 for a training set, and 5,000 for a validation set.

### Learning

The Tensorflow tutorial employs softmax regression system. I understand a softmax regression as a simple model that classifies available evidence and then teaches based on those classes. In this sense, we create a model that recognizes the numbers 0 through 9 as various classes as a first step in our softmax regression and then as a second step construct a model that teaches based on the available data.

Our model is taught based on pixel intensities. See below :

In essence, certain numbers are more likely to darken certain pixels than others. Simply put, once we determine a pattern, we can train this pattern against new data.

Onto the code:

First import the dataset. To note: in this tutorial, we are using "one-hot vectors." One-hot vectors are single vector dimensions (each number is represented as 0 or 1). As such, the mnist.train.labels is an array of [55000, 10] floats.

```python

from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)

import tensorflow as tf

```
Let's walk through this diagram (NOTE : insert expression below)

- x = input
  - x is flattened into a 784-dimensional vector. Specifically, it's a 2D tensor of floating-point numbers with a shape of [None, 784]
- w = weights
  - W follows a similar dimensional pattern but the variable is a modifiable variable based on the training model.
- b = bias
- Here is the equation written out : y = softmax(Wx + b)

```python

x = tf.placeholder(tf.float32, [None, 784])

W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))

y = tf.nn.softmax(tf.matmul(x, W) + b)

```



```python

y_ = tf.placeholder(tf.float32, [None, 10])

cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))

```

```python

train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)

init = tf.initialize_all_variables()

sess = tf.Session()
sess.run(init)

```

```python

for i in range(1000):
  batch_xs, batch_ys = mnist.train.next_batch(100)
  sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})

  correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))

accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

print(sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels}))

```



### Conclusions
