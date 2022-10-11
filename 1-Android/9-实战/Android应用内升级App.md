#一、为什么需要应用内升级？
1、apk上架市场周期慢，无法回退
2、可以小规模实验以及试错（新功能实验，稳定性检测）
3、可以快速收敛版本（新功能覆盖、严重bug修复）



#二、在app中存在的几种升级形式
1、应用启动时静默检测，提示更新
2、用户手动在设置页，点击检测更新



#三、实现流程
![应用内更新App流程图](images/Android应用内升级App/1240.png)



#四、案例实现步骤
##1、网络模块设计
1）考虑通过接口隔离具体实现
好处：
（1）方便以后替换实现
（2）可以并行开发
2）使用okhttp完成接口实现，实现get请求，文件下载

##2、UI实现
1）使用DialogFragment而不是使用Dialog
2）接入网络请求，进度回调

##3、安装apk以及做一些细节处理
1）用户下载过程中cancel，如何及时的取消请求，中断下载
2）apk的完整性校验

##4、适配
**1）避免Android存储卡权限**
* 使用应用内部的cache文件夹，避免涉及到存储卡权限

**2）Android N FileProvider适配**
* 应用安装，涉及到文件uri的传递，需要进行适配

**3）Android O 对应用安装进行的权限的限制**
* 需要引入安装权限

**4）Android P 对http网络请求的约束**
* 在Android P 上，默认不允许直接使用http的请求，需要使用https





