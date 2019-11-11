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
下载地址：链接: https://pan.baidu.com/s/1qYyZQaw#list/path=%2FLabelimg%20multi    提取码: cnn6
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
- 通过观察官方voc数据集我们发现我们还有一个准备工作没做就是ImageSets里的MAIN文件夹下的txt文件，接下来在setsuna根目录里用txtcreater.py创建它们。
- 以下是txtcreater.py的代码
~~~
import os  
import random  
 
trainval_percent = 0.5  
train_percent = 0.5  
xmlfilepath = 'Annotations'  
txtsavepath = 'ImageSets/Main'  
total_xml = os.listdir(xmlfilepath)  
 
num=len(total_xml)  
list=range(num)  
tv=int(num*trainval_percent)  
tr=int(tv*train_percent)  
trainval= random.sample(list,tv)  
train=random.sample(trainval,tr)  
 
ftrainval = open(txtsavepath+'/trainval.txt', 'w')  
ftest = open(txtsavepath+'/test.txt', 'w')  
ftrain = open(txtsavepath+'/train.txt', 'w')  
fval = open(txtsavepath+'/val.txt', 'w')  
 
for i  in list:  
    name=total_xml[i][:-4]+'\n'  
    if i in trainval:  
        ftrainval.write(name)  
        if i in train:  
            ftrain.write(name)  
        else:  
            fval.write(name)  
    else:  
        ftest.write(name)  
 
ftrainval.close()  
ftrain.close()  
fval.close()  
ftest .close()
~~~
- then，将数据集转换为darknet支持的数据格式。
- yolov3提供了将VOC数据集转为YOLO训练所需要的格式的代码，在scripts/voc_label.py文件中。这里提供一个修改版本的。在darknet文件夹下新建一个voc_label.py文件，内容如下：
~~~
 
 
def convert(size, box):
    dw = 1./(size[0])
    dh = 1./(size[1])
    x = (box[0] + box[1])/2.0 - 1
    y = (box[2] + box[3])/2.0 - 1
    w = box[1] - box[0]
    h = box[3] - box[2]
    x = x*dw
    w = w*dw
    y = y*dh
    h = h*dh
    return (x,y,w,h)
 
def convert_annotation(year, image_id):
    in_file = open('setsuna/Annotations/%s.xml'%(image_id))
    out_file = open('setsuna/labels/%s.txt'%(image_id), 'w')
    tree=ET.parse(in_file)
    root = tree.getroot()
    size = root.find('size')
    w = int(size.find('width').text)
    h = int(size.find('height').text)
 
    for obj in root.iter('object'):
        difficult = obj.find('difficult').text
        cls = obj.find('name').text
        if cls not in classes or int(difficult)==1:
            continue
        cls_id = classes.index(cls)
        xmlbox = obj.find('bndbox')
        b = (float(xmlbox.find('xmin').text), float(xmlbox.find('xmax').text), float(xmlbox.find('ymin').text), float(xmlbox.find('ymax').text))
        bb = convert((w,h), b)
        out_file.write(str(cls_id) + " " + " ".join([str(a) for a in bb]) + '\n')
 
wd = getcwd()
 
for year, image_set in sets:
    if not os.path.exists('setsuna/labels/'):
        os.makedirs('setsuna/labels/')
    image_ids = open('setsuna/ImageSets/Main/%s.txt'%(image_set)).read().strip().split()
    list_file = open('setsuna/%s_%s.txt'%(year, image_set), 'w')
    for image_id in image_ids:
        list_file.write('%s/setsuna/JPEGImages/%s.jpg\n'%(wd, image_id))
        convert_annotation(year, image_id)
    list_file.close()
