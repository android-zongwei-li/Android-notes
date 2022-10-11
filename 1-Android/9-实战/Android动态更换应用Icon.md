#一、动态更换应用icon步骤
1、在AndroidManifiest.xml中添加：<activity-alias>标签。
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.lzw.life">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity-alias
            android:name=".TestIcon1"
            android:enabled="false"
            android:label="icon1"
            android:icon="@mipmap/safari_black"
            android:targetActivity=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity-alias>

        <activity-alias
            android:name=".TestIcon2"
            android:enabled="false"
            android:label="icon2"
            android:icon="@mipmap/safari_silver"
            android:targetActivity=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity-alias>
    </application>

</manifest>
```

2、Java代码控制更换icon
```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mDefault = getComponentName();
        mIcon1 = new ComponentName(getBaseContext(),"com.lzw.life.TestIcon1");
        mIcon2 = new ComponentName(getBaseContext(),"com.lzw.life.TestIcon2");
        mPm = getApplicationContext().getPackageManager();
    }

    private ComponentName mDefault;
    private ComponentName mIcon1;
    private ComponentName mIcon2;
    private PackageManager mPm;

    public void changeIcon1(View view){
        disableComponent(mDefault);
        disableComponent(mIcon2);
        enableComponent(mIcon1);
    }

    public void changeIcon2(View view){
        disableComponent(mDefault);
        disableComponent(mIcon1);
        enableComponent(mIcon2);
    }

    private void enableComponent(ComponentName componentName){
        mPm.setComponentEnabledSetting(componentName,
                PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
                PackageManager.DONT_KILL_APP);
    }

    private void disableComponent(ComponentName componentName){
        mPm.setComponentEnabledSetting(componentName,
                PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
                PackageManager.DONT_KILL_APP);
    }
}
```

#二、动态换icon小结
这种方式可以实现动态更换icon的基本需求但还存在一些问题：
1、触发更换后，App会退出
这种情况可以考虑在App退出的时候去触发更换，下次进入的时候就可以看到全新的icon了
2、这种实现，在使用AS去Run安装时会报错：
```
Error while executing: am start -n "com.lzw.life/com.lzw.life.MainActivity" -a android.intent.action.MAIN -c android.intent.category.LAUNCHER
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.lzw.life/.MainActivity }
Error type 3
Error: Activity class {com.lzw.life/com.lzw.life.MainActivity} does not exist.

Error while Launching activity
```
3、这种实现，是应用先做了配置，相对还是很死板的，期望还是能够通过服务端的数据来做动态的更换

#三、遗留课题
目前这种实现还是存在较多问题的（见二），改善空间比较大，后续方向可围绕（二）中的小结去展开。

#四、学习总结



写于：2020/11/23
参考：
1、https://mp.weixin.qq.com/s/QFUJ1tym73metDL7VzJ9YA