# MixPrecisionTrt  TRT2021



## 一· 环境搭建

环境：ubuntu：18.04

cuda：11.1

cudnn：8.0

tensorrt：7.2.3.4

OpenCV：3.4.2

python3.8（使用其他版本可能会出现问题）

Conda

## 二. 准备训练好的pytorch-yolov5模型,保存到model_save目录下

可使用的pt模型
链接：https://pan.baidu.com/s/1uE8UimOupdPuKB3o2-AFgw 
提取码：qftf 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V2的分享

或者自己下载yolov5训练
```
git clone https://github.com.cnpmjs.org/ultralytics/yolov5.git #下载yolov5
```

下载calibration.cache在model_save文件夹下
链接：https://pan.baidu.com/s/1UvK6jRrqUy33eZ-RKEZ2EA 
提取码：0zc3 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V2的分享

2.1 yolov5s检测图形结果



## 三. 导出export.py                                

### 3.1 加载测试图片img，test_image/bus.jpg

--img_path test_image/bus.jpg

### 3.2 使用torch.onnx.export导出yolov5s.onnx

​    onnx模型保存到pt模型的同一目录下，可以使用netron工具，查看图形化onnx模型

```
#在netron_yolov5s.py中修改

netron.start('此处填充简化后的onnx模型路径')
python netron_yolov5s.py                      #即可查看模型输出名
```



### 3.3 使用yolov5s.pt计算预测

 --weights

```
y=model(img) 
```

### 3.4 使用ONNX Runtime计算预测

```
ort_session = onnxruntime.InferenceSession(f)
ort_outs = ort_session.run(None, ort_inputs)
```



### 3.5 比较精度，计算mse

```
np.testing.assert_allclose(to_numpy(y[1][0]), ort_outs[0], rtol=1e-03, atol=1e-05)
mse = np.sqrt(np.mean((to_numpy(y[1][0]) - ort_outs[0]) ** 2))
```

### 3.6 结果

<img src="./doc/3.6res.png" alt="image-20210430100723048" style="zoom:80%;" />

### 示例

```
`python export.py --weights  model_save/yolov5s.pt --img_path test_image/bus.jpg --img-size 640 --batch-size 1`
```

##   四. 生成tensorrt engine，精度转换                           

tensorrt_engine.py

util_trt_modify.py

calibrator.py

### 4.1 准备COCO数据集,onnx模型，测试图片，engine保存位置

CALIB_IMG_DIR = 'coco/images/train2017'

--onnx_model_path = 'model_save/yolov5s.onnx'

--img_path = 'test_image/bus.jpg'

--engine_model_path = "model_save/yolov5s.trt"

###   4.2 确定精度

--quantize fp32/fp16/int8/int4

###   4.3 使用engine预测

`outputs = do_inference(context, bindings=bindings, inputs=inputs, outputs=outputs, stream=stream)`

### 4.4 比较精度，计算mse

```
mse = np.sqrt(np.mean((feat[0] - ort_outs[0]) ** 2))
```

### 4.5 结果

fp32

<img src="./doc/fp32.png" alt="fp32.png" style="zoom:80%;" />



fp16

<img src="./doc/fp16.png" alt="fp16.png" style="zoom:80%;" />


int8

<img src="./doc/int8.png" alt="int8" style="zoom:80%;" />

int4

<img src="./doc/Ours.png" alt="Ours" style="zoom:80%;" />



### 示例

```
python tensorrt_engine.py  --quantize fp32 --onnx_model_path model_save/yolov5s.onnx --img_path test_image/bus.jpg --engine_model_path  model_save/yolov5s.trt
```

 

 

##  五.图像检测

detect.py

### 5.1选择检测使用的模型

​    --engine_model_path

​    --onnx_model_path

###  5.2选择需要检测的图片

--img_path test_image/bus.jpg

### 5.3结果

yolov5s.trt

<img src="./doc/fp32_detect.png" alt="fp32_detect" style="zoom:80%;" />

yolov5s_fp16.trt

<img src="./doc/fp16_detect.png" alt="fp16_detect" style="zoom:80%;" />

yolov5s_int8.trt

<img src="./doc/int8_detect.png" alt="int8_detect" style="zoom:80%;" />

yolov5s_int8.trt

<img src="./doc/int4_detect.png" alt="int4_detect" style="zoom:80%;" />

### 示例

```
python detect.py --engine_model_path model_save/yolov5s.trt --img_path test_image/bus.jpg   #使用engine检测

python detect.py --onnx_model_path model_save/yolov5s.trt  --img_path test_image/bus.jpg    #使用onnx检测
```

