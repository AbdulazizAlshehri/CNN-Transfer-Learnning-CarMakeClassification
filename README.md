
<!-- ![banner Image](Graphs/class_montage.jpg?raw=true "banner") -->

# Deep Learning Car Classification
credit: [Abdulaziz Alshehri](https://github.com/AbdulazizAlshehri) & [Ibrahin Alzuhairi](https://github.com/ibalzuhairi)

## About
Computer Vision project to classify cars picture by make and model. 
</br>
![Car Classification Cam Image](Graphs/car_classification_security_cam.png?raw=true "Car Classification Cam")


## 1. Dataset
Dataset used in this experiment is Stanford Car dataset [source](http://ai.stanford.edu/~jkrause/cars/car_dataset.html), which contains ~16k total images. We splited them as follows: ~9k for training, ~4k images for validation, ~3k for testing. 
</br> 
![Dataset Example Image](Graphs/dataset.jpg?raw=true "Dataset Example")

### Preparing Dataset
The original dataset had 16k images stored in one folder, where all labels provided in .mat file.
1) restructuring image files to the following LabelID/Imagefile.jpg

2) custom train_test split per class (Note: this will create a balanced split across all classes)

![Dataset_Preparation](Graphs/PreparingDataset.JPG?raw=true "Dataset Example")

### Preprocessing Steps

a CVPre class was created to contain all preprocessing functions. this was done to make it easy to update preprocessing steps in one place across all machines

![Dataset_PreProcessing](Graphs/Preprocessing.JPG?raw=true "PreProcessing")

### Augmentation
Due to the small number of training data image-augmentation applied to the training data (i.e. scaling, rotation, brightness) to avoide overfitting, for example by applying augmentation on the following image:
</br>
<img src="Graphs/Augmentation_original.jpg" height="140">

the folllowwing 5 new images created:
![Augmentation Result Images](Graphs/Augmentation2.png?raw=true "Augmentation Result")

## 2. Experiment
### Workflow
Since we intended to perform many experiments on more than remote machine. MLFLow was a great tool to track experiments in one place, and streamline artificat file storage.


![Experiment_Workflow](Graphs/workflow.JPG?raw=true "workflow")

Note: while MLFLow did work on local network. Network related problems prevented access from remote machines. Hence, it was leftout due to time constraints

## Earlier experiments:
On initial experiments troubleshooting, we realize what caused stagnation on our loss curve for transfer learning of VGG16, and MobileNetV2.
1- rescaling training dataset between -1 and 1 was miscounted step. Noticable performance increase after rescaling.
2- pre-trained VGG16 on ImageNet,  training set were rescaled to the mean and std of ImageNet colors. Another performance increase was noticed after rescaling our training set to the same mean and std of ImageNet.
3- Learning rate choice was either too small or too big. ReduceOnPlateau was used to adjust LR on stagnation. 


### Learning Rates Experiment:
to explore the impact of choosing a good learning rate. we've ran multiple experiments with different learnning rates on pre-trained MobileNetV2.
![LR_Exp](Graphs/LRexp.JPG?raw=true "LRExp Example")
Above chart shows that a large learning rate will speedup initial training. However, if left as is it will cause the model to overfit.
This highlight the benefit of using LR scheduler, or reduce on plateau.

### Rescaling & Normalization:
One of the miscounted step, is to rescale and normalize training set in the same way the ImageNet pre-trained model was processed.
<br> Below charts show the difference in performance on transfered VGG16  after rescaling between -1 and 1,  and normalizing training set with ImageNet mean and STD.
![VGG16-rescaling](Graphs/VGG-rescaling.jpg?raw=true "VGG16- rescaling")
![VGG16-normalization](Graphs/VGG-normalizing.jpg?raw=true "VGG16- normalization")

ImageNet colors means [123.68, 116.779, 103.939] and std = 64 
### VGG16 Regularization:

Various Dropout configurations were tested. 0.5 ,0.2, 0.3 between the last 3 Dense layers of VGG16 model. Below table shows the difference in results for each experiment.
![VGG16-Dropout](Graphs/VGG-Dropout.jpg?raw=true "VGG16- Dropout")
for experiment DO 0.5 , we noticed that validation accuracy was greater than training accuracy after 32 Epoch. which signal that a drop out of 0.5 is causing underfitting.
Dropout value of 0.3 is performing better than 0.2. with 5% difference in training accuracy, and only 1% difference in validation accuracy.
![VGG16-Dropout3](Graphs/VGG-Dropout3.jpg?raw=true "VGG16- Dropout2")
Note: a model of from DO 0.3 + Maxnorm=3  on epoch 23 was choosen for fine-tunning that had an accuracy of 41% and validation accuracy %40.

### VGG16 Fine-tunning:
Fine-tunning is the process of unfreezing chunk of layers to adjust pretrained weights to our custom dataset. 
<br>
we've unfreezed frozen layers by chunk of 4. Thus, last 8, 12, then 16 (all layers). Below chart shows the learning curve of our model. 
![VGG16-FineTunning](Graphs/VGG-FineTunning.jpg?raw=true "VGG16- FineTunning")
with Fine-Tunning, we've observed a boost in validation accuracy by %16, from %50 (maximum from feature extraction method) to %66.

![VGG16-FineTunning10](Graphs/VGG-FineTunning10.jpg?raw=true "VGG16- FineTunning10")
With further Learnning rate tweaking, we've achieved 72% validation accuracy. giving a total boost of 22%.
Although %72 is not ideal, this is largely attributed to the high number of classes 196, and low number of images per class.





