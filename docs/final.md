---
layout: default
title: Final Report
---


## Video 

<iframe width="560" height="315" src="https://www.youtube.com/embed/jyX5lEiyxac" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Base Demo Full Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/1-X6gO7kehQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Good Demo Full Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/8bZH97Knh2o" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Project Summary

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-Intro%20picture.png" width='700' />

In our CS 175 Project, we are interested in the problem of image regression. Given 4 images of size 640x360x3 at one location in a custom-made map of Washington D.C., we predict this location's coordinates in Minecraft. That is, the result is two coordinates x and z. We solve through gathering image data and their corresponding coordinates to train Convolutional Neural Networks (CNNs). 

We would be gathering training data from 4 directions, North, South, East, and West, for a single pair of coordinates. The model would be trained in a 200x200 area: -70 to 130 in the x-direction and 0 to 200 in the z-direction.  Images will be taken every 10 blocks in the x-direction and every 10 blocks in the z-direction. 
Our goal for this project is to train CNNs to perform better than the baseline. We expect a successful model to have less than 2700 MSE. This roughly equates to predicting a coordinate with a 36-block difference from the ground truth in the x-axis and the z-axis. 

We would like to specify some specific language we would be using from now on. We will not be using North, South, East, and West to describe the 4 different views. Instead, we would be using positive x, positive z, negative x, negative z (+x, +z, -x, -z) to describe which way the agent is facing. 


### Why do we need ML/AI for this problem

This problem is non-trivial because we are not sampling every coordinate in the training/testing space. If we were to sample images from every pair of coordinates, we could just use a brute force pixel RGB color matching method. In our case, we are sampling images every 10 blocks in the x-axis and z-axis. We need AI for this problem because it is difficult to identify and extract image features manually, and it is difficult to infer the distance between 2 different images taken. Algorithms that detect image features such as ORB (Oriented FAST and Rotated BRIEF) are good with detecting image features and could identify if 2 images are similar. However, the ORB algorithm lacks the ability to infer how far 2 images are taken. This is critical for our task as we need to be able to not only know that 2 images are similar but also know how far apart the 2 images are. Thus, we need ML algorithms for this task. 

Furthermore, we are also not using a simple map. We are using a complex map with objects such as buildings, roads, cars, trees, etc. Besides, there are also elevation differences between the images taken. This makes the problem difficult as the environment simulates a real environment rather than a simplified testing environment.


## Approach

This section will be divided into 3 subsections: Data Collection, Data Preprocessing, and Models Building and Training.

### Data Collection

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-Data%20gathering.png" />

#### Image Dimension

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-Shape%20explained.png" />

### Data Preprocessing

To reduce computational cost while retaining most image features, we preprocess data by manipulating input images. Specifically, we have implemented different data preprocessing methods for different models except the baseline one. 

For LeNet-5 Individual and LeNet-5 Conv3D models, we convert images to grayscale and normalize image data to the range of [0,1]. 

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-Data%20Preprocessing.png" />

For Four Directions VGG16 and Single VGG16 models, we first normalize image data to [0,1]. As the pre-trained VGG16 model requires a 3-channels input, we keep all RBG channels in our image data but scale down images to half the current size (from 360x640x3 to 180x320x3). 

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/data_preprocessing_vgg.JPG" />


### Models Building and Training

#### Predict Center (Baseline)

This model always predicts the center of the training dataset regardless of the image input. This is a statistical phenomenon known as regression to the mean. When the input features are not correlated with the target values, the optimizer will start to predict towards the center to minimize the loss. The center for our training dataset is [30,100]. The model will always output [30, 100] as its prediction, regardless of what image is feed into it. The advantage of this model is that it does not require machine learning. It needs very minimal time to train because it only needs to average the training coordinates to get the center for the training dataset. The disadvantage of this model is that it is not very accurate. It will always predict the center regardless of how far or how close the image was taken relative to the center. This model is not very practical as one would want to know their current position. Thus, this model is our baseline model.

