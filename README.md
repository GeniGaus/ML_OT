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

**1) Template Features**

These are the features obtained from the appearance of the image. The vector created from just the raw pixel values is also a template feature. Others include HOGs, CNs, etc

**Histogram of Oriented Gradients or HOGs** :- It is a feature extraction method based on gradient of each pixel.
> 1) The entire sample image for training is divided into $16 \times 16$ blocks with 50% overlap.
> 2) Each block should consist of cells each of 2 \times 2$ cells. Each cell is of size $8 \times 8$ pixels.
> 3) This $16 \times 16$ block forms the sliding window.
> 4) When this window is passed over the image, gradient vector is calculated in terms of magnitude and direction for each pixel. In case of color images, the color channel with highest gradient magnitude is calculated.
> 5) The orientation angle is divided into 9 bins each of 20 degrees
> 6) Now we quantize the gradient orientation into 9 bins as follows:
>> i) the gradient orientation is placed in the corresponding bin with gradient magnitude as its height.
>> ii) if the gradient orientation falls between 2 different bins, then we interpolate it bi-linearly.
> 7) Each of the pixels gradient vector is concatenated to give a global descriptor of the image.

**1) Statistical Features**

These features are obtained by statistical means from the image like color histograms.

**Color Histograms or CHs**:- It is a histogram representing the distribution of colors in an image. It can be visualized as a graph (or plot) that gives a high-level intuition of the intensity (pixel value) distribution. Like for RGB color space, the pixel values will be in the range of 0 to 255. For a different color space, the pixel range may be different.

When plotting the histogram, the X-axis serves as our “bins”. If we construct a histogram with 256 bins, then we are effectively counting the number of times each pixel value occurs. In contrast, if we use only 2 (equally spaced) bins, then we are counting the number of times a pixel is in the range [0, 128) or [128, 255]. The number of pixels binned to the X-axis value is then plotted on the Y-axis.

----------

Based on what kind of feature vectors trackers utilise, they can be:

> #### 1) Template feature-based tracker

> Since these use template features, they are insensitive to illumination changes and blur but sensitive to fast changes in object's appearance like fast motion and fast deformations.

> #### 2) Statistical feature-based tracker

> Since these use statistical features, they are insensitive to fast target state changes but sensitive to illumination changes and blurring.

Thus, both of these complement each other. So, these can be combined to create a more robust and efficient tracker. This leads to the ensemble tracker utilising multiple features. The filter obtained from each is combined to obtain an optimal filter.

---------------

### Multiple features-based tracker

In this, feature vector is extracted from both template and statistical feature extraction technique. The feature vector extracted from each technique undergoes training separately and the filter obtained for each is combined in order to obtain an optimal filter. This filter is then applied on successive frames to track the target. The filter is also continuously updated based on some update strategy.

Following are the steps in detail:
> 1) Consider the image to be of dimension $M \times N \times L$ where M is height, N is width and L is the number of channels in the image.

> 2) Based on different template feature extraction techniques like raw pixel,HOGs,CNs , template feature maps of a particular frame $u$ is formed represented as $x_{t,u} \epsilon R^{M \times N \times L}$ and the desired correlation output representing the target state for each extraction technique and that particular frame $u$ is formed as $y_{t,u} \epsilon R^{M \times N}$. It is a gaussian function with peak value 1 at the center.

> 3) In order to learn a correlation filter ${w_{u}}^{l}$ for each channel 'l' of the feature map, we need to minimize $L^{2}$ error of correlation response compared to the desired correlation output $y_{t,u}$:
>> $$\epsilon = ||{{\sum}_{l=1}}^{L}{{w_{u}}^{l} * {x_{t,u}}^{l}} - y_{t,u}||^{2} + \lambda {{\sum}_{l=1}}^{L}||{w_{u}}^{l}||^{2}$$
> where * denotes correlation and the 2nd term after + is the regularization term with $\lambda$ as the regularization parameter

> 4) The above equation in time domain is solved efficiently by converting it into fourier domain. 
>>>Using Parseval's theorem which states that the sum (or integral) of the square of a function is equal to the sum (or integral) of the square of its transform, taking the fourier transform of the right-hand side of equation will still give the error but in fourier domain.

>>>Using correlation theorem as mentioned above, the correlation term ${{w_{u}}^{l} * {x_{t,u}}^{l}}$ is converted into ${{W_{u}}^{l} . {X_{t,u}}^{l}}$, where "." represents element-wise multipication and letters in capital denote their respective fourier transform.
> So, equation 3 becomes:
>> $$E = ||{{\sum}_{l=1}}^{L}{({W_{u}}^{l})^{*} . {X_{t,u}}^{l}} - Y_{t,u}||^{2} + \lambda {{\sum}_{l=1}}^{L}||{W_{u}}^{l}||^{2}$$
>> Expanding the norm in n-dimension,
>> $$E = {\sum}_{n}(|{{\sum}_{l=1}}^{L}{({W(n)_{u}}^{l})^{*} . {X(n)_{t,u}}^{l}} - Y(n)_{t,u}|^{2} + \lambda {{\sum}_{l=1}}^{L}|{W(n)_{u}}^{l}|^{2})$$
>> $$E = {\sum}_{n}(|{{\sum}_{l=1}}^{L}{{W(n)_{u}}^{l} . ({X(n)_{t,u}}^{l}})^{*} - Y(n)_{t,u}|^{2} + \lambda ||{W(n)_{u}}^{l}||^{2})$$
>> For each n, derive normal equation of above equation and successive application of the formula for inverse of rank-1 adjustment gives:
>>> $${{W}_{l}}^{u} = \frac {{{Y}^{*}}_{t,u} . {X(n)_{t,u}}^{l}} { { {{\sum}_{l=1}}^{L}({X(n)_{t,u}}^{l})^{*} . {X(n)_{t,u}}^{l}} + \lambda } -eq 5$$

> 5) Similarly, for statistical feature extraction techniques like CHs also, corresponding filters are found out using eq 5
