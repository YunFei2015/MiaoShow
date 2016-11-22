Face++由三位85后的清华师兄弟于2011年创建，先后获得联想之星的天使投资和创新工场的A轮投资。2013年底开始，先后在世界最权威的人脸检测(FDDB评测)、人脸关键点定位（300-W评测），和人脸识别(LFW评测)获得三个世界第一。这意味着在人脸技术三个最核心的技术模块，Face++都达到了世界最高水平。同时Face++所提供的技术云平台(Face++ 最好的免费人脸识别云服务)也是世界上最大的人脸技术平台，累计处理图片总数接近10亿规模，并为阿里，联想，360，世纪佳缘，美图秀秀，camera360等一批国内外著名互联网企业提供了技术服务。

在今年6月份的WWDC主题演讲大会上，苹果介绍了一项全新的人脸识别功能。这项功能被添加在 iOS 10 的照片应用当中，会根据面部表情对照片进行分类。苹果软件工程高级副总裁克雷格·费德里奇在演讲中表示，“先进计算机视觉”（Advanced Computer Vision）是今年照片应用最大幅度的一次更新，因为苹果通过深度学习技术将人脸识别带到了 iPhone 身上。

![iphone photos](/Users/yunfei/Documents/Work/公众号文章/2016.11.07人脸搜索/IMG_0093.JPG)

本文将介绍如何通过Face++来实现人脸搜索。

![search demo](/Users/yunfei/Documents/Work/公众号文章/2016.11.07人脸搜索/搜索示例.gif)

###一、准备工作

1\. 请到GitHub上下载SDK：[facepp-ios-sdk](https://github.com/FacePlusPlus/facepp-ios-sdk)

2\. 解压后可以看到两个目录：FaceppSDK和FaceppSDK\_ARC。顾名思义，FaceppSDK\_ARC是在ARC环境下使用的，FaceppSDK是在MRC环境下使用的，本文着重介绍ARC环境的实现方法。

3\. 在Finder中，将FaceppSDK_ARC目录拖入工程目录下，将FaceppSDK_ARC添加至您的工程中;

4\. 在应用程序入口处添加以下代码：

* `#import "FaceppAPI.h"`
* 在入口代码中添加如下代码:
	
```
#import "FaceppAPI.h"
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	[FaceppAPI initWithApiKey: @"YOUR_KEY" andApiSecret: @"YOUR_SECRET"];
	...
}
```

5\. 使用调试模式：

开启调试模式之后，程序将向控制台输出所有http请求的url，以及返回json的原始数据，以供调试使用。

`[FaceppAPI setDebugMode: TRUE];`

###二、开始工作

####1\. 使用`/detection/detect`接口来对图片进行检测，在获得数据以后其将返回一个FaceppResult结构

```
FaceppResult *result = [[FaceppAPI detection] detectWithURL:nil 
                        imageData:[NSData dataWithContentsOfFile:@"LOCAL_FILE_PATH"]];
```

####2\. 从FaceppResult中获取结果

获得第一张脸的face_id：

	NSString *face_id = [result content][@"face"][0][@"face_id"];
####3\. 对所有图片执行1~2步后，把每个图片对应的face_id存入数组，然后使用`/faceset/create`接口来创建faceset

```
FaceppResult *setResult = [[FaceppAPI faceset] createWithFacesetName:nil andFaceId:_facesIds andTag:@"1"];
NSString *faceset_id = [setResult content][@"faceset_id"];
```
####4\. 针对search功能对一个faceset进行训练

	[[FaceppAPI train] trainAsynchronouslyWithId:_faceset_id orName:nil andType:FaceppTrainSearch];
在一个faceset内进行search之前，必须先对该faceset进行Train。

####5\. train成功后，针对某张图片的face_id进行搜索

	FaceppResult *searchResult = [[FaceppAPI recognition] searchWithKeyFaceId:face_id andFacesetId:_faceset_id orFacesetName:nil];
搜索到的相似图片的face_id都存储在`[searchResult content][@"candidate"]`里，以下为运行结果：

```        
 [FacePlusPlus]response JSON: 
{
    "candidate": [
        {
            "face_id": "589bb6f9c7cd98a45c662a93f4607316", 
            "similarity": 99.9909, 
            "tag": ""
        }, 
        {
            "face_id": "4aa39ac1c989ff1e597dafa8979446be", 
            "similarity": 75.3242, 
            "tag": ""
        }, 
        {
            "face_id": "6ae11866cf062abb1908087b7467a03a", 
            "similarity": 59.9056, 
            "tag": ""
        }
    ], 
    "session_id": "e37308bb0e354c75b87fdf1de9b7e5c7"
}
```
