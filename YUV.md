&emsp;&emsp;android中在做视频开发或相机预览时，参数只能设置为ImageFormat.NV21或者ImageFormat.YV12，设置别的参数摄像头不会打开，实现PreviewCallback接口会获取一个byte[]字节流，获取到的图像数据是YUV格式的。NV21是YUV420的一种。开发当中有时需要将YUV格式转换为RGBA格式，接下来我们简单分析YUV格式与RGBA格式。

![](http://img.blog.csdn.net/20180122220253998?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![](http://img.blog.csdn.net/20180122215739703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
## YUV
&emsp;&emsp;YUV，分为三个分量，“Y”表示明亮度（Luminance或Luma），也就是灰度值；而“U”和“V” 表示的则是色度（Chrominance或Chroma），作用是描述影像色彩及饱和度，用于指定像素的颜色。与我们熟知的RGB类似，YUV也是一种颜色编码方法，主要用于电视系统以及模拟视频领域，它将亮度信息（Y）与色彩信息（UV）分离，没有UV信息一样可以显示完整的图像，只不过是黑白的，这样的设计很好地解决了彩色电视机与黑白电视的兼容问题。并且，YUV不像RGB那样要求三个独立的视频信号同时传输，所以用YUV方式传送占用极少的频宽。
### 1.采样方式
&emsp;&emsp;YUV码流的存储格式其实与其采样的方式密切相关，主流的采样方式有三种，YUV4:4:4，YUV4:2:2，YUV4:2:0，如何根据其采样格式来从码流中还原每个像素点的YUV值，因为只有正确地还原了每个像素点的YUV值，才能通过YUV与RGB的转换公式提取出每个像素点的RGB值，然后显示出来。

用三个图来直观地表示采集的方式吧，以黑点表示采样该像素点的Y分量，以空心圆圈表示采用该像素点的UV分量。

![](http://img.blog.csdn.net/20180123212229443?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

先记住下面这段话，以后提取每个像素的YUV分量会用到。

YUV 4:4:4采样，每一个Y对应一组UV分量，每像素32位。

YUV 4:2:2采样，每两个Y共用一组UV分量，每像素16位。

YUV 4:2:0采样，每四个Y共用一组UV分量，每像素16位。

平常所讲的YUV A:B:C的意思一般是指基于4个象素来讲,其中Y采样了A次，U采样了B次,V采样了C次。
### 2. 存储方式
&emsp;&emsp;YUV420P(I420)三个分量顺序是Y、U、V。YV12三个分量的顺序是Y、V、U。NV21是按YUV 4:2:0抽样的。即对于一张图片的每一个像素，完整保留其Y分量，而对于U和V分量，以2X2的像素块比例采样。其中Y分量按平面存储，U和V则交错打包存储。NV12和NV21都是Y分量在前,U和V分量打包在后。不同处在于NV12是按UV顺序打包，而NV21是按VU顺序打包的。
#### I420
![I420](http://img.blog.csdn.net/20180123212550260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### YV12
![YV12](http://img.blog.csdn.net/20180123212327975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### NV12
![NV12](http://img.blog.csdn.net/20180123212829331?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## RGB
&emsp;&emsp;RGB 色彩模式是工业界的一种颜色标准，是通过对红(R)、绿(G)、蓝(B)三个颜色通道的变化以及它们相互之间的叠加来得到各式各样的颜色的，RGB即是代表红、绿、蓝三个通道的颜色，这个标准几乎包括了人类视力所能感知的所有颜色，是目前运用最广的颜色系统之一。ARGB 是一种色彩模式，也就是RGB色彩模式附加上Alpha（透明度）通道，常见于32位位图的存储结构。RGBA又可以称为 ARGB，是Alpha（透明度），Red，Green，Blue组成的色彩空间。但ARGB也可以指Adobe RGB色彩空间。

RGB565    每个像素用16位表示，RGB分量分别使用5位、6位、5位

RGB555    每个像素用16位表示，RGB分量都使用5位（剩下1位不用）

RGB24    每个像素用24位表示，RGB分量各使用8位

RGB32    每个像素用32位表示，RGB分量各使用8位（剩下8位不用）

ARGB32    每个像素用32位表示，RGB分量各使用8位（剩下的8位用于表示Alpha通道值）

## YUV转化RGBA
NV21转化RGBA（有待考证！！）

如图，NV21的图与此图中的uv顺序相反
![](http://img.blog.csdn.net/20180123223009492?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzYyOTkyMTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

R = Y1 + 1.370705 * ( V1 - 128 );

G = Y1 -  0.698001 * ( U1 - 128 )  - 0.703125 * (V1 - 128);

B = Y1 + 1.732446 * ( U1 - 128 );



### java实现
```js
public byte[] NV21toRGBA(byte[] data, int width, int height) {
    int size = width * height;
    byte[] bytes = new byte[size * 4];
    int y, u, v;
    int r, g, b;
    int index;
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            index = j % 2 == 0 ? j : j - 1;

            y = data[width * i + j] & 0xff;
            u = data[width * height + width * (i / 2) + index + 1] & 0xff;
            v = data[width * height + width * (i / 2) + index] & 0xff;

            r = y + (int) 1.370705f * (v - 128);
            g = y - (int) (0.698001f * (v - 128) + 0.337633f * (u - 128));
            b = y + (int) 1.732446f * (u - 128);

            r = r < 0 ? 0 : (r > 255 ? 255 : r);
            g = g < 0 ? 0 : (g > 255 ? 255 : g);
            b = b < 0 ? 0 : (b > 255 ? 255 : b);

            bytes[width * i * 4 + j * 4 + 0] = (byte) r;
            bytes[width * i * 4 + j * 4 + 1] = (byte) g;
            bytes[width * i * 4 + j * 4 + 2] = (byte) b;
            bytes[width * i * 4 + j * 4 + 3] = (byte) 255;//透明度
        }
    }
    return bytes;
}
```
### jni实现
```js
JNIEXPORT void JNICALL
Java_seetaface_JniClient_YUV2RGBA
        (JNIEnv *env, jobject obj, jbyteArray v_yuv_data,
         jbyteArray v_rgba_data, jint width, jint height) {

    jbyte *tYUVData = env->GetByteArrayElements(v_yuv_data, 0);
    jbyte *tRGBData = env->GetByteArrayElements(v_rgba_data, 0);
    int length = width * height * 4;
    jbyte yValue, vValue, uValue;
    int index;
    int r, g, b;

    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            index = j % 2 == 0 ? j : j - 1;

            yValue = tYUVData[width * i + j];
            vValue = tYUVData[width * height + width * (i / 2) + index];
            uValue = tYUVData[width * height + width * (i / 2) + index + 1];

            r = yValue + (1.370705 * (vValue - 128));
            g = yValue - (0.698001 * (vValue - 128)) - (0.337633 * (uValue - 128));
            b = yValue + (1.732446 * (uValue - 128));

            r = r < 0 ? 0 : (r > 255 ? 255 : r);
            g = g < 0 ? 0 : (g > 255 ? 255 : g);
            b = b < 0 ? 0 : (b > 255 ? 255 : b);

            tRGBData[width * i * 4 + j * 4 + 0] = r;
            tRGBData[width * i * 4 + j * 4 + 1] = g;
            tRGBData[width * i * 4 + j * 4 + 2] = b;
            tRGBData[width * i * 4 + j * 4 + 3] = 255;
        }
    }
    jbyte *buf = (jbyte *) malloc(length);
    memcpy(buf, tRGBData, length);
    env->SetByteArrayRegion(v_rgba_data, 0, length, buf);
    free(buf);
}
```
jni耗时比java要久一些，个人觉得jni代码有问题，对c语言不是很熟悉，望大家指出错误。

参考：

1.https://www.cnblogs.com/silence-hust/p/4465354.html

2.http://doc.okbase.net/raomengyang/archive/186891.html
