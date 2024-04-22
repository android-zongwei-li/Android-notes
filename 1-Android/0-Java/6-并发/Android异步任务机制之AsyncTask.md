特别说明：在Android 10.0+中，AsyncTask已经标记为弃用了。
# 一、什么是AsyncTask？

AsyncTask是Android提供的轻量级（只是代码上轻量一些，而实际上要比handler更耗资源）异步任务类。它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。

AsyncTask 是一个抽象的泛型类，它提供了 Params、Progress 和 Result 这三个泛型参数，其中 Params 表示参数的类型，Progress 表示后台任务的执行进度和类型，而 Result 则表示后台任务的返回结果的类型，如果 AsyncTask 不需要传递具体的参数，那么这三个泛型参数可以用 Void 来代替。

它的出现是为了降低异步通信的开发难度。使用AsyncTask可以忽略Looper、MessageQueue、Handler 等复杂对象，更便捷的完成异步耗时操作。

问题：什么是异步通信？

# 二、消息机制与AsyncTask

要弄懂Android中的消息通信机制，需要掌握主线程的刷新机制，Thread，Handler，Looper，MessageQueue，Message以及AsyncTask等知识点。

AsyncTask适用于执行较为简单的耗时操作，如倒计时、下载等。我们只需掌握几个方法就可以完成消息通信，降低了消息通信难度。

# 三、如何使用AsyncTask
掌握AsyncTask的基本概念后，就是学习它的使用了。

##它的使用有以下步骤：
1.新建内部类继承AsyncTask 。 
2.定义AsyncTask的三种泛型参数。
3.重写方法。
4.在需要启动的地方调用execute方法。

##AsyncTask的泛型参数说明：
Params：启动任务执行的输入参数
Progress：后台任务执行的百分比
Result：后台执行任务最终返回的结果

##AsyncTask常用方法介绍
| AsyncTask常用方法 |                          调用时机                           |                           方法说明                           |
| :---------------: | :---------------------------------------------------------: | :----------------------------------------------------------: |
|   onPreExecute    |          异步任务开始执行时，系统最先调用此方法。           |      此方法运行在主线程中，可以对控件进行初始化等操作。      |
|  dolnBackground   |          执行完onPreExecute方法后，系统执行此方法           |    此方法运行在子线程中，比较耗时的操作放在此方法中执行。    |
| onProgressUpdate  | 触发此方法，需要在dolnBackground中使用publishProgress方法。 | 此方法运行在主线程中，可以修改控件状态，例如: 显示当前进度，适用于下载或扫描这类需要实时显示进度的需求。 |
|   onPostExecute   |          当异步任务执行完成后，系统会调用此方法。           |   此方法运行在主线程中，可以修改控件状态，例如: 下载完成。   |
|  publishProgress  |                  在doInBackground中使用。                   |                用于触发onProgressUpdate方法。                |

注：onPreExecute()、onProgressUpdate(Integer... values)、
onPostExecute(String s)这三个方法都是运行在主线程的，因此可以在里面进行UI更新。

下面看一个 倒计时 Demo：

    public class MainActivity extends AppCompatActivity {
    
    private EditText et;
    private TextView tv;
    private Button bt;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    
        et = findViewById(R.id.et);
        tv = findViewById(R.id.tv);
        bt = findViewById(R.id.bt);
    
        bt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int time = 0;
                try {
                    time = Integer.parseInt(et.getText().toString());
                }catch (Exception e){
                    Toast.makeText(MainActivity.this,"请输入正确的数字",Toast.LENGTH_SHORT).show();
                    return;
                }
                new MyTask().execute(time);
            }
        });
    }
    
    class MyTask extends AsyncTask<Integer,Integer,String>{
    
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
        }
    
        @Override
        protected String doInBackground(Integer... integers) {
            for (int i = integers[0]; i > 0; i--){
                try {
                    publishProgress(i);
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            return "计时结束";
        }
    
        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
            tv.setText(values[0]+"");
        }
    
        @Override
        protected void onPostExecute(String s) {
            super.onPostExecute(s);
            tv.setText(s);
        }
    }
    }
