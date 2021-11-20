# Android Q 之后单独编译 Framework 

之前修改 framework 部分代码都是`mmm framework/bas -j12`编译，最近尝试修改 Android11 代码之后发现这样单独编译有问题 ，编译完之后烧录进系统直接卡在卡机动画部分起不来。

由于 Android 后面的编译都是用的 bp 文件，bp 文件配置的可以直接在项目跟目录下来直接编译到对应的模块，正确的方式应该是如下所示：

```bash
# 编译 framework 需要用framework-minus-apex,编译出来的文件还是叫framework.jar
make framework-minus-apex -j12

# 编译 services
make services -j12 

adb root && adb remount && adb disable-verity && adb reboot 

adb root  && adb remount 

adb shell ”rm -rf /system/framework/oat/“

adb push framework.jar system/framework/

adb shell sync

adb shell stop && adb shell start 
```

* 一定要删除掉机器本身的`/system/framework/oat/`不然可能会卡开机动画