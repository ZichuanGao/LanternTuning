# Fakeddit v2.0（本地副本）数据集中文说明

本文档解释本仓库 `Fakeddit datasetv2.0/` 下的数据集结构、字段、标签含义与数值映射。说明依据两部分：
1) 论文 **“r/Fakeddit: Fine‑grained Fake News Detection”**（`/Users/gaozichuan/Downloads/1911.03854v2.pdf`）
2) 对本地 TSV 文件的实际统计与验证。

## 1) 数据集是什么？
Fakeddit 是一个基于 Reddit 的大规模假新闻检测数据集，具有 **多模态**（文本 + 图像 + 元数据）特性，并为每条样本提供三种粒度的标签：
- **2‑way**：真假二分类
- **3‑way**：真 + 两种假
- **6‑way**：真 + 5 类细粒度假

标签采用 **distant supervision（远程监督）**：先把 subreddit 归入固定类别，再将该 subreddit 中的所有样本赋予对应标签。

## 2) 目录结构
```
Fakeddit datasetv2.0/
├─ all_samples (also includes non multimodal)/
│  ├─ all_train.tsv
│  ├─ all_validate.tsv
│  └─ all_test_public.tsv
└─ multimodal_only_samples/
   ├─ multimodal_train.tsv
   ├─ multimodal_validate.tsv
   └─ multimodal_test_public.tsv
```

- **all_samples**：包含有图 + 无图样本
- **multimodal_only_samples**：仅包含有图样本（图文配对）

## 3) Split 样本数量（以本地文件为准）
> 下面是样本数（不含表头）。

**All samples**
| Split | 文件 | 样本数 |
|---|---|---:|
| Train | `Fakeddit datasetv2.0/all_samples (also includes non multimodal)/all_train.tsv` | 878,570 |
| Validate | `Fakeddit datasetv2.0/all_samples (also includes non multimodal)/all_validate.tsv` | 92,482 |
| Test (public) | `Fakeddit datasetv2.0/all_samples (also includes non multimodal)/all_test_public.tsv` | 92,484 |

**Multimodal only**
| Split | 文件 | 样本数 |
|---|---|---:|
| Train | `Fakeddit datasetv2.0/multimodal_only_samples/multimodal_train.tsv` | 564,211 |
| Validate | `Fakeddit datasetv2.0/multimodal_only_samples/multimodal_validate.tsv` | 59,364 |
| Test (public) | `Fakeddit datasetv2.0/multimodal_only_samples/multimodal_test_public.tsv` | 59,338 |

## 4) 文件格式
- TSV（制表符分隔），UTF‑8。
- 第一行是表头。
- `all_samples` 里有一些导出时残留的 `Unnamed:*` 列，可以丢弃。

### 4.1 多模态样本表头（16 列）
来自 `multimodal_train.tsv`：
```
author, clean_title, created_utc, domain, hasImage, id, image_url,
linked_submission_id, num_comments, score, subreddit, title,
upvote_ratio, 2_way_label, 3_way_label, 6_way_label
```

### 4.2 全量样本表头（20 列）
来自 `all_train.tsv`：
```
<empty>, Unnamed: 0, Unnamed: 0.1, Unnamed: 0.1.1,
author, clean_title, created_utc, domain, hasImage, id, image_url,
linked_submission_id, num_comments, score, subreddit, title,
upvote_ratio, 2_way_label, 3_way_label, 6_way_label
```
**说明：**开头空列与 `Unnamed:*` 列是导出索引的残留，可直接删除。

## 5) 字段解释（实际使用含义）
| 字段 | 含义 | 备注 |
|---|---|---|
| `author` | Reddit 用户名 | 可能为空/已删除。
| `clean_title` | 清洗后的标题 | 小写化、去标点/数字、去除暴露 subreddit 的词等（见论文）。
| `created_utc` | Unix 时间戳（秒） | UTC 时间。
| `domain` | 链接域名 | 如 `i.imgur.com`。
| `hasImage` | 是否有图 | `True/False`，多模态集应全为 True。
| `id` | Reddit 提交 ID | 唯一 ID。
| `image_url` | 图片预览 URL | `all_samples` 中可能为空。
| `linked_submission_id` | 关联提交 ID | 常为空。
| `num_comments` | 评论数 | 可能为空。
| `score` | Reddit 分数 | 整数。
| `subreddit` | 子版块名 | 用于标签分配。
| `title` | 原始标题 | 未清洗。
| `upvote_ratio` | 点赞比例 | 0–1。
| `2_way_label` | 二分类标签 | 见下文映射。
| `3_way_label` | 三分类标签 | 见下文映射。
| `6_way_label` | 六分类标签 | 见下文映射。

