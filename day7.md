# Day 7: 目标检测：YOLOv5的TensorRT加速部署



**《把YOLOv5改造成F1赛车：TensorRT加速の奥义》**

---

### 🚗 **场景代入：原版YOLOv5像家用轿车**
- **出厂状态**：在PyTorch上跑 → 时速60km/h（够用但不够爽）
- **目标**：改装成方程式赛车 → 用TensorRT飙到300km/h
- **核心矛盾**：既要速度飞起，又不能把乘客（检测精度）甩出车外

---

### 🔧 **改装四部曲：从买菜车到贴地飞行**

#### **第一步：拆座椅（模型减肥）**
```bash
# 把模型从PyTorch拆成零件状态
python export.py --weights yolov5s.pt --include onnx  # 输出onnx文件
```
**作用**：
- 卸掉PyTorch的豪华真皮座椅（冗余运算）
- 把零件标准化（转成ONNX通用格式）
**常见翻车**：
- 报错`Unsupported operator: aten::...` → 像拆出个外星零件（用`--dynamic`参数解决）

---

#### **第二步：装氮气（TensorRT优化）**
```python
# 使用TensorRT的涡轮增压器
import tensorrt as trt

builder = trt.Builder(logger)
network = builder.create_network()
parser = trt.OnnxParser(network, logger)

# 开始注入氮气！
builder.max_batch_size = 8
config = builder.create_builder_config()
config.set_flag(trt.BuilderFlag.FP16)  # 开启半精度模式
config.max_workspace_size = 1 << 30    # 给1GB内存当燃料
```
**改装清单**：
- **FP16加速**：给计算引擎灌氮气（速度↑↑，精度↓1%）
- **层融合**：把10个小零件焊成1个整体（减少数据传输）
- **显存预分配**：提前把赛道划好（避免运行时堵车）

---

#### **第三步：调教悬挂（精度校准）**
```python
# 防止加速后把猫认成狗
class MyCalibrator(trt.IInt8Calibrator):
    def get_batch(self, names):
        # 喂100张校准图片，像给赛车做动平衡
        return [np.random.randn(1,3,640,640).astype(np.float32)]
      
config.int8_calibrator = MyCalibrator()  # 开启INT8量子模式
```
**玄学参数**：
- **校准集**：最好用业务真实数据（别用喵星人图片校准车辆检测）
- **量化方式**：
  - `FP32`：原厂配置 → 稳如老狗
  - `FP16`：轻度改装 → 速度x2
  - `INT8`：爆改模式 → 速度x3（可能把斑马线识别成钢琴键）

---

#### **第四步：上路实测（部署推理）**
```c++
// C++版弹射起步代码
nvinfer1::IExecutionContext* context = engine->createExecutionContext();

// 把数据塞进引擎
void* buffers[2];
cudaMalloc(&buffers[0], input_size);
cudaMalloc(&buffers[1], output_size);

// 点火！起飞！
context->executeV2(buffers);
```
**性能对比**：
| 设备         | PyTorch(FPS) | TensorRT(FPS) | 效果                  |
|--------------|-------------|--------------|-----------------------|
| NVIDIA Jetson| 15          | 45           | 从PPT变成电影          |
| RTX 3090     | 120         | 240          | 肉眼可见的经费燃烧      |

---

### 🧨 **改装风险说明书**
**风险1：神秘报错**
- **症状**：`[TensorRT] ERROR: ...` 像摩斯电码
- **药方**：
  ```bash
  export CUDA_LAUNCH_BLOCKING=1  # 开启上帝视角
  sudo rm -rf ~/.nv/             # 清除缓存（玄学重启）
  ```

**风险2：精度跳水**
- **症状**：检测框开始群魔乱舞
- **抢救步骤**：
  1. 检查预处理是否一致（别用BGR喂RGB模型）
  2. 在`build_engine`时调高`workspace`（给更多内存）
  3. 给输出层烧香（误）

**风险3：设备冒烟**
- **症状**：风扇开始直升机模式
- **终极方案**：
  ```python
  while True:
      try:
          run_model()
      except:
          print("显卡说：爷要罢工！")
          time.sleep(10)  # 假装在调试
  ```

---

### 🚀 **高手进阶：改装厂黑科技**
1. **动态Shape支持**
   ```python
   profile = builder.create_optimization_profile()
   profile.set_shape("input", (1,3,320,320), (1,3,640,640), (1,3,1280,1280)) 
   ```
   - 让模型能处理任意尺寸输入（像变形金刚）

2. **多流并发**
   - 同时处理8个视频流（左手画圆，右手画方）

3. **自定义插件**
   - 自己写C++算子（像给赛车装翅膀，可能坠机也可能上天）

---

### 🤹 **哲学时间：速度与精度的平衡术**
- **真理1**：没有绝对完美的加速，只有妥协的艺术
- **真理2**：每提升10%速度，就离玄学调试近了一步
- **真理3**：当你觉得代码完美时，换张显卡就会爆炸

---

**课后作业**：
1. 尝试把模型加速到风扇起飞，记录主板温度
2. 故意用错预处理，观察模型如何花式误判
3. 当产品经理说"顺便加个人脸识别"，请优雅地递上《论加特征层的代价》

（附加题：用TensorRT部署时加入BGM《速度与激情》，体验代码与音浪双重冲击）

---

**改装完毕！现在你的YOLOv5已经不是从前的它了——
它是钮祜禄·YOLOv5 TRT Edition！**