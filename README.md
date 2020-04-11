# Multi-Label Image Classifier for Netflix Movies & Korean Dramas 

The repository provides all the files used to build a multi-label image classification model using movie posters with keras in jupyter notebook. The idea came about from looking at other projects such as the one listed [here](https://www.analyticsvidhya.com/blog/2019/04/build-first-multi-label-image-classification-model-python/). There are a couple of things we can add on to it.

It uses a relatively simple model with about 0.24 validation loss. Pre-trained models can be used instead to tey to better that and improve the classfication accuracy which we infer tobe about 90%. 

Secondly on a lighter note, after doing this particular project,  I can finally take some time to catch up with the latest movies or even the latest Korean dramas, something I haven't been doing for a while. 

We can also see if the model can be used to predict the genres of the latest popular dramas at this time of wrting such as Itaewon Class and Crash Landing on You.

<p align="center">
  <img src="testing/kdrama.jpg">
</p>

## The Difference Between Multi-Labels & Multi-Class
Suppose we are given images of shapes to be classified into their corresponding categories. For ease of understanding, let’s assume there are a total of 4 shapes (circle, rectangle, square and triangle) in which a given image can be classified. Now, there can be two scenarios:

  1. Multi-class
  Each image contains only a single shape (either of the above 4 shapes) and hence, it can only be classified in one of the 4 
  classes.
  
  2. Multi-label
  The image might contain more than one shape (from the above 4 categories) and hence the image will belong to more than 
  one shape.


## How to Get Started

With this understandning we will now begin to work on our multi-label image classification.

### Gather Images

We need images of movie and drama posters and have a folder containing all the images for training the model. Along with images, we will require the true labels of images in the form of a .csv file that contains the names of all the training images and their corresponding true labels.


### Load and pre-process the data

The images will be loaded and pre-processed and split for training and validation.
 
### Determine the model’s architecture

The next step is to define the architecture of the model. This includes deciding the number of hidden layers, number of neurons in each layer, activation function, and so on. We will use a pre-trained model and weights.

 
### Train and validate the model

The training images and their corresponding true labels will be used to train the model. We also pass the validation images to help us validate how well the model will perform on unseen data.

 
### Make predictions on new images

Finally, we will use the trained model to get predictions on new images.


## Buidling our Image Classfier

Our aim is to predict the genre of a movie using just its poster image. Can you guess why it is a multi-label image classification problem? Think about it for a moment before you look below.

A movie can belong to more than one genre, right? It doesn’t just have to belong to one category, like action or comedy. The movie can be a combination of two or more genres. Hence, multi-label image classification.

The dataset we’ll be using contains the poster images of several multi-genre movies. I have made some changes in the dataset and converted it into a structured format, i.e. a folder containing the images and a .csv file for true labels. You can download the structured dataset from [here](https://drive.google.com/file/d/1iQV5kKF_KGZL9ALx9MMXk_Lg7PklBLCE/view). Below are a few posters from our dataset:


<p align="center">
  <img src="testing/postersample.jpg">
</p>

## The Codes
First, import all the required Python libraries:

```
import tensorflow.keras
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Dropout, Flatten
from tensorflow.keras.layers import Conv2D, MaxPooling2D
from tensorflow.keras.utils import to_categorical
from tensorflow.keras.preprocessing import image
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from tqdm import tqdm
from tensorflow.keras.applications.inception_v3 import InceptionV3
from tensorflow.keras.applications.vgg19 import VGG19
from tensorflow.keras.regularizers import l2
from tensorflow.keras.models import Sequential, Model
from tensorflow.keras.layers import Dense, Dropout, Activation, Flatten
from tensorflow.keras.layers import Convolution2D, MaxPooling2D, ZeroPadding2D, GlobalAveragePooling2D, AveragePooling2D
%matplotlib inline
```
```
train = pd.read_csv('data/train.csv')    # reading the csv file
train.head()      # printing first five rows of the file
```
<p align="center">
  <img src="testing/headcsv.JPG">
</p>

```
train.columns
```
<p align="center">
  <img src="testing/columnlabels.JPG">
</p>

The genre column contains the list for each image which specifies the genre of that movie. So, from the head of the .csv file, the genre of the first image is Comedy and Drama. The remaining 25 columns are the one-hot encoded columns. So, if a movie belongs any of the 25 genrew, its value will be 1, otherwise 0. The image can belong up to 25 different genres.

We will now load and preprocessing the data and read in all the training images:

```
train_image = []
for i in tqdm(range(train.shape[0])):
    img = image.load_img('data/Images/'+train['Id'][i]+'.jpg',target_size=(128,128,3))
    img = image.img_to_array(img)
    img = img/255
    train_image.append(img)
X = np.array(train_image)
```
The images have been resized to 128 x 128 x 3. This is mainly to allow processing to be done faster since running on a local machine GPU, my jupyter notebook also kept giving a dead kernel which suggests running out of memory space.

Let's look at the shape of the data we are dealing with:
```
X.shape
```
(7254, 128, 128, 3)

```
plt.imshow(X[1001])
train['Genre'][1001]
```

<p align="center">
  <img src="testing/Dadposter.JPG">
</p>

This movie has 2 genres – Comedy and Drama. The next thing our model would require is the true label(s) for all these images. 
For each image, we will have 25 targets, i.e. All these 25 targets will have a value of either 0 or 1.
The Id and genre columns from the train file and convert the remaining columns to an array which will be the target for our images:
```
y = np.array(train.drop(['Id', 'Genre'],axis=1))
y.shape
```
(7254, 25)

The shape of the output array is (7254, 25) as we expected. Now, let’s create a validation set which will help us check the performance of our model on unseen data. We will randomly separate 10% of the images as our validation set:
```
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=100, test_size=0.2)
```
## Model Architecture
The next step is to define the architecture of our model. The output layer will have 25 neurons (equal to the number of genres) and we’ll use sigmoid as the activation function.

I will be using a certain architecture (given below) to solve this problem. You can modify this architecture as well by changing the number of hidden layers, activation functions and other hyperparameters.
```
vgg19 = VGG19(weights='imagenet', include_top=False)
x = vgg19.output
x = GlobalAveragePooling2D()(x)
x = Dense(128,activation='relu')(x)
x = Dropout(0.2)(x)
predictions = Dense(25,kernel_regularizer=l2(0.005), activation='sigmoid')(x)
model = Model(inputs=vgg19.input, outputs=predictions)
```
Let’s print our model summary:
```
model.summary()
```
<p align="center">
  <img src="testing/vgg19model.JPG">
</p>


We will use binary_crossentropy as the loss function and ADAM as the optimizer (again, you can use other optimizers as well):
```
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
model.fit(X_train, y_train, epochs=10, validation_data=(X_test, y_test), batch_size=32)
```
We will train the model for 20 epochs and also pass the validation data which we created earlier in order to validate the model’s performance:

<p align="center">
  <img src="testing/trainmodel.JPG">
</p>

We can see that the training loss went below to 0.23 and the validation loss is also in sync. Validation accuracy was up to 91%.

```
val_eval = model.evaluate(X_test, y_test, verbose = 1)
print('Validation loss:', val_eval[0])
print('Validation accuracy:', val_eval[1])
```
Validation loss: 0.25739657137412025
Validation accuracy: 0.91283244

## Making Predictions

Now, we will pre-process and predict the genre for these posters using our trained model. The model will tell us the probability for each genre and we will take the top 3 predictions from that.

```
img = image.load_img('data/testing/roskywalker.jpg',target_size=(128,128,3))
img = image.img_to_array(img)
img = img/255
classes = np.array(train.columns[2:])
proba = model.predict(img.reshape(1,128,128,3))
top_3 = np.argsort(proba[0])[:-4:-1]
for i in range(3):
    print("{}".format(classes[top_3[i]])+" ({:.3})".format(proba[0][top_3[i]]))
plt.imshow(img)
```
```
<p align="center">
  <img src="testing/.JPG">
</p>