new MyTask().execute(time);

最终会把time作为参数传给doInBackground(Integer... integers)
doInBackground方法的返回值会作为参数传给onPostExecute。

ok，再使用AsyncTask做一个进度条：

    public class SecondActivity extends AppCompatActivity implements View.OnClickListener {
    
    private Button btn1,btn2;
    private ProgressBar pro1,pro2;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
    
        btn1 = findViewById(R.id.bt1);
        btn2 = findViewById(R.id.bt2);
        pro1 = findViewById(R.id.pr1);
        pro2 = findViewById(R.id.pr2);
        btn1 .setOnClickListener(this);
        btn2 .setOnClickListener(this);
    }
    
    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.bt1:
                DownloadTask d1= new DownloadTask();
                d1.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,1);
                btn1.setText("正在下载");
                btn1.setEnabled(false);
                break;
            case R.id.bt2:
                DownloadTask d2= new DownloadTask();
                d2.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,2);
                btn2.setText("正在下载");
                btn2.setEnabled(false);
                break;
            default:
                break;
        }
    }
    
    private class DownloadTask extends AsyncTask<Integer,Integer,Integer> {
        @Override
        protected Integer doInBackground(Integer... integers) {
            for (int i = 1; i <= 10; i++){
                try {
                    Thread.sleep(1000);
                    publishProgress(i * 10,integers[0]);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            return integers[0];
        }
    
        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
            switch (values[1]){
                case 1:
                    pro1.setProgress(values[0]);
                    break;
                case 2:
                    pro2.setProgress(values[0]);
                    break;
                default:
                    break;
            }
        }
    
        @Override
        protected void onPostExecute(Integer integer) {
            super.onPostExecute(integer);
            switch (integer){
                case 1:
                    btn1.setText("下载完成");
                    btn1.setEnabled(false);
                    break;
                case 2:
                    btn2.setText("下载完成");
                    btn2.setEnabled(false);
                    break;
                default:
                    break;
            }
        }
    }
    }


# 五、AsyncTask和Handle的简单对比
## 1、Handler和AsyncTask的执行过程

