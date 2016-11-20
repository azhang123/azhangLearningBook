##### 6.0 大文件断点下载 {#6-0}

（1）实现思路

```
在下载文件的时候不再是整块的从头开始下载，而是看当前文件已经下载到哪个地方，然后从该地方接着往后面下载。可以通过在请求对象中设置请求头实现。

```

（2）解决方案（设置请求头）

```
//2.创建请求对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];

    //2.1 设置下载文件的某一部分
    // 只要设置HTTP请求头的Range属性, 就可以实现从指定位置开始下载
    /*
     表示头500个字节：Range: bytes=0-499
     表示第二个500字节：Range: bytes=500-999
     表示最后500个字节：Range: bytes=-500
     表示500字节以后的范围：Range: bytes=500-
     */
    NSString *range = [NSString stringWithFormat:@"bytes=%zd-",self.currentLength];
    [request setValue:range forHTTPHeaderField:@"Range"];

```

（3）注意点（下载进度并判断是否需要重新创建文件）

```
//获得当前要下载文件的总大小（通过响应头得到）
    NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;

    //注意点：res.expectedContentLength获得是本次请求要下载的文件的大小（并非是完整的文件的大小）
    //因此：文件的总大小 == 本次要下载的文件大小+已经下载的文件的大小
    self.totalLength = res.expectedContentLength + self.currentLength;

    NSLog(@"----------------------------%zd",self.totalLength);

    //0 判断当前是否已经下载过，如果当前文件已经存在，那么直接返回
    if (self.currentLength >0) {
        return;
    }

```