##### 5.0 大文件的下载 {#5-0}

（1）实现思路

```
边接收数据边写文件以解决内存越来越大的问题

```

（2）核心代码

```

//当接收到服务器响应的时候调用，该方法只会调用一次
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    //0.获得当前要下载文件的总大小（通过响应头得到）
    NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;
    self.totalLength = res.expectedContentLength;
    NSLog(@"%zd",self.totalLength);

    //创建一个新的文件，用来当接收到服务器返回数据的时候往该文件中写入数据
    //1.获取文件管理者
    NSFileManager *manager = [NSFileManager defaultManager];

    //2.拼接文件的全路径
    //caches文件夹路径
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];

    NSString *fullPath = [caches stringByAppendingPathComponent:res.suggestedFilename];
    self.fullPath  = fullPath;
    //3.创建一个空的文件
    [manager createFileAtPath:fullPath contents:nil attributes:nil];

}
//当接收到服务器返回的数据时会调用
//该方法可能会被调用多次
-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{

    //1.创建一个用来向文件中写数据的文件句柄
    //注意当下载完成之后，该文件句柄需要关闭，调用closeFile方法
    NSFileHandle *handle = [NSFileHandle fileHandleForWritingAtPath:self.fullPath];

    //2.设置写数据的位置(追加)
    [handle seekToEndOfFile];

    //3.写数据
    [handle writeData:data];

    //4.计算当前文件的下载进度
    self.currentLength += data.length;

    NSLog(@"%f",1.0* self.currentLength/self.totalLength);
    self.progressView.progress = 1.0* self.currentLength/self.totalLength;
}

```