#五、具体实现
##1、搭建网络访问模块
#####1）定义接口，共三个
网络访问接口，负责发起get请求、下载文件请求、取消
```
public interface INetManager {

    /**
     * 发起请求
     *
     * @param url         地址
     * @param netCallback 处理返回的结果
     * @param tag         标识当前的请求
     */
    void get(String url, INetCallback netCallback, Object tag);

    /**
     * 下载
     *
     * @param url              资源地址
     * @param targetFile       保存到：targetFile
     * @param downloadCallback 下载结果回调
     * @param tag              标识当前的下载请求
     */
    void download(String url, File targetFile, IDownloadCallback downloadCallback, Object tag);

    /**
     * 取消数据请求
     *
     * @param tag 标识要取消的请求
     */
    void cancel(Object tag);
}
```
处理网络请求结果的接口
```
public interface INetCallback {
    /**
     * 请求成功，再此进行处理
     * @param response
     */
    void onSuccess(String response);

    /**
     * 请求失败，在此进行处理
     * @param throwable
     */
    void onFailed(Throwable throwable);
}
```
处理下载结果的接口
```
public interface IDownloadCallback {
    /**
     * 下载成功，在此处理
     * @param apkFile
     */
    void onSuccess(File apkFile);

    /**
     * 下载进度，在此处理
     * @param progress
     */
    void progress(int progress);

    /**
     * 下载失败，在此处理
     * @param throwable
     */
    void onFailure(Throwable throwable);
}

```
#####2)接口实现类：
接口定义好了，自然就是实现了，这里使用Okhttp来完成网络的访问。
待会在业务代码：AppUpdater中就可以看到接口隔离实现的好处之一：
可以很方便的替换具体实现。当不想用Okhttp的时候，可以便捷的修改为其他网络访问框架。
```
public class OkHttpNetManager implements INetManager {
    private static final String TAG = "OkHttpNetManager";

    private static OkHttpClient sOkHttpClient;

    private static Handler sHandler = new Handler(Looper.getMainLooper());

    static {
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        builder.connectTimeout(15, TimeUnit.SECONDS);
        sOkHttpClient = builder.build();
    }

    @Override
    public void get(String url, final INetCallback netCallback, Object tag) {
        Request.Builder builder = new Request.Builder();
        Request request = builder.url(url).get().tag(tag).build();

        Call call = sOkHttpClient.newCall(request);

        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NotNull Call call, @NotNull final IOException e) {
                sHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        netCallback.onFailed(e);
                    }
                });
            }

            @Override
            public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {
                try {
                    final String string = response.body().string();
                    sHandler.post(new Runnable() {
                        @Override
                        public void run() {
                            netCallback.onSuccess(string);
                        }
                    });
                } catch (final IOException e) {
                    e.printStackTrace();
                    sHandler.post(new Runnable() {
                        @Override
                        public void run() {
                            netCallback.onFailed(e);
                        }
                    });
                }
            }
        });
    }

    @Override
    public void download(String url, final File targetFile, final IDownloadCallback downloadCallback, Object tag) {
        if (!targetFile.exists()) {
            targetFile.getParentFile().mkdirs();
        }

        //发起请求
        Request.Builder builder = new Request.Builder();
        final Request request = builder.url(url).get().tag(tag).build();
        Call call = sOkHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NotNull Call call, @NotNull final IOException e) {
                sHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        downloadCallback.onFailure(e);
                    }
                });
            }

            @Override
            public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {
                InputStream is = null;
                OutputStream os = null;

                try {
                    final long totalLen = response.body().contentLength();

                    is = response.body().byteStream();
                    os = new FileOutputStream(targetFile);

                    byte[] buffer = new byte[8 * 1024];
                    int bufferLen;
                    int curLen = 0;
                    while (!call.isCanceled() && (bufferLen = is.read(buffer)) != -1) {
                        os.write(buffer, 0, bufferLen);
                        os.flush();
                        curLen += bufferLen;

                        final int finalCurLen = curLen;
                        sHandler.post(new Runnable() {
                            @Override
                            public void run() {
                                downloadCallback.progress((int) (finalCurLen * 1.0f / totalLen * 100));
                            }
                        });
                    }

                    if (call.isCanceled()){
                        return;
                    }

                    sHandler.post(new Runnable() {
                        @Override
                        public void run() {
                            downloadCallback.onSuccess(targetFile);
                        }
                    });

                } catch (final FileNotFoundException e) {
                    if (call.isCanceled()){
                        return;
                    }
                    e.printStackTrace();
                    sHandler.post(new Runnable() {
                        @Override
                        public void run() {
                            downloadCallback.onFailure(e);
                        }
                    });
                } finally {
                    if (is != null) {
                        is.close();
                    }
                    if (os != null) {
                        os.close();
                    }
                }
            }
        });
    }

    @Override
    public void cancel(Object tag) {
        List<Call> queuedCalls = sOkHttpClient.dispatcher().queuedCalls();
        if (queuedCalls != null) {
            for (Call call : queuedCalls) {
                if (tag.equals(call.request().tag())) {
                    Log.d("cancel", "find call = " + tag);
                    call.cancel();
                }
            }
        }

        List<Call> runningCalls = sOkHttpClient.dispatcher().runningCalls();
        if (runningCalls != null) {
            for (Call call : runningCalls) {
                if (tag.equals(call.request().tag())) {
                    Log.d("cancel", "find call = " + tag);
                    call.cancel();
                }
            }
        }
    }
}
```

###2、AppUpdater类，为应用提供App更新的接口
默认使用OkHttpNetManager()实现类，当要换其他网络访问框架时，使用setINetManager更新即可。
```
public class AppUpdater {

    private static AppUpdater sInstance = new AppUpdater();

    public static AppUpdater getInstance() {
        return sInstance;
    }

    /**
     * 默认的网络访问方式：OkHttpNetManager
     */
    private static INetManager sINetManager = new OkHttpNetManager();

    public INetManager getINetManager() {
        return sINetManager;
    }

    /**
     * 指定网络访问方式
     *
     * @param netManager
     */
    public void setINetManager(INetManager netManager) {
        sINetManager = netManager;
    }

```

