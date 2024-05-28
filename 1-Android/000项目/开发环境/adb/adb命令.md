## 放慢系统整体动画速度

adb shell settings put global animator_duration_scale 1

adb shell settings put global animator_duration_scale 5

数字越大，动画越慢。



### 抓取日志

adb logcat > log.txt(这个可以自己命名)

### 导出文件

adb pull sdcard/crash

### adb 卸载应用

adb uninstall 包名

或者：

C:\Users\840>adb shell
salvator:/ # pm uninstall 包名


## 检查（支付宝引擎）App 是否有（悬浮窗）权限 

adb shell dumpsys package com.alipay.arome.app | findstr SYSTEM_ALERT_WINDOW 

### 如果没有，加上授权

 adb shell pm grant com.alipay.arome.app android.permission.SYSTEM_ALERT_WINDOW 

### 给指定用户添加权限

adb shell pm grant --user 10 com.alipay.arome.app android.permission.SYSTEM_ALERT_WINDOW

