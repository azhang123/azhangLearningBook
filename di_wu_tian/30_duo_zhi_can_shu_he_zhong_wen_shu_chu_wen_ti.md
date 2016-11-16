##### 3.0 多值参数和中文输出问题 {#3-0}

（1）多值参数如何设置请求路径

```
//多值参数
/*
 如果一个参数对应着多个值，那么直接按照"参数=值&参数=值"的方式拼接
 */
-(void)test
{
    //1.确定URL
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/weather?place=Beijing&place=Guangzhou"];
    //2.创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //3.发送请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {

        //4.解析
        NSLog(@"%@",[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }];
}

```

（2）如何解决字典和数组中输出乱码的问题

```
答：给字典和数组添加一个分类，重写descriptionWithLocale方法，在该方法中拼接元素格式化输出。
-(nonnull NSString *)descriptionWithLocale:(nullable id)locale

```