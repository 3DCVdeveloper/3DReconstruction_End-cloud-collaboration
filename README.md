# 3DReconstruction_End-cloud-collaboration
Real time dense 3D reconstruction through end-to-end cloud collaboration based on embedded platform

# 基于嵌入式的实时三维重建系统运行流程

#### 运行相机节点—>运行IMU模块->启动云端系统->启动移动端系统

# 云端重建系统

代码说明文档可通过doxygen在根目录生成，readme.txt中提供了工程的整体说明，demo/中有工程的可执行文件(demo)及其制作方法。 

# 环境

Ubuntu 16.04、18.04

cmake; 

Eigen >= 3.3.5 (可从官网下载3.3.9); 

OpenGL; (可参考https://blog.csdn.net/zhangliang_571/article/details/25241911/)

CUDA >= 8.0

kindr

AzureKinect、Realsense等传感器的驱动及SDK。

# 运行

传感器在线输入:
```shell
./FastFusionV2-x86_64.AppImage calibration_file
```

# 可执行文件说明(Demo)

通过appimage格式封装，/demo/ 中的.AppImage 为打包好的可执行文件，在opensuse leap 15.2, ubuntu16.04~20.04上通过运行测试。建议优先选择ubuntu 18.04。
若遇见permission denied 错误， 需要赋予对应的执行权限:

```shell
chmod u+x FastFusionV2-x86_64.AppImage
```

打包工具及方法详见 /demo/tools/。

## 相机参数文件说明 （calibration）

以[FMDataset](https://github.com/zhuzunjie17/FastFusion)对应的Realsense ZR300相机参数文件（/Files/realsense/calib.txt）为例：

```latex

640 480                 % color 图像size    width, height
608.446 607.847         % color 相机内参    fx, fy
331.332 245.549         % color 相机内参    cx, cy

640 480                 % depth 图像size    width, height
583.980 583.980         % depth 图像内参    fx, fy
325.103 240.718         % dpeth 图像内参    cx,cy

0.999980211 -0.00069964811 -0.0062491186 -0.057460     % transformation matrix 
0.000735448557 0.999983311 0.00572841847 -0.001073     % from depth to color
0.00624500681 -0.00573290139 0.999964058 -0.002205     % Extrinsics

    0.999758    0.0138749   -0.0170966     0.036333    % transformation matrix
  -0.0134149     0.999552    0.0267351    0.0029128    % from depth to imu
   0.0174598   -0.0264998     0.999496 -0.000897499    % Extrinsics

affine 0.001 0.0                   % depth module, use affine normally.    depth scale, 0

4e-7 5.88e-4 2.5e-09 1.22e-6       % imu Noise model       
% acc.bias_Covariances[m^2/s^5] acc.noise_Covariances[m^2/s^3] gyro.bias_Covariances[rad^2/s^3] gyro.noise_Covariances[rad^2/s]

% IMU noise model可参考 https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model
```

# 主要交互说明

| 按键  | 功能                                                              |
| --- |:--------------------------------------------------------------- |
| 鼠标　| 左键旋转，右键平移，中键绕光轴旋转　　　　　　　　　　　　　　　　　　　　　　|
| n   | PROCESS_FRAME模式，单帧模式运行 ，需反复按下n键                                 |
| b   | PROCESS_VIDEO模式， 以视频的方式允许                                       |
| ESC | 退出                                                              |
| f   | 在free viewpoint(自由视角) 和 follow camera(固定视角)切换，默认为free viewpoint |
| u,o | 自由视角下的缩放按钮
| c   | 模型表面材质， 在 纹理贴图， 法向量贴图， 置信度贴图，默认灰色模型 之间切换                        |
| w   | 保存mesh 网格数据， obj 格式，带color信息，使用meshlab可看点云                      |
|     |                                                                 |

# 移动端重建系统

移动端重建系统基于ROS，将云端系统作为ros功能包运行

## 运行

作为ros功能包，移动端重建系统需要通过catkin进行编译

```shell
catkin_make
```

编译产生库文件及可执行文件

运行移动端重建系统前需要先开启相机及IMU

### 相机启动

```
roslaunch astra_camera astrapro.launch
```

### IMU启动

```
roslaunch imu_launch imu_msg.launch
```

### 系统运行

```
./FastFusionV2-arm64.AppImage calibration_file
```

# ROS下使用astrapro

1. 安装ros

2. 安装依赖

   ```
   $ sudo apt install ros-*-rgbd-launch ros-*-libuvc ros-*-libuvc-camera ros-*-libuvc-ros
   ```

3. 创建工作空间

   ```
   $ mkdir -p ~/ros_astra_camera/src
   $ cd ~/ros_astra_camera/src
   $ catkin_init_workspace
   ```

4. 配置工作空间环境变量

   ```
   $ cd ros_astra_camera
   $ sourse ./devel/setup.bash
   ```

5. 下载ros下的astra功能包

   ```
   $ cd ros_astra_camera/src
   $ git clone https://github.com/orbbec/ros_astra_camera
   ```

6. 创建Astra udev rule

   ```
   $ roscd astra_camera
   $ ./scripts/create_udev_rules
   ```

7. 在工作空间编译astra相机

   ```
   $ cd ~/ros_astra_camera
   $ catkin_make --pkg astra_camera
   ```

8. 使用astrapro

   ```
   $  roslaunch astra_camera astrapro.launch 
   ```

9. 实时显示视频流

   ```
   rviz
   或
   rqt_image_view
   ```

# IMU使用

## 打开USB设备的可执行权限：

```shell
   $ sudo chmod 777 /dev/ttyUSB0
```

## 安装ROS serial软件包

1. 首先执行如下命令，下载安装serial软件包：

```shell
$ sudo apt-get install ros-kinetic-serial
```

2. 然后输入`roscd serial`命令，进入serial下载位置，如果安装成功，就会出现如下信息：

```shell
$:/opt/ros/kinetic/share/serial
```

## 编译serial_imu_ws工作空间

1. 打开终端进入serial_imu_ws 目录

2. 执行`catkin_make`命令，编译成功后出现完成度100%的信息

## 显示数据

共有三种查看数据方式：

1. 显示所有的数据信息，便于查看数据。
2. 打印ROS标准imu_msg 数据
3. rviz工具实现可视化
4. 3D显示

### 	输出IMU原始数据

1.打开另一个终端，执行：

```shell
$ roslaunch imu_launch imu_msg.launch imu_package:=0x91
```

2. 如果执行失败，提示找不到相应的launch文件，则需要配置环境，在当前终端执行：

```shell
$source <serial_imu_ws_dir>/devel/setup.bash
```

3. 执行成功后，就可以看到所有的信息：

```txt
     Devie ID:     0
    Run times: 0 days  3:26:10:468
  Frame Rate:   100Hz
       Acc(G):   0.933    0.317    0.248
   Gyr(deg/s):   -0.02     0.30    -0.00
      Mag(uT):    0.00     0.00     0.00
   Eul(R P Y):   52.01   -66.63   -60.77
Quat(W X Y Z):   0.770    0.066   -0.611   -0.172

```

### 	输出ROS标准 Imu.msg

执行`roslaunch imu_launch imu_msg.launch`命令。执行成功后，就可以看到ROS定义的IMU话题消息：

```txt
header: 
  seq: 595
  stamp: 
    secs: 1595829903
    nsecs: 680423746
  frame_id: "base_link"
orientation: 
  x: 0.0663746222854
  y: -0.611194491386
  z: -0.17232863605
  w: 0.769635260105
orientation_covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
angular_velocity: 
  x: 0.0851199477911
  y: 0.0470183677971
  z: 0.00235567195341
angular_velocity_covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
linear_acceleration: 
  x: 0.93323135376
  y: 0.317857563496
  z: 0.247811317444
linear_acceleration_covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]

```

### 	rviz可视化

打开终端，执行`roslaunch imu_launch imu_rviz.launch`命令，执行成功后，rviz工具被打开。

先点击左下角的Add标签，然后在弹出窗口中，选择 By display type标签，查找rviz_imu_plugin；找到之后，选择它下面的imu标签，点击OK, 这时，我们可以看到rviz的左侧的展示窗口中已经成功添加上了Imu的标签。在FixedFrame中填入**base_link** 。topic中添加 **/IMU_data**。这时，可以看到坐标系随传感器改变而改变

### 3D显示

打开终端，执行`roslaunch imu_launch imu_display_3D.launch`命令，执行成功后，会出现3D图形显示

# 失败检测模型训练与测试

将7 scenes数据集训练得到的3维特征向量按照一定比例分为训练集与测试集，丢入libsvm中进行训练测试

## LIBSVM

Libsvm 是一个简单、易用、高效的 SVM 分类和回归软件。 它解决了 C-SVM 分类、nu-SVM 分类、one-class-SVM、epsilon-SVM 回归和 nu-SVM 回归。 它还为C-SVM分类提供了自动模型选择工具。 本文档解释了 libsvm 的使用。

Libsvm 可在 http://www.csie.ntu.edu.tw/~cjlin/libsvm 请在使用 libsvm 之前阅读 COPYRIGHT 文件。

## 目录

- 安装和数据格式
- `svm-train' 用法
- `svm-predict' 用法
- `svm-scale'用法
- 实际使用技巧
- 例子
- 预计算内核



### 安装和数据格式

在 Unix 系统上，输入 make' 来构建svm-train'、svm-predict' 和svm-scale' 程序。 不带参数运行它们以显示它们的用法。

在其他系统上，请参阅“Makefile”来构建它们（例如，请参阅此文件中的“构建 Windows 二进制文件”）或使用预构建的二进制文件（Windows 二进制文件位于“windows”目录中）。

训练和测试数据文件的格式为：

<标签> <索引1>:<值1> <索引2>:<值2> ...

每行包含一个实例并以 '\n' 字符结束。 对于训练集中的<label>，我们有以下几种情况。

- 分类：<label> 是一个整数，表示类标签（支持多类）。
- 对于回归，<label> 是目标值，可以是任何实数数字。
- 对于一类 SVM，不使用 <label>，可以是任意数字。

在测试集中，<label> 仅用于计算准确率或误差。 如果它是未知的，任何数字都可以。 对于一类 SVM，如果已知非异常值/异常值，则它们在测试文件中的标签必须为 +1/-1 以进行评估。

<index>:<value> 对给出了一个特征（属性）值：<index> 是一个从 1 开始的整数，<value> 是一个实数。 唯一的例外是预计算内核，其中 <index> 从 0 开始； 请参阅预计算内核部分。 指数必须按升序排列。

此包中包含的示例分类数据是“heart_scale”。 要检查您的数据格式是否正确，请使用“tools/checkdata.py”（详细信息在“tools/README”中）。

输入svm-train heart_scale'，程序将读取训练数据并输出模型文件heart_scale.model'。 如果您有一个名为 heart_scale.t 的测试集，则键入“svm-predict heart_scale.t heart_scale.model output”以查看预测精度。 “输出”文件包含预测的类标签。

对于分类，如果训练数据只有一个类（即所有标签都相同），那么‘svm-train’会发出警告消息：‘警告：训练数据只有一个类。 详情请参阅自述文件，'这意味着训练数据非常不平衡。 测试时直接返回训练数据中的标签。

这个包中还有一些其他有用的程序。

**svm-scale:**

这是一个用于缩放输入数据文件的工具。

**svm-toy:**

这是一个简单的图形界面，显示了 SVM 如何在平面中分离数据。 您可以在窗口中单击以绘制数据点。 使用“更改”按钮选择类别 1、2 或 3（即最多支持三个类别）、“加载”按钮从文件加载数据、“保存”按钮将数据保存到文件、“运行”按钮 获取 SVM 模型，然后按“清除”按钮清除窗口。

您可以在窗口底部输入选项，选项的语法与“svm-train”相同。

请注意，“加载”和“保存”在分类和回归案例中都考虑了密集数据格式。 对于分类，每个数据点都有一个必须为 1、2 或 3 的标签（颜色）和 [0,1] 中的两个属性（x 轴和 y 轴值）。 对于回归，每个数据点在 [0, 1) 中有一个目标值（y 轴）和一个属性（x 轴值）。

在相应的目录中键入“make”以构建它们。

**svm-train:**

使用方法: svm-train [选项] 训练设置文件[模型文件]

选项：

-s 支撑向量机类型 : 设置支撑向量机类型 (默认0)

```
0 -- C-SVC（多类分类）
1 -- nu-SVC（多类分类）
2——一类SVM
3 -- epsilon-SVR（回归）
4 -- nu-SVR（回归）
```

-t 核函数类型: 设置核函数 (默认2)

```
0 -- 线性：u'*v
1 -- 多项式：(gamma*u'*v + coef0)^degree
2 -- 径向基函数：exp(-gamma*|u-v|^2)
3 -- sigmoid: tanh(gamma*u'*v + coef0)
4 -- 预计算内核（training_set_file 中的内核值）
```

-d degree : 在核函数中设置度数（默认为 3）

-g gamma : 在核函数中设置 gamma (默认 1/num_features)

-r coef0 : 在内核函数中设置 coef0（默认为 0） 

-c cost : 设置C-SVC、epsilon-SVR、nu-SVR的参数C（默认为1） 

-n nu : 设置nu-SVC、一类SVM、nu-SVR的参数nu（默认0.5） 

-p epsilon : 设置 epsilon-SVR 损失函数中的 epsilon（默认 0.1） 

-m cachesize ：以 MB 为单位设置缓存内存大小（默认 100） 

-e epsilon : 设置终止标准的容差（默认 0.001） 

-h 收缩：是否使用收缩启发式，0 或 1（默认为 1） 

-b probability_estimates ：是否为概率估计训练 SVC 或 SVR 模型，0 或 1（默认为 0） 

-wi weight : 将类 i 的参数 C 设置为 weight*C，对于 C-SVC（默认为 1） 

-v n：n 折交叉验证模式 

-q ：安静模式（无输出）

-v 将数据随机分成 n 个部分并计算它们的交叉验证准确度/均方误差。

有关输出的含义，请参阅 libsvm FAQ。

**svm-predict:**

用法：svm-predict [options] test_file model_file output_file options： -b probability_estimates：是否预测概率估计，0或1（默认0）； 对于一类 SVM 仅支持 0

model_file 是 svm-train 生成的模型文件。 test_file 是你要预测的测试数据。 svm-predict 将在 output_file 中产生输出。

**svm-scale:**

options：svm-scale [options] data_filename 选项： -l lower : x 缩放下限（默认 -1） -u upper : x 缩放上限（默认 +1） -y y_lower y_upper ：y 缩放限制（默认值：无 y 缩放） -s save_filename : 将缩放参数保存到 save_filename -r restore_filename : 从 restore_filename 恢复缩放参数

有关示例，请参阅此文件中的“示例”。



### 实际使用技巧

- 扩展您的数据。 例如，将每个属性缩放为 [0,1] 或 [-1,+1]。

- 对于 C-SVC，请考虑使用工具目录中的模型选择工具。

- nu-SVC/one-class-SVM/nu-SVR 中的 nu 近似于训练的分数 错误和支持向量。

- 如果用于分类的数据不平衡（例如，许多正面和 少数否定），通过 -wi 尝试不同的惩罚参数 C（参见 下面的例子）。

- 为巨大的问题指定更大的缓存大小（即更大的 -m）。

### 例子

> svm-scale -l -1 -u 1 -s range train > train.scale svm-scale -r range test > test.scale

将训练数据的每个特征缩放到 [-1,1]。 缩放因子存储在文件范围中，然后用于缩放测试数据。

> svm-train -s 0 -c 5 -t 2 -g 0.5 -e 0.1 data_file

使用 RBF 核 exp(-0.5|u-v|^2)、C=10 和停止容差 0.1 训练分类器。

> svm-train -s 3 -p 0.1 -t 0 data_file

在损失函数中用线性核 u'v 和 epsilon=0.1 求解 SVM 回归。

> svm-train -c 10 -w1 1 -w-2 5 -w4 2 data_file

训练一个分类器，第 1 类的惩罚为 10 = 1 * 10，第 -2 类的惩罚为 50 = 5 * 10，第 4 类的惩罚为 20 = 2 * 10。

> svm-train -s 0 -c 100 -g 0.1 -v 5 data_file

使用参数 C = 100 和 gamma = 0.1 对分类器进行五重交叉验证

> svm-train -s 0 -b 1 data_file svm-predict -b 1 test_file data_file.model output_file

获得带有概率信息的模型并用概率估计预测测试数据

### 预计算内核

用户可以预先计算内核值并将它们作为训练和测试文件输入。 那么 libsvm 不需要原始的训练/测试集。

假设有 L 个训练实例 x1, ..., xL 和。 设 K(x, y) 为核两个实例 x 和 y 的值。 输入格式

New training instance for xi:

<label> 0:i 1:K(xi,x1) ... L:K(xi,xL)

New testing instance for any x:

<label> 0:? 1:K(x,x1) ... L:K(x,xL)

也就是说，在训练文件中，第一列必须是 xi 的“ID”。 在测试中， ? 可以是任何值。

必须明确提供包括零在内的所有内核值。 训练/测试文件的任何排列或随机子集也是有效的（参见下面的示例）。

注意：格式与之前在 libsvmtools 中发布的预计算内核包略有不同。

Examples:

假设原始训练数据有 3 个四特征实例，测试数据有 1 个实例：

```
15  1:1 2:1 3:1 4:1
45      2:3     4:3
25          3:1

15  1:1     3:1
```

如果使用线性内核，我们有以下新的训练/测试集：

```
15  0:1 1:4 2:6  3:1
45  0:2 1:6 2:18 3:0
25  0:3 1:1 2:0  3:1

15  0:? 1:2 2:0  3:1

? can be any value.
```

上述训练文件的任何子集也是有效的。 例如，

```
25  0:3 1:1 2:0  3:1
45  0:2 1:6 2:18 3:0
```

意味着核矩阵是

```
[K(2,2) K(2,3)] = [18 0]
[K(3,2) K(3,3)] = [0  1]
```
