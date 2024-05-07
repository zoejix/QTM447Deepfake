# DeepFake Image Detection with MTCNN and ViT
This repo presents a group project for QTM 447: Statistical Machine Learning II at Emory University, instructed by Dr. Kevin McAlister. The data came from the Kaggle Deepfake Detection Challenge sponsored by AWS, Facebook, Microsoft, the Partnership on AI’s Media Integrity Steering Committee, and academics. The project was solely used to demonstrate the application of the topics discussed in this class and was not used in any competition.
### Collaborators
* Max Cao
* Alina Chen
* Zoe Ji
#### -- Project Status: [Completed]

## Project Intro/Objective
The proliferation of fake news has emerged as a pressing concern in recent times, posing a significant threat to the integrity of journalism and the promotion of public discourse (Borges et al., 2018). Recent advancements in computer graphics, computer vision, and machine learning have significantly simplified the process of synthesizing convincing fake audio, images, and videos. Among them, Deepfake technologies are capable of producing AI-generated videos depicting individuals engaging in fictitious actions and statements and stand to profoundly influence the way individuals assess the credibility of online content (Somers, 2020). Such advancements in content creation and manipulation can impact public discourse quality and the protection of human rights, particularly considering their potential for misuse in spreading misinformation. Detecting manipulated media poses a complex and dynamically evolving technical challenge, necessitating collaborative efforts spanning the entire technology sector and beyond (Deepfake Detection Challenge | Kaggle, 2024).

## Project Description

### Data Importing and Preprocessing
1. The data was obtained from the Kaggle Deepfake Detection Challenge, which includes a full training set of over 470 GB (Deepfake Detection Challenge | Kaggle, 2024). We selected a subset of videos totaling 100 GB. Due to the constraint on computational power, a frame is randomly selected from each video and imported as a JPG image.
2. Facial detection and cropping were carried out via the MTCNN face detector for Keras in Python. We used the default MTCNN, which bundles a face detection weights model adapted from the Facenet’s MTCNN implementation. The areas of the images inside the bounding boxes were exported to a directory containing only faces (Figure 1), giving us a total of 9,403 face images.

### Train-Test Splitting
The train-test split was carried out on the face images in a way that the faces included in the training set and the test set do not overlap, resulting in a 7958 train set size and a 1443 test set size.

### Classification Model. 
We utilized ViT model, building from scratch and training with PyTorch Lightning framework. For details of implementation, please refer to our project report or the code [ViT_Deepfake.ipynb](./Code/ViT_Deepfake.ipynb)

### Numerical Experiments and results
1. Resnet
We did experiments on two pre-trained resnet models: resnet50 and resnet101. The performance of resnet50 is extremely bad with only 34% testing accuracy, which also indicates the difficulty of deep fake classification tasks. Although, resnet101 gives a better accuracy compared to resnet50, we see a serious overfitting issue with validation accuracy 88% and testing accuracy 76%. 
2. ViT
We reach test accuracy 81% and validation accuracy 87%. We observe that the both training accuracy and validation accuracy curve are very oscillatory, different from normal patterns we saw from other classification tasks. We carefully explained the process of how we chose the hyperparameters and discussed our results in the final report. 
‌


‌


