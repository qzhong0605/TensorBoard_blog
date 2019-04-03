## MNIST: An hello-world example for Tensorflow with Tensorboard
Tensorboard 是一种可视化的工具，可以用来很好的辅助模型训练，提高模型训练的效率，熟练掌握Tensorboard工具对于从事Deep Learning开发很有必要。本文主要通过MNIST这个示例来说明TensorBoard的使用，以及Tensorboard的构建编译等过程。
### Overview
Deep Learning一般包括Train和Inference两个阶段：在Train的过程，根据输入数据和optimization方法，采用iterative method得到最后的模型参数；在Inference的过程，根据输入的数据，模型产生结果。然而模型train的过程，需要不断调整参数，比如learning rate、weight decay等，为了更好的查看当前模型的收敛情况，收集模型训练过程中产生的data信息，并可视化他们，可以很好的有利于模型训练。

在Deep Learning中，MNIST是一个包括60000张train图片和10000张test图片，图片大小为28*28*1,规模比较小，很适合作为Deep Learning领域的hello-world示例。
### Requirement for Installation
### Build
### Run with Tensorboard
