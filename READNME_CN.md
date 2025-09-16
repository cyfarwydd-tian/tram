# TRAM

**论文官方实现**  
**TRAM: Global Trajectory and Motion of 3D Humans from in-the-wild Videos**  
作者：Yufu Wang, Ziyun Wang, Lingjie Liu, Kostas Daniilidis  
[项目页面]

`example_output.mp4`

---

## 更新日志

- **[2025/06]** TRAM 已集成进我们的新工作 **PromptHMR**，欢迎查看！
- **[2025/02]** 新增训练代码与预处理数据。
- **[2025/02]** 更新了更好的重力与地面预测；新增 EMDB 评测。
- **[2024/04]** 初始发布。

---

## 安装

1. 使用 `--recursive` 参数克隆本仓库：
   ```bash
   git clone --recursive https://github.com/yufu-wang/tram
   ```

2. 创建并激活新的 Anaconda 环境：
   ```bash
   conda create -n tram python=3.10 -y
   conda activate tram
   ```

3. 安装依赖：
   ```bash
   bash install.sh
   ```

4. 编译 **DROID-SLAM**。若遇到困难，请参考其官方发布页面以获得更多信息。  
   在本项目中，我们对 DROID 做了修改以支持 **masking**：
   ```bash
   cd thirdparty/DROID-SLAM
   python setup.py install
   cd ../..
   ```

---

## 准备数据

- 在 **SMPLify** 与 **SMPL** 注册账号。脚本会使用你在这两个网站的用户名和密码来下载 **SMPL 模型**。
- 脚本还会下载训练好的 **检查点（checkpoints）** 与一个 **示例视频**。  
- 注意：第三方模型拥有各自的许可证协议。

运行以下命令，将所有模型与检查点下载到 `data/` 目录下；同时会下载 `example_video.mov` 以用于演示：
```bash
bash scripts/download_models.sh
```

---

## 在视频上运行 Demo

该项目整合了完整的 4D 人体系统：**目标跟踪**、**SLAM** 与 **世界坐标系下的 4D 人体捕获**。  
我们将核心功能拆分为多个脚本，需要 **按顺序** 运行。每一步都会将结果保存下来供下一步使用。所有结果会保存在一个与视频 **同名** 的文件夹中。

1. **运行 Masked DROID-SLAM（同时进行人体检测与跟踪）**
   ```bash
   python scripts/estimate_camera.py --video "./example_video.mov"
   # 若相机为静态，可指定该选项（算法也会尝试自动判断）：
   python scripts/estimate_camera.py --video "./another_video.mov" --static_camera
   ```

2. **使用 VIMO 进行 4D 人体捕获**
   ```bash
   python scripts/estimate_humans.py --video "./example_video.mov"
   ```

3. **整合所有结果并渲染输出视频**
   ```bash
   python scripts/visualize_tram.py --video "./example_video.mov"
   ```

在提供的 `./example_video.mov` 上依次运行以上三个脚本，会在 `./results/exapmle_video`（原文如此） 下创建并保存所有结果。  
更多可用参数请查看各脚本的 `--help`。

---

## 评估（Evaluation）

你可以在 **EMDB** 数据集上从头进行推理与评估：
```bash
# 推理与评估（结果将保存到 "results/emdb"）
bash scripts/emdb/run.sh
```

也可以跳过推理，直接下载我们已保存的结果，然后仅运行评估：
```bash
# 仅评估
python scripts/emdb/run_eval.py --split 2 --input_dir "results/emdb"
```

---

## 训练（Training）

### 数据（Data）
我们提供了预处理后的数据（如裁剪结果）与标注（**链接见“此处”**）。请下载预处理数据，并在 `data_config.py` 中将路径修改为你的本地路径。  
对于 **BEDLAM（30fps）**，请下载 30fps 版本（如 `.mp4`），并使用：
```bash
scripts/extract_bedlam_jpg.py
scripts/crop_datasets.py
```
将其处理为正确的格式。若你在此步骤遇到问题，欢迎提交 issue。

### 预训练权重（Checkpoint）
运行以下命令下载 **HMR2b** 作为初始化权重：
```bash
bash scripts/download_pretrain.sh
```

### 开始训练
```bash
python train.py --cfg configs/config_vimo.yaml
```
训练结果会保存在 `results/EXP_NAME` 目录下。

---

## 致谢（Acknowledgements）

本项目受以下开源工作的启发并复用/改写了部分代码：

- **WHAM**：可视化与评估
- **HMR2.0**：基线骨干网络
- **DROID-SLAM**：SLAM 基线
- **ZoeDepth**：度量深度预测
- **BEDLAM**：大规模视频数据集
- **EMDB**：评测数据集

此外，流水线还包含：**Detectron2**、**Segment Anything**、**DEVA-Track-Anything**。

---

## 引用（Citation）

```bibtex
@article{wang2024tram,
  title={TRAM: Global Trajectory and Motion of 3D Humans from in-the-wild Videos},
  author={Wang, Yufu and Wang, Ziyun and Liu, Lingjie and Daniilidis, Kostas},
  journal={arXiv preprint arXiv:2403.17346},
  year={2024}
}
```
