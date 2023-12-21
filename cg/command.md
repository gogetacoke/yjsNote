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

# 查询文件是否被篡改

```
md5sum pass 
a805b3a06ecb1136339695ea840e830e  pass
vim pass # 添加一个a
md5sum pass
a805b3a06ecb1136339695ea840e830e  pass

# 使用md5生成hash值，可以有效保证文件数据的安全性
md5sum /etc/*.conf | md5sum # 给etc下所有.conf结尾的文件生成hash值，在管道给md5sum生成一个hash，etc下.conf内容发生改变后，可以进行对比
3d6e2c8ab635014f1a5c2fb7841a6abd  -

# 生成hash值其余命令：sha256sum、sha512sum
```

# 查询剧本格式是否正确

```shell
ansible-playbook xx.yml --syntax-check
```

