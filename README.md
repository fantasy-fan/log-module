## A Logging Library

一个用于C++的日志模块，主要用于学习目的，具备以下特性：

* 针对Linux系统开发
* 采用C风格的API
* 支持多线程
* 支持多种日志级别
* 日志打印采用C标准库printf的方式
* 自己管理缓冲区
* 支持按照周期自动切分日志
* 使用专门的线程，采用异步方式将日志写入磁盘

## Quick Example

```C

    /* 初始化日志模块 */
    if (log_init() != 0) {
        exit(EXIT_FAILURE);
    }

    /* 设置日志级别, 具体级别见头文件 */
    log_set_level("INFO");

    /* 启用日志缓冲 */
    log_set_cache(true);

    /* 设置日志自动切分周期，默认按天切分，这里设置为按分钟切分 */
    log_set_rotate_cycle("M");

    /* 设置日志存储目录, 以及日志文件名称
     * 例如下面例子中, 日志文件会保存在 ./log/Shakespeare_50.log中 
     */
    if (log_set_prefix(".", "Shakespeare_50") != 0) {
        exit(EXIT_FAILURE);
    }   
    
    logError("%s", "As if by some instinct the wretch did know");
    logError("His rider lov'd not speed being made from thee.'");

    logWarning("%s", "The bloody spur cannot provoke him on,");
    logWarning("That sometimes anger thrusts into his hide,");
        
    logNotice("%s", "Which heavily he answers with a groan,");
    logNotice("More sharp to me than spurring to his side;");

    logInfo("%s","For that same groan doth put this in my mind,");
    logInfo("My grief lies onward, and my joy behind.");
    
    /* 卸载日志模块 */
    log_destroy();
```

## Logging API

See logger.h for detail.

## Comment

本日志模块编写目的只是为了学习。 

本模块的编写，比起开源的logging module，如log4cpp，log4cxx，glog等，并不成熟完备。但
是如果从学习的角度，却是个不错的开始。与其将模块写的更通用，我更倾向于将程序写的更清
晰，以便可以随时根据自己的需要修改，实现代码快速演化。我希望如果他人有缘使用此模块，
可以快速的理解其中的概念，并在此基础上定制自己的logging module。

我想logging module的使用，第一条就是尽量少写日志。因为日志太多，使用者往往懒得去看日
志的内容。当然，日志如果是供专门程序分析除外，即便如此，日志内容也要尽量精简。因为比
起内存，磁盘操作相当低效。如果实在需要打印大量日志，则需要合理选择日志级别。比如INFO
级别日志会优先写入内存缓冲，而ERROR级别日志则需要马上写入磁盘。因此日志是供程序分析
之用，用INFO级别显然更合适。

最初编写，是受余庆的FastDFS中logger module启发。 但是很不巧，应用恰巧是CPU密集型，
当缓冲区写满，或由于日志级别需要，强制写入磁盘时， 会产生很大的时间延迟。之前有些
担心线程间锁的争用对性能的影响，现在看来果然磁盘IO才是最大的问题。因为日志模块，
导致程序性能下降，实在有些心有不甘。

在进行改进时，受陈硕的多线程编程指南文章的启发， 采用独立的线程将缓冲数据写入磁盘，
并采用阻塞队列作为线程间传递数据的数据结构，大大提高了日志模块响应时间。实际上，这
里采用异步IO（AIO）似乎更合适，可惜Linux异步IO实现似乎并不成熟，效果如何难以预期，
故暂不采用异步IO。

在编写过程中，深深感到多线程程序在使用中，线程间同步多有不便，颇不自然。在模块卸载时，
还是可能存在一些同步问题，虽然可以解决，但不免将程序结构变得更复杂。但是我同意一些人
的看法，程序退出时，可以容忍一些瑕疵出现，只要无关大雅即可。 如果应用对时间并非十分敏
感，个人还是更喜欢第一版的单线程版本，简单自然。过些天将第一版也放到github上来。