###3、使用网络模块请求数据并更新UI
1）发起获取新版本信息的请求，并根据结果做具体处理
```
btnCheckVersion.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                AppUpdater.getInstance().getINetManager().get(Constants.Url.appUpdaterJsonUrl, new INetCallback() {
                    @Override
                    public void onSuccess(String response) {
                        //TODO 分析结果，看是否要更新

                        //1、解析json
                        //2、做版本适配
                        //如果需要更新
                        //3、弹窗
                        //4、点击下载

                        AppVersionInfoBean appVersionInfoBean = AppVersionInfoBean.parse(response);

                        if (appVersionInfoBean == null){
                            Toast.makeText(SettingActivity.this, "版本检测接口返回数据异常", Toast.LENGTH_SHORT).show();
                            return;
                        }

                        // TODO 检测是否需要更新
                        try {
                            long versionCode = Long.parseLong(appVersionInfoBean.getVersionCode());
                            if (versionCode <= AppUtils.getVersionCode(SettingActivity.this)){
                                Toast.makeText(SettingActivity.this, "已经是最新版本，无需更新", Toast.LENGTH_SHORT).show();
                                return;
                            }
                        } catch (NumberFormatException e) {
                            e.printStackTrace();
                            Toast.makeText(SettingActivity.this, "版本检测接口返回版本号异常", Toast.LENGTH_SHORT).show();
                            return;
                        }

                        // TODO 弹出更新窗口
                        UpdateVersionShowDialog.show(SettingActivity.this,appVersionInfoBean);
                    }

                    @Override
                    public void onFailed(Throwable throwable) {
                        throwable.printStackTrace();
                        Toast.makeText(SettingActivity.this, "版本更新接口请求失败", Toast.LENGTH_SHORT).show();
                    }
                },SettingActivity.this);
            }
        });
```
2）上面有一个AppVersionInfoBean类
我们把获取到的版本信息解析、封装成一个Bean类，用于版本验证和UI更新的数据来源。

这里有一个解析的小技巧：
把解析代码放到Bean类中。
```
public class AppVersionInfoBean implements Serializable {

    private String title;
    private String content;
    private String url;
    private String md5;
    private String versionCode;

    private AppVersionInfoBean(String title, String content, String url, String md5, String versionCode) {
        this.title = title;
        this.content = content;
        this.url = url;
        this.md5 = md5;
        this.versionCode = versionCode;
    }

    /**
     * 把response转换为AppVersionInfoBean。
     *
     * @param response
     * @return
     */
    public static AppVersionInfoBean parse(String response) {
        try {
            JSONObject responseJson = new JSONObject(response);
            String title = responseJson.optString("title");
            String content = responseJson.optString("content");
            String url = responseJson.optString("url");
            String md5 = responseJson.optString("md5");
            String versionCode = responseJson.optString("versionCode");

            //TODO 是否需要对获取到的值进行检验
            // 不应该在这里检测，检测属于使用这个bean，不适合在这里处理

            return new AppVersionInfoBean(title,content,url,md5,versionCode);
        } catch (JSONException e) {
            e.printStackTrace();
        }

        return null;
    }

    public String getTitle() {
        return title;
    }

    public String getContent() {
        return content;
    }

    public String getUrl() {
        return url;
    }

    public String getMd5() {
        return md5;
    }

    public String getVersionCode() {
        return versionCode;
    }
}
```

