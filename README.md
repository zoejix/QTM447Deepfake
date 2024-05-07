





DeepFake Image Detection with MTCNN and ViT
By
Max Cao, Alina Chen, Zoe Ji
A final project for class QTM 447 of Emory College of Arts and Sciences
Instructed by Dr. Kevin Mcalister

Acknowledgments
This paper presents a group project for QTM 447: Statistical Machine Learning II at Emory University, instructed by Dr. Kevin McAlister. The data came from the Kaggle Deepfake Detection Challenge sponsored by AWS, Facebook, Microsoft, the Partnership on AI’s Media Integrity Steering Committee, and academics. The project was solely used to demonstrate the application of the topics discussed in this class and was not used in any competition.
Introduction
The proliferation of fake news has emerged as a pressing concern in recent times, posing a significant threat to the integrity of journalism and the promotion of public discourse (Borges et al., 2018). Recent advancements in computer graphics, computer vision, and machine learning have significantly simplified the process of synthesizing convincing fake audio, images, and videos. Among them, Deepfake technologies are capable of producing AI-generated videos depicting individuals engaging in fictitious actions and statements and stand to profoundly influence the way individuals assess the credibility of online content (Somers, 2020). Such advancements in content creation and manipulation can impact public discourse quality and the protection of human rights, particularly considering their potential for misuse in spreading misinformation. Detecting manipulated media poses a complex and dynamically evolving technical challenge, necessitating collaborative efforts spanning the entire technology sector and beyond (Deepfake Detection Challenge | Kaggle, 2024).
In dealing with images and videos, Convolutional Neural Networks (CNNs), Recurrent Neural Networks (RNNs), Long Short-Term Memory (LSTM) networks, Generative Adversarial Networks (GANs), and Transformers are some of the prominent algorithms. These models are effective at analyzing spatial features or temporal patterns. Among the algorithms, Multi-Task Cascaded Convolutional Neural Networks (MTCNN) is the most accurate and popular neural network method for facial and facial landmark detection (Zhang et al., 2016). In particular, it adopts a cascaded structure with three stages of deep convolutional networks that predict face and landmark locations. 
One of the most effective CNN architectures for image recognition is the Residual Network (ResNet) (He et al., 2015). ResNet addresses the vanishing gradient problem by introducing skip connections, or shortcuts, which allow gradients to flow directly through the network without degradation, thereby enabling the training of deep networks while maintaining performance. ResNet's hierarchical feature representation capabilities make it effective for image recognition tasks including object recognition and image classification. Although Transformer architectures are used more often in natural language processing tasks, they have potential in computer vision problems as well. For example, Vision Transformer (ViT) is a transformer architecture for image recognition (Dosovitskiy et al., 2020). ViT divides an input image into fixed-size patches, flattens them into sequences, and feeds them into a transformer encoder, enabling the model to capture global spatial dependencies within the image. By leveraging self-attention mechanisms, ViT effectively learns representations of both local and global features, leading to state-of-the-art performance on various image recognition benchmarks (Dosovitskiy et al., 2020).
Method
In this section, we will describe our approach to data preprocessing, face detection, and classification.
Data Importing. The data was obtained from the Kaggle Deepfake Detection Challenge, which includes a full training set of over 470 GB (Deepfake Detection Challenge | Kaggle, 2024). We selected a subset of videos totaling 100 GB. Due to the constraint on computational power, a frame is randomly selected from each video and imported as a JPG image.
Facial Detection and Cropping. Facial detection and cropping were carried out via the MTCNN face detector for Keras in Python. We used the default MTCNN, which bundles a face detection weights model adapted from the Facenet’s MTCNN implementation. The areas of the images inside the bounding boxes were exported to a directory containing only faces (Figure 1), giving us a total of 9,403 face images.
Figure 1. Illustration of cropped face images.
Train-Test Splitting. The train-test split was carried out on the face images in a way that the faces included in the training set and the test set do not overlap, resulting in a 7958 train set size and a 1443 test set size.
Classification Model. We utilized ViT model, building from scratch. In this section, we will walk step by step through the construction of the Vision Transformer. Firstly, we implemented the image preprocessing: make images to patches. An image of size N by N has been split into (N/M)2 patches of size M by M, which represent the input words to the Transformer.
Here, for our images of size 224 by 224, we chose a patch size of 16. Hence we obtain sequences of 196 patches (we also did experiment for patch size of 32, the corresponding patch number is 49). We visualize them in Figure x. Compared to the original images, we can see that it is more difficult to recognize the objects from those patches. The model will learn how to combine these patches to recognize the objects. Notice that in this kind of input format, the inductive bias in CNNs which an image is a grid of pixels is lost. 
Next step is the construction of the Transformer model. We utilized the PyTorch module nn.MultiheadAttention. We adopted the Pre-Layer Normalization version of the Transformer blocks proposed in the study (Ruibin Xiong et al., 2020). The key idea is to apply Layer Normalization as a first layer in the residual blocks, rather than between residual blocks. This method can help to reorganize the layers, supporting better gradient flow and removing the need for a warm-up stage. 
After this, we built the following elements:
A linear projection layer that converts input patches into a larger feature vector. This layer is a simple linear layer that processes each patch of size M by M separately.
A classification token, called CLS token, is appended to the input sequence. The feature vector of this token is used to make classification. 
Learnable positional encodings are added to the tokens before they are processed by the Transformer. These encodings help in capturing positional related information, allowing the model to treat the input as a sequence rather than a set. 
An MLP(Multi-Layer Perceptron) head is utilized to map the feature vector of the CLS token to a classification output. Generally, this MLP head consists of a simple feed-forward network or a single linear layer. 
With these elements, we built the full vision transformer. Finally we put everything into a PyTorch Lightning Module. We use torch.optim.AdamW as the optimizer. AdamW is the Adam with a corrected weighted decay implementation. Usually, we need to use a learning rate warmup for Transformer. But since we employ the Pre-LN Transformer version, we don’t need a warm up stage. 
PyTorch Lightning. Our code utilizes the library PyTorch Lightning. PyTorch Lightning is a framework that simplifies the code for training and testing in PyTorch. It handles logging into Tensorboard and saving the checkpoints automatically. 
Numerical Experiments
Resnet
We first did experiments on two pre-trained resnet models: resnet50 and resnet101, both of which were imported from models of torchvision. Adam is used as an optimizer with learning rate 0.001. We employ the cross entropy loss as the criterion. Since we are using a pre-trained model, we will optimize with respect to the last layer. The performance of resnet50 is extremely bad with only 34% testing accuracy, which also indicates the difficulty of deep fake classification tasks. Although, resnet101 gives a better accuracy compared to resnet50, we see a serious overfitting issue with validation accuracy 88% and testing accuracy 76%. 
ViT
The Vision Transformers usually are applied to large scale image classification problems such as ImageNet. However, due to constraint of computational resources, we have relatively little data. We also wanted to take a step back and investigate if the Vision Transformer can succeed on a relatively small dataset like what we have. 
In our implementation, we have some hyperparameters needed to be tuned. We will introduce each hyperparameter and its influence: 
Patch size. Notice that by making smaller patches, we will have longer input sequences into the Transformer. Although it allows the Transformer to model more complex functions, it demands a long computational time with quadratic memory usage in the attention layer. Additionally, using patches that are too small will lead to difficulties for Transformer to learn which patches are close to each other. But we want to emphasize that relatively small patches could help us to learn details of an image. We have done experiments with patch size 2, 4, 8, 16, and 32. We found for this classification task, 16 patches result in the best performance.
Embedding and hidden dimension. We found that these two have a similar influence on a Transformer as to an MLP. If we increase the sizes, we will have a more complex model, which takes longer to train. Due to the usage of Multi-Head Attention in the Transformer, we also need to consider the size of query and key. Note that each key has the feature dimensionality of embedded dimension/number of heads . If we have an input sequence with length 64, for example, a minimum size for the key vectors should be 16 or 32. If we set the dimension to be low, we may restrain the effect of attention maps. During the experiments, for our case, using more than 8 heads does not give us better performance. Therefore, we set an embedding dimension to be 256 and a hidden dimension of 512. This choice is also driven by several tutorials we have read online. 
The learning rate. The learning rate for Transformers, as we have seen in several popular online tutorials, is quite small. In different papers, a common value is 3e-5. To reduce the overfitting, we utilize a dropout ratio 0.2.
We can not guarantee these choices are the optimal ones. But among several trails, we found these choices currently give us decent accuracy. If we have more GPU resources, we may run trials more times and further fine-tuning those parameters. 
Results. We reach test accuracy 81% and validation accuracy 87%. We observe that the both training accuracy and validation accuracy curve are very oscillatory, different from normal patterns we saw from other classification tasks. We suspect that it might be caused by the nature of deep fake classification itself. Additionally, we are concerned about whether the Vision Transformer learns the right embeddings. For a Convolutional Neural Network (CNN), it assumes two pixels that are close to each other are more related than two pixels far away from each other. 
Reference
Borges, L., Martins, B., & Calado, P. (2018). Combining Similarity Features and Deep Representation Learning for Stance Detection in the Context of Checking Fake News. ArXiv.org. https://arxiv.org/abs/1811.00706
Somers, Meredith. (2020, July 21). Deepfakes, explained | MIT Sloan. MIT Sloan. https://mitsloan.mit.edu/ideas-made-to-matter/deepfakes-explained
Deepfake Detection Challenge | Kaggle. (2024). Kaggle.com. https://www.kaggle.com/competitions/deepfake-detection-challenge/overview
Zhang, K., Zhang, Z., Li, Z., & Qiao, Y. (2016). Joint Face Detection and Alignment Using Multitask Cascaded Convolutional Networks. IEEE Signal Processing Letters, 23(10), 1499–1503. https://doi.org/10.1109/lsp.2016.2603342
He, K., Zhang, X., Ren, S., & Sun, J. (2015). Deep Residual Learning for Image Recognition. ArXiv.org. https://arxiv.org/abs/1512.03385
Dosovitskiy, A., Beyer, L., Kolesnikov, A., Weissenborn, D., Zhai, X., Unterthiner, T., Dehghani, M., Minderer, M., Heigold, G., Gelly, S., Uszkoreit, J., & Houlsby, N. (2020). An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale. ArXiv.org. https://arxiv.org/abs/2010.11929
Xiong, Ruibin, et al. “On layer normalization in the transformer architecture.” International Conference on Machine Learning. PMLR, 2020.


‌


‌


