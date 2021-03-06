# EasyLogger 核心功能 API 说明

---

所有核心功能API接口都在[`\easylogger\inc\elog.h`](https://github.com/armink/EasyLogger/blob/master/easylogger/inc/elog.h)中声明。以下内容较多，可以使用 **CTRL+F** 搜索。

> 建议：点击项目主页 https://github.com/armink/EasyLogger 右上角 **Watch & Star**，这样项目有更新时，会及时以邮件形式通知你。

## 1、用户使用接口

### 1.1 初始化

初始化的EasyLogger的核心功能，初始化后才可以使用下面的API。

```
ElogErrCode elog_init(void)
```

### 1.2 启动

**注意**：在初始化完成后，必须调用启动方法，日志才会被输出。

```
void elog_start(void)
```

### 1.3 输出日志

所有日志的级别关系大小如下：

```
级别 标识 描述
0    [A]  断言(Assert)
1    [E]  错误(Error)
2    [W]  警告(Warn)
3    [I]  信息(Info)
4    [D]  调试(Debug)
5    [V]  详细(Verbose)
```

所有级别的日志输出方法如下，每种级别都有一种简写方式，用户可以自行选择。

```c
#define elog_assert(tag, ...) 
#define elog_a(tag, ...) //简写

#define elog_error(tag, ...)
#define elog_e(tag, ...)

#define elog_warn(tag, ...)
#define elog_w(tag, ...)

#define elog_info(tag, ...)
#define elog_i(tag, ...)

#define elog_debug(tag, ...)
#define elog_d(tag, ...)

#define elog_verbose(tag, ...)
#define elog_v(tag, ...)
```

|参数                                    |描述|
|:-----                                  |:----|
|tag                                     |日志标签|
|...                                     |不定参格式，与`printf`入参一致，放入将要输出日志|

**建议**：对于每个文件或者每个模块，可以重新覆盖定义上述日志输出宏定义，如下所示。这样的优点就是降低代码书写量，统一日志书写格式，代码可以做到尽可能少的依赖某个日志库，同时部分复用代码无需修改日志输出方法，可直接拷贝至其他模块或者其他项目，提高软件的可重用性。

```c
//WiFi协议处理(/wifi/proto.c)
#define LOG_TAG    "wifi.proto"
#define log_e(...) elog_e(LOG_TAG, __VA_ARGS__)
#define log_w(...) elog_w(LOG_TAG, __VA_ARGS__)
#define log_i(...) elog_i(LOG_TAG, __VA_ARGS__)

#if WIFI_DEBUG
    #define log_d(...) elog_d(LOG_TAG, __VA_ARGS__)
#else
    #define log_d(...)
#endif

//WiFi数据打包处理(/wifi/package.c)
#define LOG_TAG    "wifi.package"

#if WIFI_DEBUG
    #define log_d(...) elog_d(LOG_TAG, __VA_ARGS__)
#else
    #define log_d(...)
#endif

//CAN命令解析(/can/disp.c)
#define LOG_TAG    "can.disp"
#define log_e(...) elog_e(LOG_TAG, __VA_ARGS__)

#if CAN_DEBUG
    #define log_d(...) elog_d(LOG_TAG, __VA_ARGS__)
#else
    #define log_d(...)
#endif

```
### 1.4 断言

EasyLogger自带的断言，可以直接用户软件，在断言表达式不成立后会输出断言信息并保持`while(1)`，或者执行断言钩子方法，钩子方法的设定参考 [`elog_assert_set_hook`](#114-设置断言钩子方法)。

```
#define ELOG_ASSERT(EXPR)
```

|参数                                    |描述|
|:-----                                  |:----|
|EXPR                                    |表达式|

### 1.5 使能/失能日志输出

```
void elog_set_output_enabled(bool enabled)
```
|参数                                    |描述|
|:-----                                  |:----|
|enabled                                 |true: 使能，false: 失能|

### 1.6 获取日志使能状态

```
bool elog_get_output_enabled(void)
```

### 1.7 设置日志格式

每种级别可对应一种日志输出格式，日志的输出内容位置顺序固定，只可定义开启或关闭某子内容。可设置的日志子内容包括：级别、标签、时间、进程信息、线程信息、文件路径、行号、方法名。

> 注：默认为 RAW格式

```
void elog_set_fmt(uint8_t level, size_t set)
```

|参数                                    |描述|
|:-----                                  |:----|
|level                                   |级别|
|set                                     |格式集合|

例子：

```c
/* 断言：输出所有内容 */
elog_set_fmt(ELOG_LVL_ASSERT, ELOG_FMT_ALL);
/* 错误：输出级别、标签和时间 */
elog_set_fmt(ELOG_LVL_ERROR, ELOG_FMT_LVL | ELOG_FMT_TAG | ELOG_FMT_TIME);
/* 警告：输出级别、标签和时间 */
elog_set_fmt(ELOG_LVL_WARN, ELOG_FMT_LVL | ELOG_FMT_TAG | ELOG_FMT_TIME);
/* 信息：输出级别、标签和时间 */
elog_set_fmt(ELOG_LVL_INFO, ELOG_FMT_LVL | ELOG_FMT_TAG | ELOG_FMT_TIME);
/* 调试：输出除了方法名之外的所有内容 */
elog_set_fmt(ELOG_LVL_DEBUG, ELOG_FMT_ALL & ~ELOG_FMT_FUNC);
/* 详细：输出除了方法名之外的所有内容 */
elog_set_fmt(ELOG_LVL_VERBOSE, ELOG_FMT_ALL & ~ELOG_FMT_FUNC);
```

### 1.8 设置过滤级别

默认过滤级别为5(详细)，用户可以任意设置。在设置高优先级后，低优先级的日志将不会输出。例如：设置当前过滤的优先级为3(警告)，则只会输出优先级别为警告、错误、断言的日志。

```
void elog_set_filter_lvl(uint8_t level)
```

|参数                                    |描述|
|:-----                                  |:----|
|level                                   |级别|

### 1.9 设置过滤标签

默认过滤标签为空字符串(`""`)，即不过滤。当前输出日志的标签会与过滤标签做字符串匹配，日志的标签包含过滤标签，则该输出该日志。例如：设置过滤标签为WiFi，则系统中包含WiFi字样标签的（WiFi.Bsp、WiFi.Protocol、Setting.Wifi）日志都会被输出。

>注：RAW格式日志不支持标签过滤

```
void elog_set_filter_tag(const char *tag)
```

|参数                                    |描述|
|:-----                                  |:----|
|tag                                     |标签|

### 1.10 设置过滤关键词

默认过滤关键词为空字符串("")，即不过滤。检索当前输出日志中是否包含该关键词，包含则允许输出。

> 注：对于配置较低的MCU建议不开启关键词过滤（默认为不过滤），增加关键字过滤将会在很大程度上减低日志的输出效率。实际上当需要实时查看日志时，过滤关键词功能交给上位机做会更轻松，所以后期的跨平台日志助手开发完成后，就无需该功能。

```
void elog_set_filter_kw(const char *keyword)
```

|参数                                    |描述|
|:-----                                  |:----|
|keyword                                 |关键词|

### 1.11 设置过滤器

设置过滤器后，只输出符合过滤要求的日志。所有参数设置方法，可以参考上述3个章节。

```
void elog_set_filter(uint8_t level, const char *tag, const char *keyword)
```

|参数                                    |描述|
|:-----                                  |:----|
|level                                   |级别|
|tag                                     |标签|
|keyword                                 |关键词|

### 1.12 输出RAW格式日志

```
void elog_raw(const char *format, ...)
```
|参数                                    |描述|
|:-----                                  |:----|
|format                                  |样式，类似`printf`首个入参|
|...                                     |不定参|

### 1.13 使能/失能日志输出锁

默认为使能状态，当系统或MCU进入异常后，需要输出异常日志时，就必须失能日志输出锁，来保证异常日志能够被正常输出。

```
void elog_output_lock_enabled(bool enabled)
```

|参数                                    |描述|
|:-----                                  |:----|
|enabled                                 |true: 使能，false: 失能|

例子：

```c
/* EasyLogger断言钩子方法 */
static void elog_user_assert_hook(const char* ex, const char* func, size_t line) {
    /* 失能日志输出锁 */
    elog_output_lock_enabled(false);
    /* 失能EasyLogger的Flash插件自带同步锁（Flash插件自带方法） */
    elog_flash_lock_enabled(false);
    /* 输出断言信息 */
    elog_a("elog", "(%s) has assert failed at %s:%ld.\n", ex, func, line);
    /* 将缓冲区中所有日志保存至Flash（Flash插件自带方法） */
    elog_flash_flush();
    while(1);
}
```

### 1.14 设置断言钩子方法

默认断言钩子方法为空，设置断言钩子方法后。当断言`ELOG_ASSERT(EXPR)`中的条件不满足时，会自动执行断言钩子方法。断言钩子方法定义及使用可以参考上一章节的例子。

```c
void elog_assert_set_hook(void (*hook)(const char* expr, const char* func, size_t line))
```

|参数                                    |描述|
|:-----                                  |:----|
|hook                                    |断言钩子方法|

### 1.15 使能/失能日志颜色

日志颜色功能是将各个级别日志按照颜色进行区分，默认颜色功能是关闭的。日志的颜色修改方法详见《EasyLogger 移植说明》中的 `设置参数` 章节。

```
void elog_set_text_color_enabled(bool enabled)
```

|参数                                    |描述|
|:-----                                  |:----|
|enabled                                 |true: 使能，false: 失能|

### 1.16 将缓冲区中的日志全部输出

在缓冲输出模式下，执行此方法可以将缓冲区中的日志全部输出干净。

```
void elog_flush(void)
```

### 1.17 日志输出接口

在异步输出模式下，如果用户没有启动 pthread 库，此时需要启用额外线程来实现日志的异步输出功能。使用此方法即可获取到异步输出缓冲区中的指定长度的日志。如果设定日志长度小于日志缓冲区中已存在日志长度，将只会返回已存在日志长度。

```C
size_t elog_async_get_log(char *log, size_t size)
```

|参数                                    |描述|
|:-----                                  |:----|
|log                                     |取出的日志内容|
|size                                    |待取出的日志大小|

## 2、配置

参照 《EasyLogger 移植说明》（[`\docs\zh\port\kernel.md`](https://github.com/armink/EasyLogger/blob/master/docs/zh/port/kernel.md)）中的 `设置参数` 章节
