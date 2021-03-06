# Pneumonia Detection from Chest X-Rays
In this project, we perform pneumonia detection from chest x-ray images using state-of-the-art transformers. During the dataset analysis process, we discovered a large imbalance between the classes. To remedy this, we balanced the data through repetitions and used carefully selected augmentations to mitigate some of the issues with our balancing methodology. Instead of using traditional loss functions, we opted for the Robust Bi-Tempered Logistic Loss for a more robust decision boundary. Finally, we fine-tuned a pre-trained Vision Transformer, achieving an F1 score of 92.27%. 

## Model: Pre-Trained Vision Transformer
Until recently, most computer vision tasks have relied solely on convolutional architectures. With the [Vision Transformer (ViT)](https://arxiv.org/abs/2010.11929), researchers have now found a way of applying transformers directly to images, without any major architectural modifications. To do so, ViT first splits an image into a sequence of patches, projecting them into a sequence of embeddings. Subsequently, ViT propagates these dynamic patch embeddings through a standard transformer encoder to generate a task-specific output. The model architecture is shown in the image below.

![https://i.imgur.com/DXnJVqS.png](https://i.imgur.com/DXnJVqS.png)

The original paper explores multiple ways of projecting the inputted image into embeddings. The simplest option is to divide the image into flattened patches and transforming them using a learned linear projection. The other is to have a more hybrid approach using Convolutional Neural Networks (CNNs). The input embeddings could instead be generated by segmenting the feature maps of any CNN into patches, and projecting them into dynamic embeddings that can be propagated through a transformer.

In this project we use a pre-trained, hybrid ViT that uses a ResNet to extract feature maps from the inputted image. This allows the transformer to receive more informative features as input.

## Bi-Tempered Logistic Loss
Instead of the traditional Binary Cross-Entropy Loss function, we opted for the [Robust Bi-Tempered Logistic Loss](https://papers.nips.cc/paper/2019/hash/8cd7775f9129da8b5bf787a063d8426e-Abstract.html) published in NeurIPS 2019. This should help the model's generalisation by learning a more robust decision boundary, despite the small amount of training data.

Research shows that convex losses commonly used for classification tasks, like the softmax loss, are not robust against outliers. This is partly due to their unbounded nature. An unbound function will return high loss values, resulting in large gradient updates for misclassified outliers. Consequently, this lack of robustness results in a decision boundary that is heavily influenced by singular outlying data points.

![https://i.imgur.com/WwdKojh.png](https://i.imgur.com/WwdKojh.png)

Another issue, particularly with the softmax loss, is the exponentially decaying tail of the softmax function itself. Whenever there are outliers close to the decision boundary, the short tail of the softmax forces the model to give them higher importance. In contrast, heavy-tailed alternatives significantly improve the robustness of the loss towards such outliers.

The [Robust Bi-Tempered Logistic Loss](https://papers.nips.cc/paper/2019/hash/8cd7775f9129da8b5bf787a063d8426e-Abstract.html) addresses these issues through a bounded loss with tail-heaviness. The researchers did this by replacing the log and softmax in the softmax loss, with their respective tempered versions. The boundedness and tail-heaviness can be controlled through two hyperparameters: t1 and t2. When 0 <= t1 < 1, the loss is bounded, while when t2 > 1, the loss has a heavy tail.

![https://i.imgur.com/pWLU7hT.png](https://i.imgur.com/pWLU7hT.png)

The difference between softmax (logistic) loss and the Bi-Tempered Logistic Loss is shown in the image below.

![https://i.imgur.com/ppp02AJ.png](https://i.imgur.com/ppp02AJ.png)

