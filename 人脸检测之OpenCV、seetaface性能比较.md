# 人脸检测之OpenCV、seetaface性能比较
![人脸](http://img.blog.csdn.net/20180125093705895?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
使用OpenCV与seetaface检测同一人脸，对比起速度。
## OpenCV
![OpenCV时间](http://img.blog.csdn.net/20180125093746372?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
对于720P的分辨率，识别人脸所需时间大约为0.02秒。
###关键代码
![OpenCV关键代码](http://img.blog.csdn.net/20180125093830821?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
&emsp;&emsp;使用nativeDetect对灰度进行检测，耗时较短。
## seetaface
![seetaface](http://img.blog.csdn.net/20180126091917400?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
对比720P的分辨率，检测人脸所需时间大约为2秒。
### 关键代码
![seetaface关键代码](http://img.blog.csdn.net/20180125093851244?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![seetaface关键代码2](http://img.blog.csdn.net/20180125093857020?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
&emsp;&emsp;使用face_detection.cpp中的Detect(
const seeta::ImageData &img)把YUV格式字节流转为ImageData，再进行检测，其中耗时主要fust.cppDetect(seeta::fd::ImagePyramid* img_pyramid)方法。
## 总结
&emsp;&emsp;建议使用OpenCV进行人脸检测。


