---
title: Implementation Waste Recognition
layout: default
parent: wastebin
nav_order: 3
---


# Implementation of the waste recognition

One of the core mechanics of this prototype is the recognition of different kinds of waste, which triggers the catapult mechanism in case of the wrong kind of trash is thrown into the bin.
In the following, the implementation of the waste recognition shall be described in detail, as well as the different attempts at implementing it. 

## Setup
For the recognizing different kinds of trash, a Raspberry Pi in combination with a camera module was used. This camera was then directed at the area of the bin where the waste gets thrown in by people passing by. For the implementation of the recognition system, python was chosen as language. 

## Iteration A

The first approach to solve the waste recognition task was to find pre-trained models which could be easily used to recognize different kinds of waste. Beside being able to classify different kinds of waste, the model also needed to be usable on a Raspberry Pi. After some research, a firs pre-trained model was found. It was a TensorFlow-lite model pre-trained on the COCO-dataset, which includes 80 different classes of objects. 

Since there was no suitable waste dataset within reach at this point, the idea was formed to test this model as a first try and only use the class "bottle" for recognition.
Meaning that the only distinction would have been between a bottle and every other kind of waste. But experimenting with the camera module, the realization came fast that the model was not working well enough for the purpose of this prototype. While the model was not a suitable solution, a first code snippet for loading and applying models was found.

## Iteration B

Since the first model was not working well enough, the decision was made to look for pre-trained models which were trained on more specific datasets, meaning models that were actually able to distinguish mainly between waste and did not include other objects. After some research, a dataset called TACO ("trash annotated in context") on Kaggle. This dataset included only images of different kinds of waste and offered detection as well as classification. Another issue arose with the fact that the only on the Kaggle website available pre-trained model weights for this dataset was in the format of a TensorFlow frozen-graph (.pb) file. 

After some research, attempts at converting this file to a format which could be loaded in a TensorFlow-lite application were made. As it seems, the method for this conversion process was no longer functional in the tensorflow2 and different attempts at using another version of TensorFlow resulted in the same error. This option was then abandoned. Another possible solution was to use another a different PyTorch model and use this for inference during the detection process. 

## Iteration C

In this step, a during research discovered PyTorch model was used as well as a different tutorial for which it was necessary to install a new operating system on the Raspberry Pi. On the basis of the tutorial code, the found pre-trained ResNet50-model was integrated and then tested with the help of the camera module. This model was trained on a different dataset, which was also found on the Kaggle website.
After a short test evaluation with the camera module, the results were not as good as expected, since the model was not very confident in its predictions as well as not very accurate. One of the reasons for this might be performance issue on the Raspberry Pi, and another reason which might have played a role is that the model might have learned the background in the dataset images on accident as well. Since all the images had the same grayish background. These circumstances might have given the model more problems with the recognition in real time than normally.

Since there was Jupyter-notebook attached, another try to train a model ourselves which was more up to the task was attempted. For this, different variations of some of the training parameters were tried (number of epochs, learn rate, optimizer, etc.).
Sadly the results did not much differ and after three more attempts with similar results a different solution for the problem was sought. 

## Iteration D

Since all previous attempts at using machine learning techniques were not especially successful, the decision to use a computer vision-based approach was made. After some research, a fitting tutorial for color recognition was found. Based on this, the final implementation of the color recognition script was created. The idea behind it is quite simple. A lower and an upper boundary for the color values are defined, then a mask is created. On this basis, the size of the color area is determined and the decision if a certain color is within the frame is made. This approach worked quite well from the start, and it was also working well in for the live camera feed.

Apart from the recognition, it was also necessary to add the functionality for the audio files with the insults as well as send signals to the different Arduino pins. The audio files were simply loaded to the Raspberry Pi and then within an array containing the file names chosen randomly to be played. As for the communication with the Arduino, there exist two methods within the script. One for the right kind of trash, which lets down the 3D-printed board in the bin. And one method for what happens when the wrong kind of trash is recognized in the bin.

Since it was not possibly to differentiate between different kinds of waste in reality, the decision was made to use colors instead. More specifically, red for the wrong kind of trash and green for the right kind of trash.

After some tweaking, the code worked quite well. The main problem which remained was a nearly always full video buffer which still had frames inside from the last recognition and thus caused quite some problems while debugging. But the simple solution for this was to just cal the read()-method of the OpenCV videocaputure object a few times when a color was recognized.

As the final result, the application is able to tell if a green or red object is within the bin and trigger the according action. For a future iteration, the integration of a "real" waste classification model that is able to tell different kinds of trash apart would be the goal. 
