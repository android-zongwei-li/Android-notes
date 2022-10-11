VMware 和 ubuntu 的安装见：

https://blog.csdn.net/davidhzq/article/details/102575343

下面介绍在 ubuntu 上安装 git 和 Java jdk 。

#### 一、安装 git

```
sudo apt-get install git-core gnupg
```

#### 二、安装 Java jdk

> 可以选择安装 openjdk 或者 oracle jdk 。

##### 1、安装 openjdk

1、安装 openjdk

```
sudo apt-get update
```

2、安装 openjdk-8-jdk

```
sudo apt-get install openjdk-8-jdk
```

3、查看 java 版本，看看是否安装成功

```
java -version
```

![image-20210416153906646](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416153906646.png)



##### 2、手动下载压缩包安装oracle Java JDK

1、前往oracle Java官网下载JDK（http://www.oracle.com/technetwork/java/javase/downloads/index.html）

推荐华为镜像地址：https://mirrors.huaweicloud.com/java/jdk/

下载好了默认在：

![image-20210416164207090](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416164207090.png)

2、解压缩到指定目录（以jdk-8u192-linux-x64.tar.gz为例）

- 创建目录

```
sudo mkdir /usr/lib/jvm
```

> 如果之前安装过 openjdk，此目录可能已经存在。

- 解压到该目录
  首先要 cd Downloads，然后执行：

```
sudo tar -zxvf jdk-8u192-linux-x64.gz -C /usr/lib/jvm
```

![image-20210416164408274](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416164408274.png)

3、修改环境变量

```
sudo gegit ~/.bashrc
```

![image-20210416164720057](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416164720057.png)

打开文件后，在文件末尾加入以下内容：

```
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_192  ## 这里要注意目录要换成自己解压的jdk 目录
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

![image-20210416165013956](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416165013956.png)

使环境变量马上生效：（进入到jdk的目录中）

![image-20210416164011439](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416164011439.png)

4、查看 Java -version，发现已经从 openjdk 变成了 oracle jdk。见上图。



#### 三、其他依赖包

![image-20210416170947391](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416170947391.png)

相较于 《Android系统源码情景分析》少了一个 libwxgitk2.6-dev（报错）。后面可能会有问题。

![image-20210416171134153](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210416171134153.png)



#### 四、下载 Android 源码

   1.打开终端

   2.依次输入以下命令：

```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo   #使用tuna的git-repo镜像
chmod a+x ~/bin/repo
```

   3.打开bin文件夹下的repo文件，将

```
REPO_URL = 'https://gerrit.googlesource.com/git-repo'
```

改为

```
REPO_URL = 'https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

见：

https://www.cnblogs.com/shenchanghui/p/8503623.html







参考：

1、[Linux之Ubuntu18.04安装Java JDK8的三种方式](https://blog.csdn.net/zbj18314469395/article/details/86064849)

如果后续遇到需要切换 jdk 版本，可以查看下这篇，看看能不能解决。



2、[Ubuntu激活密钥](https://zhidao.baidu.com/question/432413156276109052.html)

