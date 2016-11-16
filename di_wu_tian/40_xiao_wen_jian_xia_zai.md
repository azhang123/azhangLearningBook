##### 4.0 小文件下载 {#4-0}

（1）第一种方式（NSData）

```
//使用NSDta直接加载网络上的url资源（不考虑线程）
-(void)dataDownload
{
    //1.确定资源路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_01.png"];

    //2.根据URL加载对应的资源
    NSData *data = [NSData dataWithContentsOfURL:url];

    //3.转换并显示数据
    UIImage *image = [UIImage imageWithData:data];
    self.imageView.image = image;

}

```

（2）第二种方式（NSURLConnection-sendAsync）

```
//使用NSURLConnection发送异步请求下载文件资源
-(void)connectDownload
{
    //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_01.png"];

    //2.创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //3.使用NSURLConnection发送一个异步请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
        //4.拿到并处理数据
        UIImage *image = [UIImage imageWithData:data];
        self.imageView.image = image;

    }];

}

```

（3）第三种方式（NSURLConnection-delegate）

```
//使用NSURLConnection设置代理发送异步请求的方式下载文件
-(void)connectionDelegateDownload
{
    //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];

    //2.创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //3.使用NSURLConnection设置代理并发送异步请求
    [NSURLConnection connectionWithRequest:request delegate:self];

}

#pragma mark--NSURLConnectionDataDelegate

//当接收到服务器响应的时候调用，该方法只会调用一次
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    //创建一个容器，用来接收服务器返回的数据
    self.fileData = [NSMutableData data];

    //获得当前要下载文件的总大小（通过响应头得到）
    NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;
    self.totalLength = res.expectedContentLength;
    NSLog(@"%zd",self.totalLength);

    //拿到服务器端推荐的文件名称
    self.fileName = res.suggestedFilename;

}
//当接收到服务器返回的数据时会调用
//该方法可能会被调用多次
-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
//    NSLog(@"%s",__func__);

    //拼接每次下载的数据
    [self.fileData appendData:data];

    //计算当前下载进度并刷新UI显示
    self.currentLength = self.fileData.length;

    NSLog(@"%f",1.0* self.currentLength/self.totalLength);
    self.progressView.progress = 1.0* self.currentLength/self.totalLength;

}
//当网络请求结束之后调用
-(void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    //文件下载完毕把接受到的文件数据写入到沙盒中保存

    //1.确定要保存文件的全路径
    //caches文件夹路径
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];

    NSString *fullPath = [caches stringByAppendingPathComponent:self.fileName];

    //2.写数据到文件中
    [self.fileData writeToFile:fullPath atomically:YES];

    NSLog(@"%@",fullPath);
}

//当请求失败的时候调用该方法
-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
    NSLog(@"%s",__func__);
}

```