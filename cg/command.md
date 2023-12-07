# 随机生成6位数密码

```
strings /dev/urandom | tr -cd '_a-zA-Z0-9' | head -c 6 

strings 动态查看文件
/dev/urandom 随机生成字符
tr 截取字符  -cd 取反删除
head -c 指定输出几个字符
```

# tar打包排除不需要的文件

```
> tar -zcf test.tar.gz --exclude=*.txt /te
```

# ZIP格式压缩解压

```
unzip xx.zip -d 指定解压路径 # 解压
unzip xx.zip xx xx # 压缩

-p：解压到标准输出，同时显示解压进度。
-q：解压时静默模式，不显示详细输出信息。
-v：显示压缩文件的详细信息，而不进行解压操作。
-t：测试压缩文件是否有效，但不进行解压操作。
-d：指定解压目录。
-j：跳过压缩文件中的目录结构，直接将文件解压到当前目录。
```

