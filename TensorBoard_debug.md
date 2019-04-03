## Tensorboard的调试
### Overview
Tensorboard是Google开发的用于Deep Learning可视化的工具，可以可视化Deep Learning过程中的image、text等数据、训练产生的loss、accuracy等scalar数据、profile模型训练过程中GPU等device的使用率等情况。Tensorboard和Tensorflow之间是一个相对解藕的，也就是说Tensorboard不怎么依赖TensorFlow，其中大部分需要的是TensorFLow去解析Event文件，TensorBoard根据各个Plugins定义的操作可视化Event文件数据内容。
### Introduction 
Tensorboard是大部分采用python和javascript两种语言编写的，并内置了很多插件。目前在Tensorboard-1.13版本上包含了[Image](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/image), [Scalar](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/scalar), [Text](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/text), [Audio](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/audio), [Graph](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/graph),[Debugger](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/debugger), [Histogram](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/histogram), [Distribution](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/histogram), [Interactive_inference](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/interactive_inference), [Profile](https://github.com/tensorflow/tensorboard/tree/master/tensorboard/plugins/profile)等。多数情况下Tensorboard插件可以满足需要，然而有时候需要根据具体情况去定制Tensorboard插件，或者开发新的TensorBoard插件，在这种情况下，需要去调试debug TensorBoard，熟悉TensorBoard的工作流程以及功能模块。TensorBoard涉及到了多进程和多线程，所以调试有点麻烦。本文分别[The process for Tensorboard](#The-process-for-Tensorboard)、[Requirement for debugging Tensorboard](#Requirement-for-debugging-Tensorboard)、[Debug with ipdb](#Debug-with-ipdb)和[Debug with Logging](#Debug-with-Logging)三个方面展开
### The process for Tensorboard
Tensorboad的主要工作就是解析event格式文件或者database数据库文件，然后可视化；当前的TensorBoard支持的数据库格式为SQLite，但是大多数情况下还是以event格式方式工作运行偏多，所以本文还是介绍以event文件格式为内容解析方式的Tensorboard的Debug流程。总体来说，本文这里把TensorBoard工作模块分成3部分：Backend、Plugins、Frontend。

* Backend：Tensorboard读取的event文件内容是一种格式化的数据文件，该文件以record方式存储，每个record同时包含了CRC信息，在一个具体的record的data内容保存的是event的message protobuf
* Plugins：Tensorboard遵循的是WSGI方式运行Web程序，每个Plugin组合多个apps（按照WSGI的方式）response每个request
* Frontend：每个Plugins返回的reponse由Frontend生成html并显示
### Requirement for debugging Tensorboard
Tensorboard编译需要bazel，因此为了能够调试需要安装bazel，bazel版本尽量在0.22以上。
编译TensorBoard的方式如下
```
bazel build //tensorboard
```
运行Tensorboard方式如下：
```
bazel run tensorboard -- --logdir=/tmp/images_demo/box_to_gaussian  --port=10050
```
查看所有的命令行选项
```
bazel run tensorboard -- --helpfull
```
### Debug with ipdb
ipdb是pdb的增强版本，提供了更加友好的提示功能和颜色高亮。TensorBoard运行过程中默认生成了一个守护线程/进程来加载和解析数据，所以采用python的ipdb module方式并无法调试Tensorboard。为了能够采用ipdb调试,采用在python文件引入函数调用的方式。

下面以event process 过程来说明ipdb的使用。在`tensorboard/backend/event_processing/event_file_loader.py`文件的`class RawEventFileLoader`的`Load`方法加入如下代码：
``` Load function
 48   def Load(self):
 49     """Loads all new events from disk as raw serialized proto bytestrings. \n
 50
 51     Calling Load multiple times in a row will not 'drop' events as long as the
 52     return value is not iterated over.
 53
 54     Yields:
 55       All event proto bytestrings in the file that have not been yielded yet.
 56     """
 57     import ipdb; ipdb.set_trace() # Debug Event
 58     logger.debug('Loading events from %s', self._file_path)
 ```
运行如下命令，重新build和运行tensorboard
``` build and run tensorboard
bazel run tensorboard -- --logdir=/tmp/images_demo/box_to_gaussian  --port=10050
```
这样在运行过程中就可以看到TensorBoard的event读取过程
``` view the backtrace 
     57     import ipdb; ipdb.set_trace() # Debug Event
---> 58     logger.debug('Loading events from %s', self._file_path)
     59
```
接下来就是正常的ipdb的命令，具体的ipdb的操作可以使用help
``` ipdb help 
ipdb> help

Documented commands (type help <topic>):
========================================
EOF    cl         disable  interact  next    psource  rv         unt
a      clear      display  j         p       q        s          until
alias  commands   down     jump      pdef    quit     source     up
args   condition  enable   l         pdoc    r        step       w
b      cont       exit     list      pfile   restart  tbreak     whatis
break  continue   h        ll        pinfo   return   u          where
bt     d          help     longlist  pinfo2  retval   unalias
c      debug      ignore   n         pp      run      undisplay

Miscellaneous help topics:
==========================
exec  pdb
```
### Debug with Logging 
Tensorboard 除了后台的event和plugin等处理方式还需要frontend响应来自客户端的http request，这部分操作采用ipdb调试虽然也可以，但是会带来流程混乱，采用日志的方式可以更好的掌握Tensorboard数据处理流程。Tensorboard的日志处理方式是`python`的 `Logging package` ，通过设置Log Level方式查看日志内容来分析TensorBoard的流程。

Tensorboard设置Log Level方式在文件 `tensorboard/util/tb_logging.py`中，可以通过修改如下方式，获取日志信息
``` set log level
 20 _logger = logging.getLogger('tensorboard')
 21 # for debugging
 22 _logger.setLevel(logging.DEBUG)
```
可以获取到如下日志信息，
``` Log info 
I0403 17:41:01.068085 139926410622720 application.py:287] Plugin listing: is_active() for scalars took 0.000 seconds
I0403 17:41:01.068258 139926410622720 application.py:288] Request: <Request 'http://10.60.1.80:10050/data/plugins_listing' [GET]>  Response: {'core': True, 'beholder': False, 'scalars': False}
I0403 17:41:01.068446 139926410622720 application.py:287] Plugin listing: is_active() for custom_scalars took 0.000 seconds
```
