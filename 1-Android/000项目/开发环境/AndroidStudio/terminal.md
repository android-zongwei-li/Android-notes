# 环境配置

配置Java jdk 环境：将jdk/bin目录（java.bat程序路径）加到环境变量 PATH 中。



# 乱码解决

cmd中添加命令

 -Dfile.encoding=utf-8 

```kotlin
java -Dfile.encoding=utf-8 GCRootLocalVariable.java
```

terminal中：试了往上的各种方案，都不行