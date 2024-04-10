## 前置知识

AIDL，Binder



IBinder mToken

client -> server
ActivityClient -> ActivityClientController

ActivityRecord

ActivityTaskSupervisor


Q:Activity调用finish()后怎么走到onDestroy的？

我们知道，在 finish() 以后，Activity 的生命周期如下变化：
当前 Activity ：pause                                      onStop --> onDestroy
下一个 Activity ：    --> create --> start --> resume -->

因此为了更系统完整的理解上面一个问题，我们拆解为以下几个小问题：
MyQ：调用 Activity.finish() 后怎么执行到 Activity.onPause() 的？

# 一、源码跟踪

以

1、Activity.finish()

```java
    public void finish() {
        finish(DONT_FINISH_TASK_WITH_ACTIVITY);
    }

    private void finish(int finishTask) {
        if (mParent == null) {
            int resultCode;
            Intent resultData;
            synchronized (this) {
                resultCode = mResultCode;
                resultData = mResultData;
            }
            if (false) Log.v(TAG, "Finishing self: token=" + mToken);
            if (resultData != null) {
                resultData.prepareToLeaveProcess(this);
            }
          // 注释1：
            if (ActivityClient.getInstance().finishActivity(mToken, resultCode, resultData,
                    finishTask)) {
                mFinished = true;
            }
        } else {
            mParent.finishFromChild(this);
        }
...
    }

2、ActivityClient
      public boolean finishActivity(IBinder token, int resultCode, Intent resultData,
            int finishTask) {
        try {
            return getActivityClientController().finishActivity(token, resultCode, resultData,
                    finishTask);
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }

    private static IActivityClientController getActivityClientController() {
        final IActivityClientController controller = INTERFACE_SINGLETON.mKnownInstance;
        return controller != null ? controller : INTERFACE_SINGLETON.get();
    }

3、

```

2、





1、不得不提的 mToken

1、onPause 的兜底策略
为了确保整个流程不会被应用侧阻塞、保证下一个页面能及时响应，系统做了一个兜底 ———— 500 ms TimeOut 处理，
如果超过500ms应用侧还没有处理完onPause，那么不再等待，直接进入下一步 ———— stop流程。

Q：Binder 异步、同步调用方式

PauseActivityItem





分析方法：
1、debug
（1）对于不确定如何执行的判断分支，比如 if ，通过debug看执行路径



单词