~~~
- 双击点一下就完事了QVQ,以下是效果图发现多了两个文件labels和setsuna_traint.txt
![image](https://raw.githubusercontent.com/prometheus-code/darenetcode/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20191111170109.png)

## 4.修改darknet/cfg下的voc.data和yolov3-voc.cfg文件
- 复制这两个文件，并分别重命名为my_data.data和my_yolov3.cfg(取这个名字只是因为是自定义的yolov3模型QWQ)
我的my_data.data的内容为：
~~~
classes= 1 ##这个是你要识别的数目
train  = C:/darknet/setsuna/setsuna_train.txt ##你刚刚用voc_label.py创造出来的setsuna_train.txt
names = C:/darknet/setsuna/setsuna.name ##你要识别的对象名字就放name文件了
backup = C:/darknet/build/darknet/x64/backup ##后面训练出的权重文件

~~~
>注意这里的目录路径要调成与你自己的文件路径（不一定和我一样）
- my_yolov3.cfg的内容和 yolov3-voc.cfg内容相似，需要修改一下几处：
~~~
[convolutional]
size=1
stride=1
pad=1
filters=18             ## anchors_num * (classes_num + 5)[这里很关键]
activation=linear
 
[yolo]
mask = 6,7,8
anchors = 10,13,  16,30,  33,23,  30,61,  62,45,  59,119,  116,90,  156,198,  373,326
classes=1               ## 你识别的类别数目
num=9
jitter=.3
ignore_thresh = .5
truth_thresh = 1
random=0                ## multi-scale training (1 indicates using)
~~~
- **每层的yolo**之前的那个 convolutional层都要修改filters的数目，filters=anchors_num * (classes_num + 5),anchors_num为3（一般不变），classes_num为1（根据这个修改就行），修改yolo中classes的数目。注意是每个yolo和yolo前的convolutional层都做相同的修改。random为多尺度训练，1为打开多尺度训练，0为相反。**以下是my_yolov3.cfg的修改**

~~~
[net]
# Testing            ### 测试模式                                          
# batch=64           ###出错调成1 
# subdivisions=64    ###出错调成1 
# Training           ### 训练模式，每次前向的图片数目 = batch/subdivisions 
batch=64
subdivisions=16
width=416            ### 网络的输入宽、高、通道数
height=416
channels=3
momentum=0.9         ### 动量 
decay=0.0005         ### 权重衰减
angle=0
saturation = 1.5     ### 饱和度
exposure = 1.5       ### 曝光度 
hue=.1               ### 色调
learning_rate=0.001  ### 学习率 
burn_in=1000         ### 学习率控制的参数
max_batches = 50200  ### 迭代次数!!!(这里你觉得时间久你可以试着改小点，比如1000左右)                                          
policy=steps         ### 学习率策略 
steps=40000,45000    ### 学习率变动步长 
scales=.1,.1         ### 学习率变动因子
~~~
在**setsuna**文件夹下新建setsuna.names文件，文件内容如下：
>e-bike  (这是我自己的类别，可以根据自己的数据集进行调整)

&emsp;
## 5.训练
- 终于可以开始训练了，首先下载Imagenet上预先训练的权重文件放在darknet根目录里
下载地址如下：
>https://pjreddie.com/media/files/darknet53.conv.74
直接用浏览器就可以下，我是用Chrome浏览器下载的
![image](https://raw.githubusercontent.com/prometheus-code/darenetcode/master/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20191111171028.jpg)
- 然后开始使用我们的训练指令
>cd C:\darknet
darknet detector train cfg/my_data.data cfg/my_yolov3.cfg darknet53.conv.74
- 图片
![image](https://raw.githubusercontent.com/prometheus-code/darenetcode/master/1.png)
![image](https://raw.githubusercontent.com/prometheus-code/darenetcode/master/2.png)
- 最后在shell里出现Saving weight to 。。。你就成功了（因为上次成功没截图，也只能打文字了）
- **记得把my_yolov3_final.weights文件放在darknet根目录**(这样方便)
![image](https://raw.githubusercontent.com/prometheus-code/darenetcode/master/3.png)

## 6.测试
- 相信你在网上看到可能有人让你用以下代码测试：
>darknet detect cfg/my_yolov3.cfg weights/my_yolov3.weights myData//JPEGImages/0001.jpg
- 然后发现识别的名字不是你要的，当然这是因为没调用data文件，让我们看看darknet编码格式：
>./darknet detector test <data_cfg> <models_cfg> <weights> <test_file> [-thresh] [-out]
./darknet detector train <data_cfg> <models_cfg> <weights> [-thresh] [-gpu] [-gpus] [-clear]
./darknet detector valid <data_cfg> <models_cfg> <weights> [-out] [-thresh]
./darknet detector recall <data_cfg> <models_cfg> <weights> [-thresh]
- 以上darknet detect cfg/my_yolov3.cfg weights/my_yolov3.weights myData//JPEGImages/0001.jpg也没有错只是它调用的是coco.data,不是我们的my_data.data,所以出错了。以下是我的测试指令（仅供参考)
>darknet detector test cfg/my_data.data cfg/my_yolov3.cfg my_yolov3_final.weights setsuna/JPEGImages/000001.jpg
效果图：
![image](https://raw.githubusercontent.com/prometheus-code/liPrometheus/master/20191111174516.jpg)
---
参考文献：
1.https://github.com/AlexeyAB/darknet
2.https://mp.weixin.qq.com/s/hl_nFkw1oWm2TLpMFK2mZw
