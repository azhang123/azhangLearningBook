##### 10.0 文件的上传 {#10-0}

*   10.1 文件上传步骤

    ```
      （1）确定请求路径
      （2）根据URL创建一个可变的请求对象
      （3）设置请求对象，修改请求方式为POST
      （4）设置请求头，告诉服务器我们将要上传文件（Content-Type）
      （5）设置请求体（在请求体中按照既定的格式拼接要上传的文件参数和非文件参数等数据）
          001 拼接文件参数
          002 拼接非文件参数
          003 添加结尾标记
      （6）使用NSURLConnection sendAsync发送异步请求上传文件
      （7）解析服务器返回的数据

    ```

*   10.2 文件上传设置请求体的数据格式

    ```
      //请求体拼接格式
      //分隔符：----WebKitFormBoundaryhBDKBUWBHnAgvz9c

      //01.文件参数拼接格式

       --分隔符
       Content-Disposition:参数
       Content-Type:参数
       空行
       文件参数

      //02.非文件拼接参数
       --分隔符
       Content-Disposition:参数
       空行
       非文件的二进制数据

      //03.结尾标识
      --分隔符--

    ```

*   10.3 文件上传相关代码

```
- (void)upload
{
    //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/upload"];

    //2.创建一个可变的请求对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];

    //3.设置请求方式为POST
    request.HTTPMethod = @"POST";

    //4.设置请求头
    NSString *filed = [NSString stringWithFormat:@"multipart/form-data; boundary=%@",Kboundary];
    [request setValue:filed forHTTPHeaderField:@"Content-Type"];

    //5.设置请求体
    NSMutableData *data = [NSMutableData data];
    //5.1 文件参数
    /*
     --分隔符
     Content-Disposition:参数
     Content-Type:参数
     空行
     文件参数
     */
    [data appendData:[[NSString stringWithFormat:@"--%@",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
    [data appendData:KnewLine];
    [data appendData:[@"Content-Disposition: form-data; name=\"file\"; filename=\"test.png\"" dataUsingEncoding:NSUTF8StringEncoding]];
    [data appendData:KnewLine];
    [data appendData:[@"Content-Type: image/png" dataUsingEncoding:NSUTF8StringEncoding]];
    [data appendData:KnewLine];
    [data appendData:KnewLine];
    [data appendData:KnewLine];

    UIImage *image = [UIImage imageNamed:@"test"];
    NSData *imageData = UIImagePNGRepresentation(image);
    [data appendData:imageData];
    [data appendData:KnewLine];

    //5.2 非文件参数
    /*
     --分隔符
     Content-Disposition:参数
     空行
     非文件参数的二进制数据
     */

    [data appendData:[[NSString stringWithFormat:@"--%@",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
    [data appendData:KnewLine];
    [data appendData:[@"Content-Disposition: form-data; name=\"username\"" dataUsingEncoding:NSUTF8StringEncoding]];
    [data appendData:KnewLine];
    [data appendData:KnewLine];
    [data appendData:KnewLine];

    NSData *nameData = [@"wendingding" dataUsingEncoding:NSUTF8StringEncoding];
    [data appendData:nameData];
    [data appendData:KnewLine];

    //5.3 结尾标识
    //--分隔符--
    [data appendData:[[NSString stringWithFormat:@"--%@--",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
    [data appendData:KnewLine];

    request.HTTPBody = data;

    //6.发送请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * __nullable response, NSData * __nullable data, NSError * __nullable connectionError) {

        //7.解析服务器返回的数据
        NSLog(@"%@",[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }];
}

```

*   10.4 如何获得文件的MIMEType类型

（1）直接对该对象发送一个异步网络请求，在响应头中通过response.MIMEType拿到文件的MIMEType类型

```
//如果想要及时拿到该数据，那么可以发送一个同步请求
- (NSString *)getMIMEType
{
    NSString *filePath = @"/Users/文顶顶/Desktop/备课/其它/swift.md";

    NSURLResponse *response = nil;
    [NSURLConnection sendSynchronousRequest:[NSURLRequest requestWithURL:[NSURL fileURLWithPath:filePath]] returningResponse:&response error:nil];
    return response.MIMEType;
}

//对该文件发送一个异步请求，拿到文件的MIMEType
- (void)MIMEType
{

    //    NSString *file = @"file:///Users/文顶顶/Desktop/test.png";

    [NSURLConnection sendAsynchronousRequest:[NSURLRequest requestWithURL:[NSURL fileURLWithPath:@"/Users/文顶顶/Desktop/test.png"]] queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * __nullable response, NSData * __nullable data, NSError * __nullable connectionError) {
        //       response.MIMEType
        NSLog(@"%@",response.MIMEType);

    }];
}

```

（2）通过UTTypeCopyPreferredTagWithClass方法

```
//注意：需要依赖于框架MobileCoreServices
- (NSString *)mimeTypeForFileAtPath:(NSString *)path
{
    if (![[[NSFileManager alloc] init] fileExistsAtPath:path]) {
        return nil;
    }

    CFStringRef UTI = UTTypeCreatePreferredIdentifierForTag(kUTTagClassFilenameExtension, (__bridge CFStringRef)[path pathExtension], NULL);
    CFStringRef MIMEType = UTTypeCopyPreferredTagWithClass (UTI, kUTTagClassMIMEType);
    CFRelease(UTI);
    if (!MIMEType) {
        return @"application/octet-stream";
    }
    return (__bridge NSString *)(MIMEType);
}

```