#### LeNet-5 Individual

This model is based on the LeNet-5, a classical CNN model proposed by Yann LeCun et al. in "Gradient-Based Learning Applied to Document Recognition." It is not the exact LeNet-5 model because we have made modifications to it. This model extracts features from the 4 different images individually. Each direction (+z, -z, +x, -z) has its own convolution layers to extract features, which are then concatenated and feed into a feed-forward multi-layer perceptron. One advantage of this model is that due to its unique architecture of extracting features from the 4 directions individually, the model is able to use Conv2D layers. Recall the shape of our training data is (441, 4, 360, 640, 1). The data has 5 dimensions, and Conv2D layers only accept 4 dimensions, namely (data size, height, width, channels). Using 4 different Conv2D layers to extract image features from the 4 directions allows us to reduce the data dimension to 4 rather than 5. Conv2D has an advantage over Conv3D as it trains faster. This gives us more time to tune the model and adjust it. One disadvantage of this model is that it is not able to recognize what image feature is from what direction. Sometimes images from different coordinates have similar features, but these similar features are captured in a different direction. This will cause the model to think that these 2 images are taken near each other due to their similar image features. The model is unaware of the fact that similar image features are from different directions causing high MSE loss.

We will present 2 pictures below: a high-level picture of the model’s architecture and detailed configurations.

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-lenet%20individual.png" />

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/indivudal%20model.PNG" />

The first convolutional layer has filters = 8, kernel_size = (3, 3), activation = relu, padding = same. The second convolutional layer has filters = 16, kernel_size = (3, 3), activation = relu, padding = same. The first dense layer has units = 512, with linear activation as default and the output layer has an output size of 2 with linear activation function. We fit the training data with batch_size = 10 and epochs = 20. To avoid overfitting the data, we decide to add dropout layers and stop early. We programed a callback function that will stop training once the loss fell below 700. We got this number after trial and error. It was determined that the optimal test and validation values occurs when the loss is in the range of 500 to 700. 

#### LeNet-5 Conv3D

This model is based on the LeNet-5 model. It is not the exact LeNet-5 model because we have made modifications to it. We replaced the Conv2D layers in the LeNet-5 model with Conv3D layers in order to fit the data. Recall the shape of our training data is (441, 4, 360, 640, 1). The data has 5 dimensions, and Conv3D can accept data up to 5 dimensions (data size, frames, height, width, RGB channels). Having Conv3D layers instead of Conv2D layers allows us to avoid complex data preprocessing steps that separate pictures based on directions. This saves storage space and is one of the advantages of this model. In addition, the model is relatively simple, making it easy to adjust and tune due to its simple architecture. One disadvantage of this model is that it takes longer to train compared to similar models with Conv2D layers. 

We will present 2 pictures below: a high-level picture of the model’s architecture and detailed configurations.

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-Conv3D.png" />

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/conv3d%20description.PNG" />

The first convolutional layer has filters = 16, kernel_size = 5, activation = relu, padding = same. The second convolutional layer has filters = 32, kernel_size = 5, activation = relu, padding = same. All dense layers have linear activation function and the output layer has an output size of 2 with linear activation function. We fit the training data with batch_size = 3 and epochs = 5. To avoid overfitting the data, we decide to stop early (Around 1000 MSE). 

