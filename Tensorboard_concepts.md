## Concepts for TensorBoard
### Overview 
TensorBoard是一个很好辅助Deep Learning训练和推断过程的可视化框架，但是TensorBoard整体包括数据存储、解析、响应客户端请求；
为了更好的理解TensorBoard，本文主要介绍TensorBoard运行和使用过程中会用到的名词解释，希望帮助理解TensorBoard的系统架构和设计，
可以定制和开发新的TensorBoard插件
### Summary
Summary 信息是在模型训练过程中产生，用于辅助模型训练，目前Summary信息包括 Image、Histogram、Audio、TensorProto，产生Summary的方式包括OP和PB（protobuf）这两种，具体的Summary的定义如下：
``` protobuf
message Summary {
  message Image {
    // Dimensions of the image.
    int32 height = 1;
    int32 width = 2;
    // Valid colorspace values are
    //   1 - grayscale
    //   2 - grayscale + alpha
    //   3 - RGB
    //   4 - RGBA
    //   5 - DIGITAL_YUV
    //   6 - BGRA
    int32 colorspace = 3;
    // Image data in encoded format.  All image formats supported by
    // image_codec::CoderUtil can be stored here.
    bytes encoded_image_string = 4;
  }

  message Audio {
    // Sample rate of the audio in Hz.
    float sample_rate = 1;
    // Number of channels of audio.
    int64 num_channels = 2;
    // Length of the audio in frames (samples per channel).
    int64 length_frames = 3;
    // Encoded audio data and its associated RFC 2045 content type (e.g.
    // "audio/wav").
    bytes encoded_audio_string = 4;
    string content_type = 5;
  }
  message Value {
    // This field is deprecated and will not be set.
    string node_name = 7;

    // Tag name for the data. Used by TensorBoard plugins to organize data. Tags
    // are often organized by scope (which contains slashes to convey
    // hierarchy). For example: foo/bar/0
    string tag = 1;

    // Contains metadata on the summary value such as which plugins may use it.
    // Take note that many summary values may lack a metadata field. This is
    // because the FileWriter only keeps a metadata object on the first summary
    // value with a certain tag for each tag. TensorBoard then remembers which
    // tags are associated with which plugins. This saves space.
    SummaryMetadata metadata = 9;

    // Value associated with the tag.
    oneof value {
      float simple_value = 2;
      bytes obsolete_old_style_histogram = 3;
      Image image = 4;
      HistogramProto histo = 5;
      Audio audio = 6;
      TensorProto tensor = 8;
    }
  }

  // Set of values for the summary.
  repeated Value value = 1;
}
```
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
### FileVersion
FileVersion是一个record，是存放于EventFile文件的第一个记录，是一种event格式的数据类型
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
### Tag 
Tag 是一个字符串，是TensorBoard用来组织Tensorboard内部数据；Tag的名字是唯一的，而且Tag的名字组织方式是层次化，比如 `foo/bar/0`
### API layer
API Layer 是用于告诉如何生成生成TensorBoard可以解析的数据内容，目前TensorBoard支持的数据格式包括EventFile和SQLite。
### Backend
Backend 是TensorBoard用于读取解析数据文件内容、处理来自Tensorboard client的请求的模块，具体处理方式为`python`
### Frontend
Frontend 是 Tensorboard用于显示TensorBoard处理来自client的请求之后的response的模块，具体处理方式包括html、js等
