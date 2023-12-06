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