#### Four Directions VGG 16

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/vgg16_architecture.JPG" width='900'/>
Image Source: [VGG16](https://neurohive.io/en/popular-networks/vgg16/)

After we created two models based on the LeNet-5 architecture and trained them from scratch, we would like to use a pre-trained VGG model for our problem by transfer learning. VGG is a CNN model proposed by K. Simonyan and A. Zisserman in "Very Deep Convolutional Networks for Large Scale Image Recognition." This model has excellent performances on object classification and localization tasks, which could be modified to solve an image regression problem. Our model is a direct implementation of pre-trained VGG16 with some tuning and adjusting to make the model a regression model rather than a classification model. First, we removed the last fully-connected classifier in the original VGG16 shown in the above figure. Similar to LeNet-5 Individual, we then applied this modified VGG16 on 4 directions and merged them into one model with the final customized dense layers to output coordinates in x-axis and z-axis.

One crucial advantage of this model is the usage of the pre-trained model with transfer learning, which VGG16 has been trained on a large dataset, ImageNet. Though we have training image data for 441 coordinates, it is still not a big enough dataset to train complex models from scratch. Therefore, we could apply this pre-trained VGG16 as the general feature extractor to improve our model's performance with the current training dataset. Having lots of convolutional layers is another major advantage of this model. Highly complex environments such as ours may need multiple convolutional layers to distinguish subtle differences. Due to the model's high complexity, Four Directions VGG16 takes a long time to train. This could be seen as a disadvantage as the time constraint could reduce our opportunities to tune the model.

We will present 2 pictures below: a high-level picture of the model's architecture and detailed configurations.


<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-VGG%2016.png" />

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/vgg1.PNG" width='500'/>

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/vgg2.PNG" width='500'/>

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/vgg3.PNG" width='500'/>

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/vgg4.PNG" width='500'/>

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/vgg5.PNG" width='500'/>

The pre-trained VGG16 model was imported from TensorFlow Keras. For specific configuration, please refer [here]( https://github.com/tensorflow/tensorflow/blob/v2.3.1/tensorflow/python/keras/applications/vgg16.py#L45-L225). To remove the fully connected layers in the pre-trained VGG16, we set include_top = False. After concatenation of 4 pre-trained VGG16, we create our dense layers. The first dense layer has units = 128, with relu activation. To avoid overfitting the data, we decide to add dropout layers with a dropout rate of 0.2. The final output layer has an output size of 2 with a linear activation function. We fit the training data with batch_size = 10 and epochs = 10. The optimizer was "adam," and the loss function is MSE. In the model training process, we freeze all Conv Blocks and keep all initial weights in pre-trained VGG16: set all layers.trainable = False. Then, we only train the dense layers to output coordinates with less training time. With the callbacks of ModelCheckpoint and EarlyStopping, the model will save every improved model and stop training when there are no improvements on MSE after 3 epochs. 

#### Single VGG 16

Unlike using 4 pre-trained VGG16 in the previous model, this time we used a single VGG16 for images from all 4 directions. We combined images from all 4 directions to a single image and fed that into our Single VGG16 model. As training images have been downscaled in the data preprocessing steps, we constructed the input images with the structure shown below in the picture.  

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/combined_images.JPG" />

Single VGG16 has a speed and space advantage as well as a good performance compared to the Four Directions VGG16 because Single VGG16 is a much smaller and simpler model. It is crucial for us to have such an advantage to develop a good real-time coordinates prediction in Minecraft with a balance of efficiency and performance. One disadvantage of this model is that the convolution layers apply one filter to extract features from all 4-direction images, which may impact our model's ability to generalize to other images from new coordinates. In our later evaluation section, we can see that, though Single VGG16 has the smallest validation MSE, its prediction on the new coordinate is not as good as the one from Four Directions VGG16 in practical use. 

We will present 2 pictures below: a high-level picture of the model’s architecture and detailed configurations. 

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-Single%20VGG%2016.png" />

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/single%20vgg16.PNG" width='500'/>

The pre-trained VGG16 model was imported from TensorFlow Keras. For specific configuration, please refer [here]( https://github.com/tensorflow/tensorflow/blob/v2.3.1/tensorflow/python/keras/applications/vgg16.py#L45-L225). To remove the fully connected layers in the pre-trained VGG16, we set include_top = False. Then we connect the modified VGG16 with our dense layers. The first dense layer has units = 128, with relu activation. To avoid overfitting the data, we decide to add dropout layers with a dropout rate of 0.2. The final output layer has an output size of 2 with a linear activation function. With the callbacks of ModelCheckpoint and EarlyStopping, the model will save every improved model and stop training when there are no improvements on MSE after 3 epochs. 

For this model, we have two training steps. In the first step, we freeze all Conv Blocks and keep all initial weights in pre-trained VGG16: set all layers.trainable = False. Then, we only train the dense layers. We fit the training data with batch_size = 10 and epochs = 7. The optimizer was "adam," and the loss function is MSE. After this training step, we obtain the trained top dense layers for our model. In the second step, we fine-tune our entire model: unfreeze the Conv Block 5 and train the last several convolutional layers with dense layers to extract features specific to our model. We again fit the training data with epochs = 3 and the optimizer "adam" with a small learning rate of 0.0001 to avoid overfitting. 

## Evaluation

This section will be divided into 2 subsections: Quantitative Evaluation and Qualitative evaluation. We will discuss our metrics in each of the evaluations, provide examples and data, and explain how we have solved the problem and to what extent have we solved the problem.

### Quantitative Evaluation

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/mse.PNG" width='500' />

We will be evaluating our models quantitatively using mean squared error. Mean square error measures the average of the squares of the difference between the estimated values and the actual value. In this project's context, the mean squared error is applied to the estimated coordinates (x and z) of where an image was taken and the true coordinates of where that image was taken. Note that the MSE formula above is not the average Euclidean distance error. However, the formula is proportional to the average Euclidean distance error. In general, the lower the MSE is, the closer our estimated coordinates are to the true coordinates. We ran our models on the training dataset and tuned it according to the test dataset's feedback. After we believe that we have achieved the optimal configuration, we ran our model on the validation dataset. The results are presented in the table below.

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/Table.PNG" width='500' />

From the MSE table above, we could see that our Single VGG 16 model performed the best followed by Four Direction VGG 16 model, LeNet-5 Individual and LeNet-5 Conv3D. Specifically, Single Direction VGG model has a test MSE of 323 and a validation MSE of 163. For a more concrete idea on these numbers, given 4 validation images of the same coordinate, LeNet-5 Individual model would predict a coordinate that is on average 33 blocks away from the true coordinates in both x and z direction. LeNet-5 Conv3D model would predict a coordinate that is on average 35 blocks away from the true coordinates in both x and z directions. Four Direction VGG model would predict a coordinate that is on average 16 blocks away from the true coordinate in both x and z directions. Last but not the least, Single VGG 16 model would predict a coordinate a coordinate that is on average 12 blocks away from the true coordinate in both the x and z direction. All of these models have a lower MSE compared to the Simple CNN model we did in our status report. The Simple CNN model has a MSE of 2264 which means on average 47 blocks away in both x and z directions. 

We believe that we have achieved our goal indicated in the project summary section. All of our CNN models have less than 2700 MSE. And all the CNN models also have significantly lower MSE than our baseline model. We have solved this regression problem to the extent that users could get a rather accurate idea of their position in the Cartesian coordinate system: on average 12 blocks away from the true coordinate. To further proof that we have solve the problem, we have provided histograms below to show the distribution of Validation MSEs for the VGG 16 models. 

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/four_directions_vgg_training_loss.JPG" width='400' /><img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/mse%20distribution.PNG" width='400' />

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/single_vgg16_training_loss.JPG" width='400' /><img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/singlevgg16msetable.PNG" width='400' />

From the plots of training loss above, we can see that both VGG16 models quickly converges after the third epochs and reaches very low MSEs in the end of training. From the histograms above, we could observe that most of the data fall below 200 MSE. Specifically, around half of the validation data have an MSE in the range 0-150. This means that around half of the predictions are less than 12 blocks in both directions to the true coordinates. There are also a few data points that go above 200. This is likely due to the map resembling a real-life scenario. Real-life environments are complexed and highly subjected to change. Even a 1-2 blocks away could have drastically different image features. The reason for showing the histograms above is to show that the practical MSE is lower than the MSE presented in the table above. There are bad sampling points, such as the one around 1300 MSE that increases the average MSE. Overall, we believe that our model has accomplished the goal of predicting coordinates within a reasonable distance. We were also very close to the moonshot case of having 100 MSE. Thus we believe that we have solved the problem sufficiently enough for users to take 4 images in Minecraft and have a general idea of where they are.

### Qualitative Evaluation

One of the most important qualitative factors of our model is the ability to predict a coordinate that yields images similar to that of the true coordinates. This is important because similar images indicate that the predicted coordinates and the true coordinates are within a reasonable distance. We analyzed our models qualitatively by manually identifying key features in the test images and determine if those features are presented in the coordinates predicted by each model. We loaded the models onto our result checker. We gave each model the same test picture, and the results are shown above. The test image is taken at [99, 173]. It consists of some key features listed below.

1.	Light & Dark green tiles in all the images
2.	Whitehouse towards the left side in the –z image
3.	The Washington Monument is in the far back in the –x image
4.	Large building structure in –x, +z, and +x

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/CS175%20Final%20Diagrams-Image%20Features.png" />

<img src="https://raw.githubusercontent.com/alaister123/Geolocator/main/docs/img_final/qualeval.png" />

We could observe that the baseline looks very different from the test image. There isn’t the presence of light & dark green tiles. The Whitehouse, Washington Monument, and the large building structure are either zoomed in or too far back. LeNet Individual model looks very similar to the test image. The 4 features listed above are presented, and they look very similar to the test image. LeNet Conv3D is slightly worse than LeNet Individual. LeNet Conv3D model predicts [63, 157]. One key feature missing is the altering of light & dark green tiles. The 3 other key features look somewhat similar to the test image. Image drawn from Four Direction VGG16’s predicted coordinates look almost identical to the true images. All four features listed above are presented, and they are scaled similarly to that of the true images. Singe VGG16 model have all 4 features shown in the images, but the point of view is far forward from the true location. 

Based on our qualitative assessment, Four Directions VGG16 solves the problem qualitatively to the extent that only small detailed differences between the test images and the images from Four Direction VGG16 could be seen. Large image features are identical to the test images.  

## References

[Project Malmo](https://www.microsoft.com/en-us/research/project/project-malmo/)

[TensorFlow](https://www.tensorflow.org/)

[Keras](https://keras.io/)

[OpenCV](https://opencv.org/)

[Feature detection and matching with OpenCV](https://blog.francium.tech/feature-detection-and-matching-with-opencv-5fd2394a590)

[Regression to the mean and its implications](https://towardsdatascience.com/regression-to-the-mean-and-its-implications-648660c9bf76)

[Feature extraction from images](https://www.kaggle.com/lorinc/feature-extraction-from-images)

[Deep Learning Models for Multi-Output Regression](https://machinelearningmastery.com/deep-learning-models-for-multi-output-regression/)

[XML Schema Documentation](https://microsoft.github.io/malmo/0.14.0/Schemas/Mission.html)

[scikit-learn](https://scikit-learn.org/stable/)

[Basic regression: Predict fuel efficiency](https://www.tensorflow.org/tutorials/keras/regression)

[Dual-input CNN with Keras](https://medium.com/datadriveninvestor/dual-input-cnn-with-keras-1e6d458cd979)

[Keras, Regression, and CNNs](https://www.pyimagesearch.com/2019/01/28/keras-regression-and-cnns/)

[Understanding 1D and 3D Convolution Neural Network Keras](https://towardsdatascience.com/understanding-1d-and-3d-convolution-neural-network-keras-9d8f76e29610)

[Draw.io](draw.io)

[Gradient-Based Learning Applied to Document Recognition](https://ieeexplore.ieee.org/document/726791)

[Very Deep Convolutional Networks for Large Scale Image Recognition](https://arxiv.org/abs/1409.1556)

[VGG16 – Convolutional Network for Classification and Detection](https://neurohive.io/en/popular-networks/vgg16/)






