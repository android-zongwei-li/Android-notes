# 前言

- `Socket`的使用在 `Android`网络编程中非常重要
- 今天我将带大家全面了解 `Socket` 及 其使用方法

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-59926986f3c800e0.png?imageMogr2/auto-orient/strip|imageView2/2/w/994/format/webp)

示意图

------

# 1.网络基础

- 阅读本文前，请先了解 关于计算机网络基础，如计算机体系结构、`TCP`、`UDP`等知识
- 具体请看文章：[这是一份详细 & 清晰的计算机网络基础 学习指南](https://www.jianshu.com/p/45d27f3e1196)

------

# 2. Socket定义

- 即套接字，**是应用层 与 `TCP/IP` 协议族通信的中间软件抽象层，表现为一个封装了 TCP / IP协议族 的编程接口（API）**

![img](https:////upload-images.jianshu.io/upload_images/944365-1a92e10c6c694d0f.png?imageMogr2/auto-orient/strip|imageView2/2/w/545/format/webp)

示意图

> 1. `Socket`不是一种协议，而是一个编程调用接口（`API`），属于传输层（主要解决数据如何在网络中传输）
> 2. 即：通过`Socket`，我们才能在Andorid平台上通过 `TCP/IP`协议进行开发
> 3. 对用户来说，只需调用Socket去组织数据，以符合指定的协议，即可通信

- 成对出现，一对套接字：



```ruby
Socket ={(IP地址1:PORT端口号)，(IP地址2:PORT端口号)}
```

- 一个 `Socket` 实例 唯一代表一个主机上的一个应用程序的通信链路

------

# 3. 建立Socket连接过程

![img](https:////upload-images.jianshu.io/upload_images/944365-eb79b5a461ac71b5.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

示意图

------

# 4. 原理

`Socket`的使用类型主要有两种：

- 流套接字（`streamsocket`） ：基于 `TCP`协议，采用 **流的方式** 提供可靠的字节流服务
- 数据报套接字(`datagramsocket`)：基于 `UDP`协议，采用 **数据报文** 提供数据打包发送的服务

具体原理图如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-8df0ed7afe6b32d1.png?imageMogr2/auto-orient/strip|imageView2/2/w/610/format/webp)

原理图

------

# 5. Socket 与 Http 对比

- `Socket`属于传输层，因为 `TCP / IP`协议属于传输层，**解决的是数据如何在网络中传输的问题**
- `HTTP`协议 属于 应用层，**解决的是如何包装数据**

由于二者不属于同一层面，所以本来是没有可比性的。但随着发展，默认的Http里封装了下面几层的使用，所以才会出现`Socket` & `HTTP`协议的对比：（主要是工作方式的不同）：

- `Http`：采用 **请求—响应** 方式。

> 1. 即建立网络连接后，当 客户端 向 服务器 发送请求后，服务器端才能向客户端返回数据。
> 2. 可理解为：**是客户端有需要才进行通信**

- `Socket`：采用 **服务器主动发送数据** 的方式

> 1. 即建立网络连接后，服务器可主动发送消息给客户端，而不需要由客户端向服务器发送请求
> 2. 可理解为：**是服务器端有需要才进行通信**

------

# 6. 使用步骤

- `Socket`可基于`TCP`或者`UDP`协议，**但TCP更加常用**
- 所以下面的使用步骤 & 实例的`Socket`将基于`TCP`协议



```go
// 步骤1：创建客户端 & 服务器的连接

    // 创建Socket对象 & 指定服务端的IP及端口号 
    Socket socket = new Socket("192.168.1.32", 1989);  

    // 判断客户端和服务器是否连接成功  
    socket.isConnected());

                     
// 步骤2：客户端 & 服务器 通信
// 通信包括：客户端 接收服务器的数据 & 发送数据 到 服务器

    <-- 操作1：接收服务器的数据 -->
        
            // 步骤1：创建输入流对象InputStream
            InputStream is = socket.getInputStream() 

            // 步骤2：创建输入流读取器对象 并传入输入流对象
            // 该对象作用：获取服务器返回的数据
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader br = new BufferedReader(isr);

            // 步骤3：通过输入流读取器对象 接收服务器发送过来的数据
            br.readLine()；


    <-- 操作2：发送数据 到 服务器 -->                  

            // 步骤1：从Socket 获得输出流对象OutputStream
            // 该对象作用：发送数据
            OutputStream outputStream = socket.getOutputStream(); 

            // 步骤2：写入需要发送的数据到输出流对象中
            outputStream.write（（"Carson_Ho"+"\n"）.getBytes("utf-8")）；
            // 特别注意：数据的结尾加上换行符才可让服务器端的readline()停止阻塞

            // 步骤3：发送数据到服务端 
            outputStream.flush();  


// 步骤3：断开客户端 & 服务器 连接

             os.close();
            // 断开 客户端发送到服务器 的连接，即关闭输出流对象OutputStream

            br.close();
            // 断开 服务器发送到客户端 的连接，即关闭输入流读取器对象BufferedReader

            socket.close();
            // 最终关闭整个Socket连接
```

------

# 7. 具体实例

- 实例 `Demo` 代码包括：客户端 & 服务器
- **本文着重讲解客户端**，服务器仅采用最简单的写法进行展示

### 7.1 客户端 实现

**步骤1：加入网络权限**



```xml
<uses-permission android:name="android.permission.INTERNET" />
```

**步骤2：主布局界面设置**

> 包括创建Socket连接、客户端 & 服务器通信的按钮



```objectivec
    <Button
        android:id="@+id/connect"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="connect" />

    <Button
        android:id="@+id/disconnect"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="disconnect" />

    <TextView
        android:id="@+id/receive_message"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/Receive"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Receive from message" />

    <EditText
        android:id="@+id/edit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/send"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="send"/>
```

**步骤3：创建Socket连接、客户端 & 服务器通信**

> 具体请看注释

*MainActivity.java*



```java
package scut.carson_ho.socket_carson;

import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MainActivity extends AppCompatActivity {

    /**
     * 主 变量
     */

    // 主线程Handler
    // 用于将从服务器获取的消息显示出来
    private Handler mMainHandler;

    // Socket变量
    private Socket socket;

    // 线程池
    // 为了方便展示,此处直接采用线程池进行线程管理,而没有一个个开线程
    private ExecutorService mThreadPool;

    /**
     * 接收服务器消息 变量
     */
    // 输入流对象
    InputStream is;

    // 输入流读取器对象
    InputStreamReader isr ;
    BufferedReader br ;

    // 接收服务器发送过来的消息
    String response;


    /**
     * 发送消息到服务器 变量
     */
    // 输出流对象
    OutputStream outputStream;

    /**
     * 按钮 变量
     */

    // 连接 断开连接 发送数据到服务器 的按钮变量
    private Button btnConnect, btnDisconnect, btnSend;

    // 显示接收服务器消息 按钮
    private TextView Receive,receive_message;

    // 输入需要发送的消息 输入框
    private EditText mEdit;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        /**
         * 初始化操作
         */

        // 初始化所有按钮
        btnConnect = (Button) findViewById(R.id.connect);
        btnDisconnect = (Button) findViewById(R.id.disconnect);
        btnSend = (Button) findViewById(R.id.send);
        mEdit = (EditText) findViewById(R.id.edit);
        receive_message = (TextView) findViewById(R.id.receive_message);
        Receive = (Button) findViewById(R.id.Receive);

        // 初始化线程池
        mThreadPool = Executors.newCachedThreadPool();


        // 实例化主线程,用于更新接收过来的消息
        mMainHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case 0:
                        receive_message.setText(response);
                        break;
                }
            }
        };


        /**
         * 创建客户端 & 服务器的连接
         */
        btnConnect.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // 利用线程池直接开启一个线程 & 执行该线程
                mThreadPool.execute(new Runnable() {
                    @Override
                    public void run() {

                        try {

                            // 创建Socket对象 & 指定服务端的IP 及 端口号
                            socket = new Socket("192.168.1.172", 8989);

                            // 判断客户端和服务器是否连接成功
                            System.out.println(socket.isConnected());

                        } catch (IOException e) {
                            e.printStackTrace();
                        }

                    }
                });

            }
        });

        /**
         * 接收 服务器消息
         */
        Receive.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // 利用线程池直接开启一个线程 & 执行该线程
                mThreadPool.execute(new Runnable() {
                    @Override
                    public void run() {

                          try {
                            // 步骤1：创建输入流对象InputStream
                            is = socket.getInputStream();

                              // 步骤2：创建输入流读取器对象 并传入输入流对象
                              // 该对象作用：获取服务器返回的数据
                              isr = new InputStreamReader(is);
                              br = new BufferedReader(isr);

                              // 步骤3：通过输入流读取器对象 接收服务器发送过来的数据
                              response = br.readLine();

                              // 步骤4:通知主线程,将接收的消息显示到界面
                              Message msg = Message.obtain();
                              msg.what = 0;
                              mMainHandler.sendMessage(msg);

                        } catch (IOException e) {
                            e.printStackTrace();
                        }

                    }
                });

            }
        });


        /**
         * 发送消息 给 服务器
         */
        btnSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // 利用线程池直接开启一个线程 & 执行该线程
                mThreadPool.execute(new Runnable() {
                    @Override
                    public void run() {

                        try {
                            // 步骤1：从Socket 获得输出流对象OutputStream
                            // 该对象作用：发送数据
                            outputStream = socket.getOutputStream();

                            // 步骤2：写入需要发送的数据到输出流对象中
                            outputStream.write((mEdit.getText().toString()+"\n").getBytes("utf-8"));
                            // 特别注意：数据的结尾加上换行符才可让服务器端的readline()停止阻塞

                            // 步骤3：发送数据到服务端
                            outputStream.flush();

                        } catch (IOException e) {
                            e.printStackTrace();
                        }

                    }
                });

            }
        });


        /**
         * 断开客户端 & 服务器的连接
         */
        btnDisconnect.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                try {
                    // 断开 客户端发送到服务器 的连接，即关闭输出流对象OutputStream
                    outputStream.close();

                    // 断开 服务器发送到客户端 的连接，即关闭输入流读取器对象BufferedReader
                    br.close();

                    // 最终关闭整个Socket连接
                    socket.close();

                    // 判断客户端和服务器是否已经断开连接
                    System.out.println(socket.isConnected());

                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        });


    }
}
```

### 7.2 服务器 实现

- 因本文主要讲解客户端，所以服务器仅仅是为了配合客户端展示；
- 为了简化服务器使用，此处采用`Mina`框架

> 1. 服务器代码请在`eclipse`平台运行
> 2. 按照我的步骤一步步实现就可以无脑运行了

**步骤1：导入`Mina`包**

请直接移步到百度网盘：[下载链接（密码: q73e）](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1i50xChv)

![img](https:////upload-images.jianshu.io/upload_images/944365-90b634041b47b84a.png?imageMogr2/auto-orient/strip|imageView2/2/w/798/format/webp)

示意图

**步骤2：创建服务器线程**
 *TestHandler.java*



```java
package mina;
// 导入包

public class TestHandler extends IoHandlerAdapter {

    @Override
    public void exceptionCaught(IoSession session, Throwable cause) throws Exception {
        System.out.println("exceptionCaught: " + cause);
    }

    @Override
    public void messageReceived(IoSession session, Object message) throws Exception {
        System.out.println("recieve : " + (String) message);
        session.write("hello I am server");
    }

    @Override
    public void messageSent(IoSession session, Object message) throws Exception {

    }

    @Override
    public void sessionClosed(IoSession session) throws Exception {
        System.out.println("sessionClosed");
    }

    @Override
    public void sessionOpened(IoSession session) throws Exception {
        System.out.println("sessionOpen");
    }

    @Override
    public void sessionIdle(IoSession session, IdleStatus status) throws Exception {
    }

}
```

**步骤3：创建服务器主代码**
 *TestHandler.java*



```java
package mina;

import java.io.IOException;
import java.net.InetSocketAddress;

import org.apache.mina.filter.codec.ProtocolCodecFilter;
import org.apache.mina.filter.codec.textline.TextLineCodecFactory;
import org.apache.mina.transport.socket.nio.NioSocketAcceptor;

public class TestServer {
    public static void main(String[] args) {
        NioSocketAcceptor acceptor = null;
        try {
            acceptor = new NioSocketAcceptor();
            acceptor.setHandler(new TestHandler());
            acceptor.getFilterChain().addLast("mFilter", new ProtocolCodecFilter(new TextLineCodecFactory()));
            acceptor.setReuseAddress(true);
            acceptor.bind(new InetSocketAddress(8989));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

至此，客户端 & 服务器的代码均实现完毕。

------

### 7.3 测试结果

- 点击 `Connect`按钮： 连接成功

![img](https:////upload-images.jianshu.io/upload_images/944365-0f69bb561fbca4f1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

- 输入发送的消息，点击 `Send` 按钮发送

![img](https:////upload-images.jianshu.io/upload_images/944365-7438aa8f57a738ed.png?imageMogr2/auto-orient/strip|imageView2/2/w/374/format/webp)

示意图

- 服务器接收到客户端发送的消息

![img](https:////upload-images.jianshu.io/upload_images/944365-4d7ec8242eeb9c47.png?imageMogr2/auto-orient/strip|imageView2/2/w/1031/format/webp)

示意图

- 点击 `Receive From Message`按钮，客户端 读取 服务器返回的消息

![img](https:////upload-images.jianshu.io/upload_images/944365-399d395e3995810c.png?imageMogr2/auto-orient/strip|imageView2/2/w/380/format/webp)

示意图

- 点击 `DisConnect`按钮，断开 客户端 & 服务器的连接

![img](https:////upload-images.jianshu.io/upload_images/944365-b30bd33353835786.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

客户端示意图

![img](https:////upload-images.jianshu.io/upload_images/944365-67d0d9eaa71cc3f5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1033/format/webp)

### 7.4 源码地址

[Carson_Ho的Github地址：Socket具体实例](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FCarson-Ho%2FSocket_learning)

------

# 8. 总结

- 相信大家已经非常了解关于Socket的使用



# 参考

[Android：这是一份很详细的Socket使用攻略](https://www.jianshu.com/p/089fb79e308b)