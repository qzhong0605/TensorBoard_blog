## Tensorboard的调试
Tensorboard是Google开发的用于Deep Learning可视化的工具，可以可视化Deep Learning过程中的image、text等数据、训练产生的loss、accuracy等scalar数据、profile模型训练过程中GPU等device的使用率等情况。Tensorboard和Tensorflow之间是一个相对解藕的，也就是说Tensorboard不怎么依赖TensorFlow，其中大部分需要的是TensorFLow去解析Event文件，TensorBoard根据各个Plugins定义的操作可视化Event文件数据内容。
### Introduction 
### The process for Tensorboard 
### Debug with ipdb
### Debug with Logging 
