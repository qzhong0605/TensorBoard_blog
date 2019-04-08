## Inspect the EventFile for TensorBoard
TensorBoard通过读取磁盘文件和数据库两种方式来可视化Deep Learning相关的数据，包括data、feature distribution等。本文主要是从磁盘文件方式考虑TensorBoard如何解析数据
### Overview
TensorBoard可视化的数据文件是以record格式存储数据文件。其他程序，主要方式通过TensorFlow产生Event数据文件，Event数据文件的组织方式采用每次run构成一个目录，多个run构成整体的TensorBoard可视化的文件数据。

具体表现如下两种数据文件组织方式：
* 目录方式
```
.
├── test                                            # 代表test的run
│   └── events.out.tfevents.1553153829.gpu80
└── train                                           # 代表train的run
    └── events.out.tfevents.1553153829.gpu80
```
* 单独文件方式
```
events.out.tfevents.1553153829.gpu80
```
### The Format of EventFile
TensorBoard可以识别的文件格式是以record方式来组织数据，每个record的data部分信息是event数据，整个文件是一个字节流。
* 文件record格式
```
  // Format of a single record:
  //  uint64    length  
  //  uint32    masked crc of length
  //  byte      data[length]
  //  uint32    masked crc of data
```
* Event 的 protobuf形式
``` protobuf
message Event {
  // Timestamp of the event.
  double wall_time = 1;

  // Global step of the event.
  int64 step = 2;

  oneof what {
    // An event file was started, with the specified version.
    // This is use to identify the contents of the record IO files
    // easily.  Current version is "brain.Event:2".  All versions
    // start with "brain.Event:".
    string file_version = 3;
    // An encoded version of a GraphDef.
    bytes graph_def = 4;
    // A summary was generated.
    Summary summary = 5;
    // The user output a log message. Not all messages are logged, only ones
    // generated via the Python tensorboard_logging module.
    LogMessage log_message = 6;
    // The state of the session which can be used for restarting after crashes.
    SessionLog session_log = 7;
    // The metadata returned by running a session.run() call.
    TaggedRunMetadata tagged_run_metadata = 8;
    // An encoded version of a MetaGraphDef.
    bytes meta_graph_def = 9;
  }
}
```
* 文件的开头部分是一个文件version的record，是一个event的record，包含`brain.Event:2`这样的string
### How to parse the event
直接用python方式可以解析event数据内容
* 读取解析eventfile字节流
```
[nav] In [1]: from tensorboard.compat.tensorflow_stub.pywrap_tensorflow import PyRecordReader_New

[ins] In [2]: reader = PyRecordReader_New("events.out.tfevents.1554430482.gpu80")

[ins] In [3]: reader.read()
```
* 解析 Event数据内容
```
[ins] In [4]: from tensorboard.compat.proto import event_pb2

[ins] In [5]: event = event_pb2.Event()

[ins] In [7]: event.ParseFromString(reader.event_strs[1])
Out[7]: 74305
```
* FileVersion Record
```
[ins] In [9]: event.ParseFromString(reader.event_strs[0])
Out[9]: 24

[ins] In [10]: event
Out[10]:
wall_time: 1554430482.0
file_version: "brain.Event:2"
```
