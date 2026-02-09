# Fakeddit 文本‑only（BERT）Notebook 详解

本文档说明 `Fakeddit_TextOnly_BERT.ipynb` 的用途、运行流程、关键配置项与结果解读，帮助你按论文思路完成 **文本分支基线**。

---

## 1. 目标与定位
该 Notebook 是你多模态方法的 **文本分支基线**：
- 数据：Fakeddit v2.0 的 `multimodal_only_samples`
- 输入：`clean_title`（已清洗文本）
- 任务：2‑way / 3‑way / 6‑way 分类
- 模型：BERT（`bert-base-uncased`）

用途：先建立文本基线，之后再加入图像分支做多模态融合。

---

## 2. 环境与依赖（Linux 服务器）
你当前约束：**PyTorch 2.8.0 / Python 3.12 / Ubuntu 22.04 / CUDA 12.8**。

本项目已生成对应依赖文件：
```
requirements.txt
```
安装方式：
```bash
pip install -r requirements.txt
```

建议运行前检查：
- `nvidia-smi` 是否能看到 GPU
- Notebook 中的“环境检查”单元输出是否为 `cuda available: True`

---

## 3. 数据与路径
Notebook 默认使用：
```
Fakeddit datasetv2.0/multimodal_only_samples/
```
包含：
- `multimodal_train.tsv`
- `multimodal_validate.tsv`
- `multimodal_test_public.tsv`

文本字段：`clean_title`

**注意**：
- 仅使用 `multimodal_only_samples` 能保证以后直接接入图像分支
- `clean_title` 已符合论文清洗规则

---

## 4. 标签与映射
Notebook 使用的数值映射已根据本地 TSV 验证：

**2‑way**
- `1` = True
- `0` = Fake

**3‑way**
- `0` = True
- `1` = Fake with true text
- `2` = Fake with false text

**6‑way**
- `0` = True
- `1` = Satire/Parody
- `2` = False Connection
- `3` = Imposter
- `4` = Manipulated
- `5` = Misleading

---

## 5. Notebook 结构说明

### 5.1 配置区（超参数 & 路径）
关键变量：
- `TASK`：2 / 3 / 6
- `MODEL_NAME`：默认 `bert-base-uncased`
- `MAX_LEN`：最大文本长度
- `BATCH_SIZE`：batch size
- `EPOCHS`：训练轮数
- `LR`：学习率

建议：
- 先用 `TASK=2` 跑通
- 想快速验证可设置 `MAX_SAMPLES=10000`

### 5.2 数据加载
- 使用 `clean_title`
- 自动过滤空文本
- 读取 `*_way_label` 作为标签

### 5.3 分词与 DataLoader
- 使用 `AutoTokenizer` 与 BERT 匹配
- `collate_fn` 进行 padding / truncation

### 5.4 模型与损失
- `AutoModelForSequenceClassification`
- 使用 **类别权重** 缓解不平衡（对 6‑way 特别重要）

### 5.5 训练与评估
每个 epoch 输出：
- `train_loss`
- `val_loss`
- `val_acc`
- `val_f1`（macro）
- `confusion_matrix`

### 5.6 测试与保存
训练结束后：
- 在 test 集评估
- 保存模型到 `outputs/bert_text_only_{TASK}way`

---

## 6. 结果如何解读

**推荐关注**：
- Macro‑F1（尤其是 3/6‑way）
- 混淆矩阵中 False Connection / Misleading 等类别的区分

如果你后面加图像分支，多模态模型应该：
- 在 False Connection / Manipulated 等类别显著提升

---

## 7. 常见问题

**Q1: 训练很慢？**
- 降低 `MAX_LEN` 或 `BATCH_SIZE`
- 设置 `MAX_SAMPLES` 做小样本预跑

**Q2: GPU 没被用到？**
- 检查 `cuda available` 输出
- 确认驱动与 CUDA 版本匹配

**Q3: 类别不平衡导致只预测大类？**
- 本 Notebook 已加类权重
- 可进一步使用 `focal loss` 或 `oversampling`

---

## 8. 后续多模态扩展建议
你后续可以：
1) 加图像分支（ResNet/ViT）提取 `image_emb`
2) 加语义一致性分支（`cosine` 或 MLP）
3) 融合：`concat(text_emb, image_emb, consistency)` → MLP

如果你需要，我可以在这个 Notebook 上继续扩展多模态版本。
