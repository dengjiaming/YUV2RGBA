# OpenCV人脸识别之实践篇 
## 前言
&emsp;&emsp;前段时间对OpenCV的人脸识别进行了一些研究，在网上找到的资料，大部分都是介绍人脸检测，很少有涉及人脸识别的模块，甚至有的人连人脸检测与人脸识别的概念都没有搞清楚，而人脸识别模块大部分还是使用C++来实现的，并没有提供java接口，因此在Android上面进行人脸识别就需要多花点时间。
&emsp;&emsp;人脸检测与人脸识别是不同的，人脸检测只需要找到人脸即可，而人脸识别需要把检测出来的人脸进行对比，识别出这是谁的脸。人脸检测可以使用OpenCV里自带的分类器，而人脸识别就需要自己收集数据，自己训练分类器。自从进入3.X时代以后，OpenCV将代码库分成了两部分，分别是[稳定的核心功能库](https://sourceforge.net/projects/opencvlibrary/files/opencv-android/3.3.0/opencv-3.3.0-android-sdk.zip/download) 和[试验性质的contrib库](https://github.com/opencv/opencv_contrib) 。人脸检测的代码在前者，人脸识别的代码在后者。
&emsp;&emsp;官方给的人脸检测Demo是在Eclipse工程下实现的，而在GitHub上的这个[工程](https://github.com/jiangdongguo/OpenCV4Android)已经转为AS工程，我们只需在此基础上加上人脸识别的功能即可。
##### 先甩下一些文档
GitHub工程地址：https://github.com/jiangdongguo/OpenCV4Android
OpenCV 3.0.0 API：https://docs.opencv.org/3.0-beta/modules/refman.html
OpenCV 2.4 API：https://docs.opencv.org/2.4/modules/refman.html
OpenCV4Android SDK：https://sourceforge.net/projects/opencvlibrary/files/opencv-android/3.3.0/opencv-3.3.0-android-sdk.zip/download
contrib模块：https://github.com/opencv/opencv_contrib
本工程源码：https://github.com/keithActiv/OpenCV4Android

# 一、改为使用cmake进行ndk开发
把app文件下的build.gradle修改一下两个地方；
![这里写图片描述](http://img.blog.csdn.net/20180215190513541?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在app文件下添加CMakeLists.txt；
![这里写图片描述](http://img.blog.csdn.net/20180215191214959?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
CMakeLists.txt的代码如下：
```js
cmake_minimum_required(VERSION 3.4.1)

set(CMAKE_VERBOSE_MAKEFILE on)
set(libs "\${CMAKE_SOURCE_DIR}/src/main/libs")
include_directories(${CMAKE_SOURCE_DIR}/src/main/cpp/include)

add_library(libopencv_java3 SHARED IMPORTED )
set_target_properties(libopencv_java3 PROPERTIES
    IMPORTED_LOCATION "\${libs}/${ANDROID_ABI}/libopencv_java3.so")

set(CMAKE_CXX_FLAGS "\${CMAKE_CXX_FLAGS} -std=gnu++11 -fexceptions -frtti")

add_library(native-lib
            SHARED
            src/main/cpp/native-lib.cpp)

add_library(detection_based_tracker
            SHARED
            src/main/cpp/DetectionBasedTracker_jni.cpp)

find_library(log-lib
              log )

target_link_libraries(native-lib android log
    libopencv_java3 #used for java sdk
    ${log-lib})

target_link_libraries(detection_based_tracker android log
    libopencv_java3 #used for java sdk
    ${log-lib})
```
在opencv/OpenCV-android-sdk/sdk/native/jni目录下找到include文件下，将其和app/src/main/jni的两个文件、native-lib.cpp放在app/src/main/cpp中；
![这里写图片描述](http://img.blog.csdn.net/20180215194417988?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](http://img.blog.csdn.net/20180215194352417?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](http://img.blog.csdn.net/20180215194441393?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
现在可以删除jni、libs、obj文件下；
在OpenCV-android-sdk/sdk/native/libs目录下找到各种版本的libopencv_java3.so，将其放在app/src/main/libs中；
![这里写图片描述](http://img.blog.csdn.net/20180215193416530?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](http://img.blog.csdn.net/20180215193348285?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
把以下代码放在第一个activity中；
![这里写图片描述](http://img.blog.csdn.net/20180215195215299?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
第一步即可完成。
# 二、图片数据集
&emsp;&emsp;OpenCV教程中给出了一组图片[The AT&T Facedatabase](http://www.cl.cam.ac.uk/Research/DTG/attarchive/pub/data/att_faces.zip) ，又称ORL人脸数据库，40个人，每人10张照片。照片在不同时间、不同光照、不同表情(睁眼闭眼、笑或者不笑)、不同人脸细节(戴眼镜或者不戴眼镜)下采集。所有的图像都在一个黑暗均匀的背景下采集的，正面竖直人脸(有些有有轻微旋转)。
![这里写图片描述](http://img.blog.csdn.net/20180215200345995?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
&emsp;&emsp;可以看到每个人一个文件夹，每个文件夹下是这个人的十张照片，但是不是我们熟悉的BMP或者是PNG或者是JPEG格式的，而是PGM格式的。
&emsp;&emsp;当我们写人脸模型的训练程序的时候，我们需要读取人脸和人脸对应的标签。直接在数据库中读取显然是低效的。所以我们用csv文件读取。csv文件中包含两方面的内容，一是每一张图片的位置所在，二是每一个人脸对应的标签，就是为每一个人编号。这个at.txt就是我们需要的csv文件。生成之后它里面是这个样子的：
![这里写图片描述](http://img.blog.csdn.net/20180215202345919?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
&emsp;&emsp;这里有一个生成at.txt的快速方法，在opencv_contrib/modules/face/samples/etc中有一个create_csv.py；
![这里写图片描述](http://img.blog.csdn.net/20180215201105162?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
运行该python文件；
![这里写图片描述](http://img.blog.csdn.net/20180215202025978?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
将得到的结果复制到at.txt文件中，前面是图片的位置，后面是图片所属人脸的人的标签。
&emsp;&emsp;这里有一点值得注意：我这里保存的图像格式是\*.jpg的，而不是跟原数据集一样是*.pgm的。经测试仍然可以训练出可以正确识别我自己人脸的模型来。但是如果大小不一致会报错。
# 三、训练模型
&emsp;&emsp;OpenCV 自带了三个人脸识别算法：Eigenfaces，Fisherfaces 和局部二值模式直方图 (LBPH)。理论知识请看：[OpenCV人脸识别之理论篇](http://blog.csdn.net/qq_36299210/article/details/79331297)
&emsp;&emsp;将opencv_contrib/modules/face/include/opencv2的文件导入到app/src/main/cpp/include/opencv2中，将opencv_contrib/modules/face/src的文件导入到app/src/main/cpp中；
![这里写图片描述](http://img.blog.csdn.net/20180215203927832?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
此时，cpp文件如下：
![这里写图片描述](http://img.blog.csdn.net/20180215204908219?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
修改CMakeLists.txt;
![这里写图片描述](http://img.blog.csdn.net/2018021520500367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
接下来把图片库与at.txt放在sdracd下面，给工程添加读取文件权限；
![这里写图片描述](http://img.blog.csdn.net/20180215211244231?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
&emsp;&emsp;以eigen为举例子，其他两次模型方法一致。首先读取首先判断内存中是否已有模型文件，如果有则加载，没有则不做操作。
![这里写图片描述](http://img.blog.csdn.net/20180215215302650?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
&emsp;&emsp;现在就是at.txt派上用场的时候了，
![这里写图片描述](http://img.blog.csdn.net/20180215215516343?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
&emsp;&emsp;调用API训练模型；
![这里写图片描述](http://img.blog.csdn.net/20180215215715786?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 四、人脸识别
&emsp;&emsp;对检测到的人脸进行裁剪，并且重新设置尺寸大小；
![这里写图片描述](http://img.blog.csdn.net/20180215221418489?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
&emsp;&emsp;最后就是通过训练模型进行人脸识别：
![这里写图片描述](http://img.blog.csdn.net/20180215220238711?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
结果如下：
![这里写图片描述](http://img.blog.csdn.net/20180215223125308?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

简单说下流程：

1. 打开摄像头。

2. 加载人脸检测器，加载人脸模型，判断是否存在人脸模型，有则跳至4。

3. 训练人脸模型。

4. 人脸检测

5. 把检测到的人脸与人脸模型里面的对比，找出这是谁的脸。

6. 打印出对应编号。

# 总结
1. 同样的400百张照片,个人认为LBPH模型的准确度最高，eigen其次，fisher准确度最低。
面对编号为1的照片：
LBPH的结果是：
![这里写图片描述](http://img.blog.csdn.net/20180215225522992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
eigen的结果是：
![这里写图片描述](http://img.blog.csdn.net/20180215225545407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
fisher的结果是：
![这里写图片描述](http://img.blog.csdn.net/20180215225606537?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2. 比较三个模型的保存训练模型速度和训练速度。
![这里写图片描述](http://img.blog.csdn.net/20180215223708279?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](http://img.blog.csdn.net/20180215223725534?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](http://img.blog.csdn.net/20180215223743539?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### 训练速度：LBPH > eigen > fisher

##### 保存速度：fisher > LBPH > eigen
3. 比较三个模型的加载模型速度：
![这里写图片描述](http://img.blog.csdn.net/20180215224352530?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### 加载速度：LBPH > fisher > eigen
其中，eigen加载与保存训练都花费将近10s。

## 建议使用LBPH模型进行人脸识别。







