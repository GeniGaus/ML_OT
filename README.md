# Object Tracking
-------

### What is object tracking?

Object tracking is used to estimate the position of a target in every frame
of each video sequence. Object tracker is model-free.
Object trackers are of two types:
> 1) **Generative Trackers** : In this, an effective appearance of the model is created and tracking is performed by searching for patches in the frame most similar to the target.

> 2) **Discriminative Trackers** : These perform tracking by separatinf the target from the background.
--------

### Discriminative Trackers

**Discriminative trackers or discriminative correlation filters(DCF)-based trackers** is being preferred over generative trackers. The reason will become clear shortly.
They are so named because the filters used in these undergo correlation operation with the feature vector or input and they posses discriminative properties in the sense that they are able to distinguish between the object and the background. Before we delve deeper, lets first look into correlation.

**Correlation**

It is the integral over all space of one function at x times another function at x-u.

![correlation](https://raw.githubusercontent.com/GeniGaus/ML_ObjectTracking/master/assets/correlation.gif)

Correlation is so named because if two functions f and g contain similar features but at a different position, then the correlation function will have large value at a vector corresponding to the shift in the position of the feature, i. e. when u = shift in position. The figure below shows this.

![](https://raw.githubusercontent.com/GeniGaus/ML_ObjectTracking/master/assets/correlation_graph.gif)

The paper [Exploiting circulant structure of tracking by detection with kernels](http://www.robots.ox.ac.uk/~joao/publications/henriques_eccv2012.pdf) prove that as we add more samples from a frame in the training data, the problem of tracking acquires a circulant structure. So, if the filter also posses cirulant structure, then there will be efficient learning. This observation leads to the use of correlation operation between feature vector(input) and filter, where filter can be thought of as f(x) which is shifted by u as in the above formula.

Now, calculating this correlation integral can be time-consuming and inefficient for real-time scenario. This problem can be overcome by **Correlation Theorem**. It states that fourier transform of correlation of two functions is equal to the product of complex conjugate of fourier transform of one function with the fourier transform of the other.

![](https://raw.githubusercontent.com/GeniGaus/ML_ObjectTracking/master/assets/corrtheorem.gif)

Thus, if we transform both filter and feature vector from time domain to fourier domain, then the response map can be calculated simply by multiplying both the transforms and then by taking its inverse transform. This operation of multiplying two transforms can be done efficiently by means of Fast Fourier Transform(FFT) and takes $O(NlogN)$ time. Whereas, correlation in time domain takes $O(N^2)$ time.

So, from above the following points evolve:
> 1) Correlation filters take advantage of the cyclic shift observed in the samples during training.
> 2) Correlation filters will also consider the target's context to have more discriminative information than just its appearance model because of its cyclic shift, which is generally true for most cases.
> 3) Its more time efficient.

Thus, the object tracking problem solving saw a shift from generative trackers to discriminative trackers.

-----------

### Feature Vector

It is a vector or matrix which describes the image and is given as input in the tracker. It can be formed from two aspects:
> 1) Deriving the vector from the image's appearance or template known as **template features**
> 2) Deriving the vector from the statistical features of the image known as **statistical features**