## 6) 标签含义（来自论文）
### 6.1 2‑way
- **True**：真实
- **Fake**：虚假

### 6.2 3‑way
- **True**
- **Fake with true text**：假新闻但文本是真实引用
- **Fake with false text**：假新闻且文本为假

### 6.3 6‑way
- **True**：内容准确
- **Satire/Parody**：讽刺/戏仿
- **Misleading Content**：刻意误导
- **Imposter Content**：冒充/伪装来源
- **False Connection**：标题/图片与内容不相符
- **Manipulated Content**：图像或内容被篡改

## 7) 数值标签映射（基于本地 TSV 严格验证）
论文只给出了类别解释与 subreddit 列表，没有提供数字编码。下面的数值映射来自对本地 TSV 的统计：同一 subreddit 的标签值在全体 split 中保持唯一，从而可和论文的 subreddit‑类别对应关系对齐。

### 7.1 2‑way 数值映射
| `2_way_label` | 含义 | 对应 subreddit |
|---|---|---|
| `1` | True | mildlyinteresting, neutralnews, nottheonion, photoshopbattles, pic, upliftingnews, usanews, usnews |
| `0` | Fake | confusing_perspective, fakealbumcovers, fakefacts, fakehistoryporn, misleadingthumbnails, pareidolia, propagandaposters, psbattle_artwork, satire, savedyouaclick, subredditsimulator, subsimulatorgpt2, theonion, waterfordwhispersnews |

### 7.2 3‑way 数值映射
| `3_way_label` | 含义 | 对应 subreddit |
|---|---|---|
| `0` | True | mildlyinteresting, neutralnews, nottheonion, photoshopbattles, pic, upliftingnews, usanews, usnews |
| `1` | Fake with true text | propagandaposters |
| `2` | Fake with false text | confusing_perspective, fakealbumcovers, fakefacts, fakehistoryporn, misleadingthumbnails, pareidolia, psbattle_artwork, satire, savedyouaclick, subredditsimulator, subsimulatorgpt2, theonion, waterfordwhispersnews |

### 7.3 6‑way 数值映射
| `6_way_label` | 含义 | 对应 subreddit |
|---|---|---|
| `0` | True | mildlyinteresting, neutralnews, nottheonion, photoshopbattles, pic, upliftingnews, usanews, usnews |
| `1` | Satire/Parody | fakealbumcovers, satire, theonion, waterfordwhispersnews |
| `2` | False Connection | confusing_perspective, fakehistoryporn, misleadingthumbnails, pareidolia |
| `3` | Imposter Content | subredditsimulator, subsimulatorgpt2 |
| `4` | Manipulated Content | psbattle_artwork |
| `5` | Misleading Content | fakefacts, propagandaposters, savedyouaclick |

**命名差异说明：**
- 论文中写的是 “photoshopbattles comments”，TSV 里对应的实际 subreddit 名为 `psbattle_artwork`。
- 论文中写的是 “confusing perspective”，TSV 中为 `confusing_perspective`。

## 8) 标签分配方式（为什么会有噪声）
论文明确说明：标签是按 subreddit 主题统一分配的，而不是逐条人工标注。因此存在一定噪声是合理的。

## 9) 文本清洗说明（影响 `clean_title`）
论文描述了清洗流程：
- 去标点、去数字
- 全部转小写
- 去除暴露 subreddit 的词（如 “PsBattle”、“colorized”）
- `savedyouaclick` 还有额外截断规则

## 10) 使用建议
- 如果要复现实验结果，请使用 `multimodal_only_samples`（官方 README 明确说明论文实验只用有图样本）。
- `all_samples` 里把空列和 `Unnamed:*` 列删掉。
- 文本模型优先用 `clean_title`。
- `image_url` 可能失效或为空，需要容错处理。

---

如果你需要，我可以再生成：
1) 自动清洗并加载 TSV 的 Python 脚本
2) 统计报告（类别分布、标题长度、图像缺失比例等）
