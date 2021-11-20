# Android 8 之后app无法接收开机广播实现自启动

客户需要在 Android8.0 之后还要通过开机广播来实现自启动 app 功能，但是在 8.0 之后整个功能被系统限制了，只能放在`/system/app/`下而且有系统签名的 app 才能获取到开机广播，并实现自启。

通过`adb shell am broadcast -a android.intent.action.BOOT_COMPLETED`手动发送开机广播测试发现，这段系统限制了app 接收广播

```bash
W/BroadcastQueue: Background execution not allowed: receiving Intent { act=android.intent.action.BOOT_COMPLETED **flg=0x400010** }
```

修改`frame/base/services/core/java/com/android/server/am/BroadcastQueue.java`就能接收到广播了。

```diff
diff --git a/services/core/java/com/android/server/am/BroadcastQueue.java b/serv
ices/core/java/com/android/server/am/BroadcastQueue.java
index 9a6f130cfd5d..306ca55172ad 100644
--- a/services/core/java/com/android/server/am/BroadcastQueue.java
+++ b/services/core/java/com/android/server/am/BroadcastQueue.java
@@ -1554,7 +1554,7 @@ public final class BroadcastQueue {
                     Slog.w(TAG, "Background execution not allowed: receiving "
                             + r.intent + " to "
                             + component.flattenToShortString());
-                    skip = true;
+                    skip = false;
                 }
             }
         }
```

修改之后发现 app 还是没办法自启动，打印 log 如下：

```bash
Background activity start [callingPackage: com.qytech.broadtest; callingUid: 10126; isCallingUidForeground: false; callingUidHasAnyVisibleWindow: false; callingUidProcState: RECEIVER; isCallingUidPersistentSystemProcess: false; realCallingUid: 10126; isRealCallingUidForeground: false; realCallingUidHasAnyVisibleWindow: false; realCallingUidProcState: RECEIVER; isRealCallingUidPersistentSystemProcess: false; originatingPendingIntent: null; isBgStartWhitelisted: false; intent: Intent { flg=0x10000000 cmp=com.qytech.broadtest/.MainActivity }; callerapp: ProcessRecord{18ae38b 1628:com.qytech.broadtest/u0a126}]
```

修改`framework/base/services/core/java/com/android/server/wm/ActivityStarter.java`就可以让后台 app 自启动了。

```diff
diff --git a/services/core/java/com/android/server/wm/ActivityStarter.java b/ser
vices/core/java/com/android/server/wm/ActivityStarter.java
index 7af237b80cfa..634f2c0eac7b 100644
--- a/services/core/java/com/android/server/wm/ActivityStarter.java
+++ b/services/core/java/com/android/server/wm/ActivityStarter.java
@@ -1412,7 +1412,7 @@ class ActivityStarter {
                     realCallingUid, realCallingUidProcState, realCallingUidHasA
nyVisibleWindow,
                     (originatingPendingIntent != null));
         }
-        return true;
+        return false;
     }
```

