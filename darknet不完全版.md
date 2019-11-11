# 从零开始自定义训练yolov3
## 前言
相信各位在经历打开摄像头成功识别物品的喜悦后，肯定会产生一个想法，我能不能也自定义一个自己的目标检测(o´ω`o)ノ。答案是肯定。接下来我来一步一步展示我的思路和方法给你们，让你们也能有一个自己的目标检测。
##1.思路
1. 网络上有voc数据集训练教程，那我能不能自定义一个voc数据集？⊙ω⊙
2. 通过分析pascal voc数据集好像也可以voc数据集的自定义，这事好像就成了⊙▽⊙

## 2.自定义voc数据集

1. 先在darknet下创建一个新文件夹命名为setsuna(看个人爱好，这个是我的命名)
2. 然后在setsuna文件夹里创建分别为annotations、ImageSets、JPEGImages的文件夹，并在ImageSets下新建main文件夹。
![image](https://raw.githubusercontent.com/prometheus-code/darenetcode/master/mmexport1573434340687.jpg)
3. 先在网上下载标记工具labelimg，并将** 网上搜集照片 ** 放在setsuna目录下的JPEGImages文件夹中，并用name.py将它们批量重命名如000001，000002等依次递增的文件名。 

以下是name.py的代码
```
import os
path = "C:setsuna/JPEGImages"#这里是你自己的图片文件夹的目录
filelist = os.listdir(path) #该文件夹下所有的文件（包括文件夹）
count=0
for file in filelist:
    print(file)
for file in filelist:   #遍历所有文件
    Olddir=os.path.join(path,file)   #原来的文件路径
    if os.path.isdir(Olddir):   #如果是文件夹则跳过
        continue
    filename=os.path.splitext(file)[0]   #文件名
    filetype=os.path.splitext(file)[1]   #文件扩展名
    Newdir=os.path.join(path,str(count).zfill(6)+filetype)  #用字符串函数zfill 以0补全所需位数
    os.rename(Olddir,Newdir)#重命名
    count+=1
```
&emsp;

这里是已经编译好的labelimg文件，解压即可使用
```
下载地址：链接: https://pan.baidu.com/s/1kwwO5VxLMpAuKFvckPpHyg    提取码: 2557
```
labelimg界面是这个样子
![image](https://raw.githubusercontent.com/prometheus-code/darenetcode/master/20181229151217544.png)

&emsp;
标注图片

- 在date文件夹下txt文件中用记事本应用程序添加你所想要识别的目标名字

  ![image]( https://raw.githubusercontent.com/prometheus-code/darenetcode/master/GIF.gif )

- 在open_dir处打开images所在的文件夹，在change save dir打开annotations 文件夹。

- 使用“Create RectBox”开始画框，单击结束画框，再双击选择类别。完成一张图片后点击“Save”保存，此时XML文件已经保存到本地了。点击“Next Image”转到下一张图片

- 动态图

  ![image]( https://raw.githubusercontent.com/prometheus-code/darenetcode/master/GIF123.gif )

## 3.转换voc数据集