3）UI模块以及安装apk
使用的是一个DialogFragment。
在这里发起了下载Apk的请求，并对请求结果做处理。
```
public class UpdateVersionShowDialog extends DialogFragment {
    private static final String TAG = "UpdateVersionShowDialog";

    private static final String KEY_APP_VERSION_INFO_BEAN = "app_version_info_bean";

    /**
     * 版本更新信息bean，由show方法传入
     */
    private AppVersionInfoBean appVersionInfoBean;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Bundle arguments = getArguments();
        if (arguments != null) {
            appVersionInfoBean = (AppVersionInfoBean) arguments.getSerializable(KEY_APP_VERSION_INFO_BEAN);
        }
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.dialog_update_app_version, container, false);
        bindView(view);
        return view;
    }

    private void bindView(View view) {
        TextView tvTitle = view.findViewById(R.id.tv_title);
        TextView tvContent = view.findViewById(R.id.tv_content);
        final TextView tvUpdate = view.findViewById(R.id.tv_update);

        tvTitle.setText(appVersionInfoBean.getTitle());
        tvContent.setText(appVersionInfoBean.getContent());

        tvUpdate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(final View v) {
                v.setEnabled(false);

                //安装包的下载地址,选择getCacheDir路径，可以避免存储权限的处理
                final File targetFile = new File(getActivity().getCacheDir(), "target.apk");
                AppUpdater.getInstance().getINetManager().download(appVersionInfoBean.getUrl(), targetFile, new IDownloadCallback() {
                    @Override
                    public void onSuccess(File apkFile) {
                        v.setEnabled(true);

                        dismiss();

                        //下载成功
                        Log.d(TAG, "success = " + apkFile.getAbsolutePath());

                        //TODO check MD5
                        String fileMd5 = AppUtils.getFileMd5(targetFile);
                        Log.d(TAG, "md5 = " + fileMd5);

                        if (fileMd5 != null && fileMd5.equals(appVersionInfoBean.getMd5())) {
                            //校验成功，安装
                            Toast.makeText(getActivity(), "开始安装", Toast.LENGTH_SHORT).show();

                            AppUtils.installApk(getActivity(), apkFile);
                        } else {
                            Toast.makeText(getActivity(), "md5检测失败", Toast.LENGTH_SHORT).show();
                        }
                    }

                    @Override
                    public void progress(int progress) {
                        Log.d(TAG, "progress = " + progress);

                        tvUpdate.setText(progress + "%");
                    }

                    @Override
                    public void onFailure(Throwable throwable) {
                        v.setEnabled(true);

                        throwable.printStackTrace();
                        Toast.makeText(getActivity(), "文件下载失败", Toast.LENGTH_SHORT).show();
                    }
                }, UpdateVersionShowDialog.this);
            }
        });
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        getDialog().requestWindowFeature(Window.FEATURE_NO_TITLE);
        getDialog().getWindow().setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
    }

    @Override
    public void onDismiss(@NonNull DialogInterface dialog) {
        super.onDismiss(dialog);

        Log.d("tag", "onDismiss: ");
        AppUpdater.getInstance().getINetManager().cancel(this);
    }

    public static void show(FragmentActivity fragmentActivity, AppVersionInfoBean appVersionInfoBean) {
        Bundle bundle = new Bundle();
        bundle.putSerializable(KEY_APP_VERSION_INFO_BEAN, appVersionInfoBean);

        UpdateVersionShowDialog updateVersionShowDialog = new UpdateVersionShowDialog();
        updateVersionShowDialog.setArguments(bundle);

        updateVersionShowDialog.show(fragmentActivity.getSupportFragmentManager(), "updateVersionShowDialog");
    }
}
```

