# 人脸检测之OpenCV、seetaface性能比较
![人脸](media/15167773619676/%E4%BA%BA%E8%84%B8.jpg)
使用OpenCV与seetaface检测同一人脸，对比起速度。
## OpenCV
![OpenCV时间](media/15167773619676/OpenCV%E6%97%B6%E9%97%B4.png)
对于720P的分辨率，识别人脸所需时间大约为0.02秒。
###关键代码
![OpenCV关键代码](media/15167773619676/OpenCV%E5%85%B3%E9%94%AE%E4%BB%A3%E7%A0%81.png)
&emsp;&emsp;使用nativeDetect对灰度进行检测，耗时较短。
## seetaface
![seetaface](media/15167773619676/WX20180124-214307@2x.png)
对比720P的分辨率，检测人脸所需时间大约为2秒。
### 关键代码
![seetaface关键代码](media/15167773619676/seetaface%E5%85%B3%E9%94%AE%E4%BB%A3%E7%A0%81.png)
![seetaface关键代码2](media/15167773619676/seetaface%E5%85%B3%E9%94%AE%E4%BB%A3%E7%A0%812.png)
&emsp;&emsp;使用face_detection.cpp中的Detect(
const seeta::ImageData &img)把YUV格式字节流转为ImageData，再进行检测，其中耗时主要fust.cppDetect(seeta::fd::ImagePyramid* img_pyramid)方法。
## 总结
&emsp;&emsp;建议使用OpenCV进行人脸检测。

