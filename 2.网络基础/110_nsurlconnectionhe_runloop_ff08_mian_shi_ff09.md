##### 11.0 NSURLConnection和Runloop（面试） {#11-0-nsurlconnection-runloop}

（1）两种为NSURLConnection设置代理方式的区别

```
    //第一种设置方式：
    //通过该方法设置代理，会自动的发送请求
    // [[NSURLConnection alloc]initWithRequest:request delegate:self];

    //第二种设置方式：
    //设置代理，startImmediately为NO的时候，该方法不会自动发送请求
    NSURLConnection *connect = [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:NO];
    //手动通过代码的方式来发送请求
    //注意该方法内部会自动的把connect添加到当前线程的RunLoop中在默认模式下执行
    [connect start];

```

（2）如何控制代理方法在哪个线程调用

```
    //说明：默认情况下，代理方法会在主线程中进行调用（为了方便开发者拿到数据后处理一些刷新UI的操作不需要考虑到线程间通信）
    //设置代理方法的执行队列
    [connect setDelegateQueue:[[NSOperationQueue alloc]init]];

```

（3）开子线程发送网络请求的注意点，适用于自动发送网络请求模式

```

//在子线程中发送网络请求-调用startf方法发送
-(void)createNewThreadSendConnect1
{
    //1.创建一个非主队列
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];

    //2.封装操作，并把任务添加到队列中执行
    [queue addOperationWithBlock:^{

        NSLog(@"%@",[NSThread currentThread]);
        //2-1.确定请求路径
        NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=dd&pwd=ww&type=JSON"];

        //2-2.创建请求对象
        NSURLRequest *request = [NSURLRequest requestWithURL:url];

        //2-3.使用NSURLConnection设置代理，发送网络请求
        NSURLConnection *connection = [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:YES];

        //2-4.设置代理方法在哪个队列中执行，如果是非主队列，那么代理方法将再子线程中执行
        [connection setDelegateQueue:[[NSOperationQueue alloc]init]];

        //2-5.发送网络请求
        //注意：start方法内部会把当前的connect对象作为一个source添加到当前线程对应的runloop中
        //区别在于，如果调用start方法开发送网络请求，那么再添加source的过程中，如果当前runloop不存在
        //那么该方法内部会自动创建一个当前线程对应的runloop,并启动。
        [connection start];

    }];
}

//在子线程中发送网络请求-自动发送网络请求
-(void)createNewThreadSendConnect2
{
    NSLog(@"-----");
    //1.创建一个非主队列
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];

    //2.封装操作，并把任务添加到队列中执行
    [queue addOperationWithBlock:^{

        //2-1.确定请求路径
        NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=dd&pwd=ww&type=JSON"];

        //2-2.创建请求对象
        NSURLRequest *request = [NSURLRequest requestWithURL:url];

        //2-3.使用NSURLConnection设置代理，发送网络请求
        //注意：该方法内部虽然会把connection添加到runloop,但是如果当前的runloop不存在，那么不会主动创建。
        NSURLConnection *connection = [NSURLConnection connectionWithRequest:request delegate:self];

        //2-4.设置代理方法在哪个队列中执行，如果是非主队列，那么代理方法将再子线程中执行
        [connection setDelegateQueue:[[NSOperationQueue alloc]init]];

        //2-5 创建当前线程对应的runloop,并开启
       [[NSRunLoop currentRunLoop]run];
    }];
}

```

**Illegal HTML tag removed :** require([&quot;gitbook&quot;], function(gitbook) { var config = {&quot;fontSettings&quot;:{&quot;theme&quot;:null,&quot;family&quot;:&quot;sans&quot;,&quot;size&quot;:2}}; gitbook.start(config); });