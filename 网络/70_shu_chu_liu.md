##### 7.0 输出流 {#7-0}

（1）使用输出流也可以实现和NSFileHandle相同的功能

（2）如何使用

```
    //1.创建一个数据输出流
    /*
     第一个参数：二进制的流数据要写入到哪里
     第二个参数：采用什么样的方式写入流数据，如果YES则表示追加，如果是NO则表示覆盖
     */
    NSOutputStream *stream = [NSOutputStream outputStreamToFileAtPath:fullPath append:YES];

    //只要调用了该方法就会往文件中写数据
    //如果文件不存在，那么会自动的创建一个
    [stream open];
    self.stream = stream;

    //2.当接收到数据的时候写数据
    //使用输出流写数据
    /*
     第一个参数：要写入的二进制数据
     第二个参数：要写入的数据的大小
     */
    [self.stream write:data.bytes maxLength:data.length];

    //3.当文件下载完毕的时候关闭输出流
    //关闭输出流
    [self.stream close];
    self.stream = nil;

```