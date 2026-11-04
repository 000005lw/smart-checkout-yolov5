# 智购 · 校园超市自助结算机（商品识别部分）

> **一句话**：把「排队扫条形码结账」换成「商品往摄像头前一放就识别品类和价格」——用 YOLOv5 训练 5 类商品检测模型，导出 NCNN，部署到广和通 SC171 模组的 Aidlux 环境里离线跑。

**比赛**：2024 年全国大学生物联网设计竞赛 · 作品《“智购”校园超市智能结算机》（参赛，未获奖）
**相关奖项**：同源作品《“智购”校园超市智能结算机》获 全国大学生嵌入式芯片与系统设计竞赛 · 南部赛区 **省级二等奖**

---

## 我的工作（林建鸿）

- **队长（3 人团队）**，负责商品识别模型的训练与端侧部署，另两名队员负责 Android 端与云端
- **数据集**：自建 5 类商品数据集（方便面 / 可乐 / 薯片 / 牙膏 / 纸巾），按 YOLO 格式标注
- **训练**：YOLOv5s，输入 640×640，SGD，150 epoch，单卡
- **部署**：PyTorch 权重 → NCNN（`deploy/best.param`），落到 SC171 模组的 Aidlux 环境做端侧实时识别
- **联调**：识别结果写入 Android / Aidlux 公有目录，由账单模块读取后生成小票与付款二维码

## 指标（来自 `runs/exp21/results.csv`，自建验证集）

| 指标 | 结果 |
| --- | --- |
| mAP@0.5 | **98.0%** |
| mAP@0.5:0.95 | **77.0%** |
| Precision | 98.5% |
| Recall | 95.5% |
| 训练轮次 | 150 epoch（另有一次 300 epoch 的对照，mAP@0.5 94.4%） |

> 注：这是 5 类商品、自建小数据集上的结果，换场景需要重新标定，不能直接当通用货架识别指标。

## 目录

```text
smart-checkout-yolov5/
├── configs/bvn.yaml      # 数据集配置（5 类，路径指向 datasets/）
├── weights/best.pt       # 训练好的权重（对应 exp21）
├── deploy/best.param     # NCNN 部署产物（端侧推理用）
├── runs/exp21/           # 训练日志：opt.yaml（超参）+ results.csv（逐 epoch 指标）
└── README.md
```

## 怎么复现

```bash
# 1. 准备数据（本仓库不含图片，745 MB 原始数据集未上传）
#    datasets/
#    ├── images/train  images/val
#    └── labels/train  labels/val

# 2. 用 Ultralytics YOLOv5 v6.2 训练
git clone -b v6.2 https://github.com/ultralytics/yolov5
cd yolov5 && pip install -r requirements.txt
python train.py --data ../configs/bvn.yaml --weights yolov5s.pt \
                --img 640 --epochs 150 --batch-size 1

# 3. 导 NCNN 部署
python export.py --weights ../weights/best.pt --include ncnn
```

## 说明（据实交代）

- 本仓库只保留**与本项目相关的配置、权重、日志**；YOLOv5 训练框架本身来自 [Ultralytics YOLOv5 v6.2](https://github.com/ultralytics/yolov5)（GPL-3.0），不在本仓库重复上传。
- **Android 端代码与 Aidlux 账单程序已遗失**，这里只覆盖 CV 模型这条链路。
- `deploy/best.param` 是 NCNN 的结构文件，配套的 `.bin` 权重当时未一并归档。
- 原始数据集（745 MB 图片）和 24 组训练记录中的其余 23 组未上传，只保留最优的一组。

## 作品文档

完整设计与实现说明见《(第二版)2024 年全国大学生物联网设计竞赛设计文档》，包含端—云架构、MQTT/HTTP 链路、华为云 IoTDA/OBS 营收报表与商业价值分析。
