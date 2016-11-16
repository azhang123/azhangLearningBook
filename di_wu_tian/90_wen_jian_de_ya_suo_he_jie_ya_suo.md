##### 9.0 文件的压缩和解压缩 {#9-0}

（1）说明

```
使用ZipArchive来压缩和解压缩文件需要添加依赖库（libz）,使用需要包含SSZipArchive文件，如果使用cocoaPoads来安装框架，那么会自动的配置框架的使用环境

```

（2）相关代码

```
//压缩文件的第一种方式
/*
 第一个参数：压缩文件要保存的位置
 第二个参数：要压缩哪几个文件
 */
[SSZipArchive createZipFileAtPath:fullpath withFilesAtPaths:arrayM];

//压缩文件的第二种方式
/*
 第一个参数：文件压缩到哪个地方
 第二个参数：要压缩文件的全路径
 */
[SSZipArchive createZipFileAtPath:fullpath withContentsOfDirectory:zipFile];

//如何对压缩文件进行解压
/*
 第一个参数：要解压的文件
 第二个参数：要解压到什么地方
 */
[SSZipArchive unzipFileAtPath:unZipFile toDestination:fullpath];
`

```