4)最后，就是一个工具类 AppUtils 
```
public class AppUtils {

    /**
     * 获取当前App的版本号
     *
     * @return  版本号
     */
    public static long getVersionCode(Context context) {
        PackageManager packageManager = context.getPackageManager();
        try {
            PackageInfo packageInfo = packageManager.getPackageInfo(context.getPackageName(), 0);
            if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.P) {
                long longVersionCode = packageInfo.getLongVersionCode();
                return longVersionCode;
            }else {
                return packageInfo.versionCode;
            }
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }
        return -1;
    }

    /**
     * MD5校验
     *
     * @param targetFile    要校验md5的文件
     * @return              文件的md5
     */
    public static String getFileMd5(File targetFile) {
        if (targetFile == null || !targetFile.isFile()){
            return null;
        }

        MessageDigest digest;
        FileInputStream fis = null;
        byte[] buffer = new byte[1024];
        try {
            digest = MessageDigest.getInstance("MD5");
            fis = new FileInputStream(targetFile);
            int bufferLen;
            while ((bufferLen = fis.read(buffer)) != -1){
                digest.update(buffer,0,bufferLen);
            }
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            return null;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }finally {
            if (fis != null){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        byte[] result = digest.digest();
        BigInteger bigInteger = new BigInteger(1,result);

        return bigInteger.toString(16);
    }

    /**
     * 安装apk
     *
     * @param activity
     * @param apkFile
     */
    public static void installApk(FragmentActivity activity, File apkFile) {
        //文件有所有者概念，现在是属于当前进程的，需要把这个文件暴露给系统安装程序（其他进程）去安装
        //因此，可能会存在权限问题，需要做下面的设置
        //如果文件是sdcard上的，就不需要这个操作了
        try {
            apkFile.setExecutable(true, false);
            apkFile.setReadable(true, false);
            apkFile.setWritable(true, false);
        } catch (Exception e) {
            e.printStackTrace();
        }

        Intent intent = new Intent();
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setAction(Intent.ACTION_VIEW);
        Uri uri;

        //TODO N FileProvider
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N){
            uri = FileProvider.getUriForFile(activity, activity.getPackageName() + ".fileprovider", apkFile);
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
        }else {
            uri = Uri.fromFile(apkFile);
        }

        intent.setDataAndType(uri,"application/vnd.android.package-archive");
        activity.startActivity(intent);

        //TODO 0 INSTALL PERMISSION
        //在AndroidManifest中加入权限即可
    }
}
```

###4、适配与问题处理
1）N FileProvider
```
<!--N FileProvider适配-->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/fileproviderpath" />
        </provider>
```
xml/fileproviderpath:
```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <root-path name="root" path="."/>

    <files-path
        name="files"
        path="."/>

    <cache-path
        name="cache"
        path="."/>

    <external-path
        name="external"
        path="."/>

    <external-cache-path
        name="external_cache"
        path="."/>

    <external-files-path
        name="external_file"
        path="."/>
</paths>
```

2）O INSTALL PERMISSION
//在AndroidManifest中加入权限:    
```
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
```

3）问题记录：java.net.UnknownServiceException: CLEARTEXT communication to 59.110.162.30 not permitted by **network security policy**
解决：
(1)在res/xml中新建：network_security_config
```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>

    <base-config cleartextTrafficPermitted="true"/>
</network-security-config>
```
(2)在AndroidManifest.xml的application中：
```
android:networkSecurityConfig="@xml/network_security_config"
```

#六、总结
其实应用内更新的基本逻辑很简单，就是获取一个Apk，然后安装。

重要的是学习，如何构建整个功能模块的思路及其思考：
1、要获取Apk，需要用到网络吧？
所以得构建网络访问框架。
2、网络访问时，http/https可能会带来什么问题？如何处理呢？
3、下载apk后，存储策略是什么？是存在sdcard还是应用内部的cache？
4、如果是cache，那么要交给系统程序去安装，就涉及到文件的跨进程传递了？要如何处理？
5、O以后涉及到了安装权限问题

除了上面，我们还有如下思考：大文件，如何下载？
1、断点续下，分区间下载
原理：http，head中有range，可以指定下载一个文件的：起始字节和终止字节
实现：
如果target.apk有300字节，所以我们可以用多个线程去下载：
线程1：0，100
线程2：101，200
线程3：201，300
最后，在本地合并，使用RandomAccessFile进行seek操作。

2、使用增量更新
apk1 本地
apk2 server

apk diff apk2 --> patch

download patch 

涉及到算法  bsdiff。

![Android应用内升级该思考的问题](images/Android应用内升级App/1240-20221011113653448.png)




参考：慕课网视频

写于：
2020/09/10