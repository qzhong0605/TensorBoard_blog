## Tensorboard的调试
### Overview
Tensorboard是Google开发的用于Deep Learning可视化的工具，可以可视化Deep Learning过程中的image、text等数据、训练产生的loss、accuracy等scalar数据、profile模型训练过程中GPU等device的使用率等情况。Tensorboard和Tensorflow之间是一个相对解藕的，也就是说Tensorboard不怎么依赖TensorFlow，其中大部分需要的是TensorFLow去解析Event文件，TensorBoard根据各个Plugins定义的操作可视化Event文件数据内容。
### Introduction 
Tensorboard是大部分采用python和javascript两种语言编写的，并内置了很多插件。目前在Tensorboard-1.13版本上包含了[Image](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/image), [Scalar](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/scalar), [Text](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/text), [Audio](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/audio), [Graph](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/graph),[Debugger](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/debugger), [Histogram](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/histogram), [Distribution](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/histogram), [Interactive_inference](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/interactive_inference), [Profile](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/profile)等。多数情况下Tensorboard插件可以满足需要，然而有时候需要根据具体情况去定制Tensorboard插件，或者开发新的TensorBoard插件，在这种情况下，需要去调试debug TensorBoard，熟悉TensorBoard的工作流程以及功能模块。TensorBoard涉及到了多进程和多线程，所以调试有点麻烦。本文分别从TensorBoard的工作流程、采用ipdb调试TensorBoard和python Logging调试TensorBoard三个方面展开
### The process for Tensorboard 
Tensorboad的主要工作就是解析event格式文件或者database数据库文件，然后可视化；当前的TensorBoard支持的数据库格式为SQLite，但是大多数情况下还是以event格式方式工作运行偏多，所以本文还是介绍以event文件格式为内容解析方式的Tensorboard的Debug流程。总体来说，本文这里把TensorBoard工作模块分成3部分：Backend、Plugins、Frontend。

* Backend：Tensorboard读取的event文件内容是一种格式化的数据文件，该文件以record方式存储，每个record同时包含了CRC信息，在一个具体的record的data内容保存的是event的message protobuf
* Plugins：Tensorboard遵循的是WSGI方式运行Web程序，每个Plugin组合多个apps（按照WSGI的方式）response每个request
* Frontend：每个Plugins返回的reponse由Frontend生成html并显示
### Debug with ipdb
### Debug with Logging 
