# Android App 控制休眠唤醒的Demo

在 Android5.1 ~Android11 上验证可以使用

项目github 地址 [Sleep/Wake](https://github.com/qy-tech/ScreenOnAndOff) 

编译好的 APK 下载 [ScreenOnAndOff.apk](https://github.com/qy-tech/ScreenOnAndOff/releases/download/v0.0.1/ScreenOnAndOff.apk) 

核心代码

```java
mPowerManager = (PowerManager) getSystemService(POWER_SERVICE);

policyManager = (DevicePolicyManager) getSystemService(Context.DEVICE_POLICY_SERVICE);

private void wake() {
  // turn on screen
  Log.d("qytech", "ON!");
  mWakeLock = mPowerManager.newWakeLock(PowerManager.SCREEN_BRIGHT_WAKE_LOCK | PowerManager.ACQUIRE_CAUSES_WAKEUP, getClass().getSimpleName());
  mWakeLock.acquire(1000L);
  mWakeLock.release();
}

private void sleep(){
  policyManager.lockNow();
}
```

