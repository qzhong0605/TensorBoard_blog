## Concepts for TensorBoard
### Overview 
TensorBoard是一个很好辅助Deep Learning训练和推断过程的可视化框架，但是TensorBoard整体包括数据存储、解析、响应客户端请求；
为了更好的理解TensorBoard，本文主要介绍TensorBoard运行和使用过程中会用到的名词解释，希望帮助理解TensorBoard的系统架构和设计，
可以定制和开发新的TensorBoard插件
### Event
TensorBoard读取文件、解析文件的格式是采用event作为最小数据单元，具体的event包括 file version、graph_def、summary、log_message、
session_log等；event采取的是protobuf定义的方式
```protobuf
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
### Record
Tensorboard读取、解析的文件是由record组成的字节流，每个record里面包含具体的event数据，具体的record格式如下：
```cpp
  // Format of a single record:
  //  uint64    length
  //  uint32    masked crc of length
  //  byte      data[length]
  //  uint32    masked crc of data
```
### EventFile 
TensorBoard 读取、解析的文件是EventFile，每个EventFile由Record组成，也就是Event包含多个Record，一个EventFile第一个Record是一个file
version的record; EventFile文件名字如下：
``` EventFile name
/some/file/path/my.file.out.events.[timestamp].[hostname][suffix]
``` 
### Plugin
TensorBoard 采取分拆多个模块实现可视化功能，而模块的重要一个环节就是plugin，一个plugin响应客户端的request和产生对应的response，一个
plugin需要实现如下功能：
``` python
def get_plugin_apps(self): 
  pass
def is_active(self):
  pass
```
### WSGI
WSGI是Web Server Gateway Interface, 它是一种规范说明，用于明确web server和web applications之间的通信方式以及web applications之间
协同工作处理来自客户端的请求。它是用于运行python的web service。整体WSGI流程如下：
![WSGI](concepts/web-browser-server-wsgi.png)
### Summary
### FileVersion
### Run
