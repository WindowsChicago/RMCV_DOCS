# OpenVINO xml模型反转回onnx教程

#### by ZYL

##### 经测试，大部分xml均可转回onnx，与原onnx性能极为相近，目前遇到的问题转出的onnx无法使用可能无法使用tensorrt8转成engine，而使用tensorrt10可以正常转换为engine使用，目前只测试了yolov5和v8的模型，可以正常反转，其他的暂未测试

### 配置环境与运行转换：

1.建议使用conda单独为这个程序建一个环境，使用3.10：

```
conda create -n openvino2onnx python=3.10
conda activate openvino2onnx
```

2.pip安装openvino2onnx组件：

```
pip install openvino2onnx
```

**注意：有些国内conda软件源没有该包，我测试了清华源没有，阿里源有**

3.onnx组件降级：这一步很重要，pip直接下载的onnx包为最新版本，直接运行openino2onnx命令会报错：

```
ModuleNotFoundError: No module named 'onnx.mapping'
```

主要是因为25年后的onnx删除了mapping，所以应该切换回旧版的onnx

```
pip install onnx==1.18.0
```

4.安装openino：实际上经测试这个转换程序不安装openvino只会有个警告，还是能正常运行，转换出的文件大小没有差异，不过我没有测试过能不能用，还是推荐先安装python版openvino，并且注意这个程序不支持openvino2023之前的版本，使用最新的2025开始会有警告：

```
DeprecationWarning: The `openvino.runtime` module is deprecated and will be removed in the 2026.0 release. Please replace `openvino.runtime` with `openvino`.
```

具体影不影响输出的文件我每测试过，建议直接使用2024版，该版本无任何报错：

```
pip install openvino==2024
```

5.执行转换：

在要转换的xml模型位置打开终端激活openvino2onnx环境，执行命令：

```
openvino2onnx 模型文件名.xml
```

注意不用加sudo，显示move saved to 文件名_o2o即转换完成，此时可以在目录下找到对应的onnx，使用tensorrt转换时会发现模型信息的创作者是openvino2onnx