![Handler和AsyncTask的执行过程](https://upload-images.jianshu.io/upload_images/9000209-0a254cab48fd258b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

和Handle相比，AsyncTask的执行过程更加简单。
## 2、AsyncTask和Handle的优缺点
AsyncTask使用的优点是简单,快捷,过程可控。使用的缺点是在使用多个异步操作和并需要进行UI变更时,就会变得复杂起来。
Handler使用的优点是结构清晰，功能定义明确,对于多个后台任务时，简单，清晰。使用的缺点是在单个后台异步处理时，显得代码过多，结构过于复杂。
所以在数据简单时使用AsyncTask，数据量多且复杂就使用handler。

#六、线程池
###关于线程池
AsyncTask 对应的线程池 ThreadPoolExecutor 都是进程范围内共享的，且都是 static 的，所以是 Asynctask 控制着进程范围内所有的子类实例。由于这个限制的存在，当使用默认线程池时，如果线程数超过线程池的最大容量，线程池就会爆掉(3.0 后默认串行执行，不会出现个问题)。针对这种情况，可以尝试自定义线程池，配合 Asynctask 使用。

###关于默认线程池
AsyncTask里面线程池是一个核心线程数为CPU + 1，最大线程数为CPU * 2 + 1， 工作队列长度为 128 的线程池，线程等待队列的最大等待数为 28，但是可以自定义线程池。线程池是由 AsyncTask 来处理的，线程池允许 tasks 并行运行，需要注意的是并发情况下数据的一致性问题，新数据可能会被老数据覆盖掉。所以希望 tasks 能够串行运行的话，使用 SERIAL_EXECUTOR。

#七、AsyncTask在不同的SDK版本中的区别
调用 AsyncTask 的 execute 方法不能立即执行程序的原因及改善方案通过查阅官方文档发现，AsyncTask 首次引入时，异步任务是在一个独立的线程中顺序的执 行，也就是说一次只执行一个任务，不能并行的执行，从 1.6 开始，AsyncTask 引入了线程池，支持同时执行 5 个异步任务，也就是说只能有 5 个线程运行，超过的线程只能等待，等待前面的线程直到某个执行完了才被调度和运行。换句话说， 如果进程中的 AsyncTask 实例个数超过 5 个，那么假如前 5 都运行很长时间的话，那么第 6 个只能等待机会了。这是 AsyncTask 的一个限制，而且对于 2.3 以前的版本无法解决。如果你的应用需要大量的后台线程去执行任务，那么只能放弃使用 AsyncTask，自己创建线程池来管理 Thread。不得不说，虽然 AsyncTask 较 Thread 使用起来方便，但是它最多只能同时运行 5 个线程，这也大大局限了它的作用，你必须要小心设计你的应用，错开使用 AsyncTask 时间，尽力做到分时，或者保证数量不会大于 5 个，否在就会遇到上面提到的问题。可能是 Google 意识到了 AsynTask 的局限性了，从 Android 3.0 开始对 AsyncTask 的 API 做出了一些调整：每次只启动一个线程执行一个任务，完了之后再执行第二个任务， 也就是相当于只有一个后台线程在执行所提交的任务。

#八、一些问题
##1）AsyncTask不与任何组件绑定生命周期
很多开发者会认为一个在 Activity 中创建的 AsyncTask 会随着 Activity 的销毁而销毁。然而事实并非如此。AsynTask 会一直执行，直到 doInBackground()方法执行完毕，然后，如果 cancel(boolean)被调用,那么 onCancelled(Result result) 方法会被执行；否则，执行 onPostExecute(Result result)方法。如果我们的 Activity 销毁之前，没有取消 AsyncTask，这有可能让我们的应用崩溃(crash)。因为它想要处理的 view 已经不存在了。所以，我们是必须确保在销毁活动之前取消任务。 总之，我们使用 AsyncTask 需要确保 AsyncTask 正确的取消。

##2）串行或者并行的执行异步任务
在 Android1.6 之前的版本，AsyncTask 是串行的，在 1.6 之后的版本，采用线程池处理并行任务，但是从 Android 3.0 开始，为了避免 AsyncTask 所带来的并发错误，又采用一个线程来串行执行任务。可以使用 executeOnExecutor()方法来并行地执行任务。

串行：Execute()，使用execute()方法启动的进度条一个结束后才能启动另一个进度条
并行：ExecuteOnExecuter()，同一时间只能启动五个线程，使用executeOnExecuter()启动的进度条同时下载
可以在第二个demo中，取消注释观察区别。

##3）内存泄漏
如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对创建了AsyncTask的Activity的引用。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将继续在内存里保留这个引用，导致Activity无法被回收，引起内存泄露。

##4）结果丢失
屏幕旋转或Activity在后台被系统杀掉等情况会导致Activity的重新创建，之前运行的AsyncTask会持有一个之前Activity的引用，这个引用已经无效，这时调用onPostExecute()再去更新界面将不再生效。

#九、AsyncTask原理
* AsyncTask 中有两个线程池（ SerialExecutor 和 THREAD_POOL_EXECUTOR）和一个 Handler（InternalHandler），其中线程池 SerialExecutor 用于任务 的排队， 而线程池 THREAD_POOL_EXECUTOR 用于真正地执行任务，InternalHandler 用于将执行环境从线程池切换到主线程。 
* sHandler 是一个静态的 Handler 对象，为了能够将执行环境切换到主线程，这就要求 sHandler 这个对象必须在主线程创建。由于静态成员会在加载类的时候进行初始化，因此这就变相要求 AsyncTask 的类必须在主线程中加载，否则同一个进程中的 AsyncTask 都将无法正常工作。

参考：
1、https://blog.csdn.net/yixingling/article/details/79517048，demo和带水印图片均来源于此，在他的demo上，做了一些自己的改动和思考。
2、https://blog.csdn.net/qq_37321098/article/details/81625580