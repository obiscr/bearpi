# BearPi SSD1306 Player


## 效果演示

[![视频演示](https://user-images.githubusercontent.com/28687074/153735317-9342941a-45bf-4d5e-b335-d5dba797524d.png)](https://www.bilibili.com/video/BV1pP4y1b7iM/)

## 环境准备

+ 0x01 前往 B站 https://www.bilibili.com/video/BV1tv411b7SA 观看P2准备环境

+ 0x02 准备一块 BearPi-HM_Nano 开发板

+ 0x03 准备一块SSD1306 OLED显示屏



## 实现过程

### 项目本身DEMO

首先需要下载 HarmonyOS 源码，前往 [小熊派开源社区 / BearPi-HM_Nano](https://gitee.com/bearpi/bearpi-hm_nano.git) 下载源 码（这部分上面提到视频P2也有）

**然后进入源码主目录**，下载鸿蒙OS的SSD1306 OLED显示屏驱动库，前往 [chhihopeorg / harmonyos-ssd1306](https://gitee.com/hihopeorg/harmonyos-ssd1306?_from=gitee_search) 下载



源码下载完成之后需要根据 [README.md](https://gitee.com/hihopeorg/harmonyos-ssd1306/blob/master/README.md) 做一些适当的配置以及安装依赖库。

然后编译代码，编译完成之后烧录到开发版，按下开发板的复位（RESET）按键即可看到动画播放。



### 播放GIF

播放GIF其实就是把一个gif动图分成每一张图片，类似于定格动画。得到了每一帧的图之后，需要把这些帧图转化为像素数组，然后显示像素数组即可。



首先假定我们现在有一张gif图片，叫做**harmony.gif**，然后通过命令把gif转化成图片

```python
 python.exe .\gif2imgs.py .\harmony.gif .\harmony_gif_out\
```

然后进入 harmony_gif_out 目录，通过java代码生成转化脚本，并且需要把 **img2code.py** 也复制到harmony_gif_out目录

假设 harmony_gif_out 目录一共生成了59张图片，则 `GenerateCode.java` 中的 imgSize 变量的值修改成59，然后修改main方法代码 如下：

```java
public static void main(String[] args) throws IOException {
    // 1. 输出转化多个img的Python脚本
    outPyScript();
    // 2. 输出 void 方法
    //        generateMethod();
    // 3. 输出方法调用名字
    //        outCallName();
}
```

执行

```bash
$ javac GenerateCode.java
$ java GenerateCode
```



然后在 harmony_gif_out 目录打开cmd窗口，把 生成的内容<sup>[1]</sup> 粘贴进去。然后会开始生成。最终每个文件都会生成一个名字与其相同，但是后缀是.c结尾的文件。

> [1]生成的内容 大概如下：（有多少张图就会有多少行脚本）

```python
python.exe .\img2code.py .\frame0.png . 128 64
python.exe .\img2code.py .\frame1.png . 128 64
python.exe .\img2code.py .\frame2.png . 128 64
...
```



然后修改java脚本如下，取消 **generateMethod();** 一行的注释

```java
public static void main(String[] args) throws IOException {
    // 1. 输出转化多个img的Python脚本
    // outPyScript();
    // 2. 输出 void 方法
    generateMethod();
    // 3. 输出方法调用名字
    //        outCallName();
}
```

重新编译并执行 java 脚本，把生成的内容粘贴到 oled屏幕驱动目录下 **ssd1306_demo.c** 文件的 **void Ssd1306TestTask(void* arg)** 方法的前面



再次修改脚本代码，改为如下

```java
public static void main(String[] args) throws IOException {
    // 1. 输出转化多个img的Python脚本
    // outPyScript();
    // 2. 输出 void 方法
    // generateMethod();
    // 3. 输出方法调用名字
    outCallName();
}
```

取消 **outCallName();** 一行的注释

重新编译并执行 java 脚本，把生成的内容粘贴到 oled屏幕驱动目录下 **ssd1306_demo.c**  的 Ssd1306TestTask 方法的while 循环 里面。



然后重新编译并且并烧录，就会看到gif的内容。
