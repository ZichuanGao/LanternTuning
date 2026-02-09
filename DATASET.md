# Fakeddit v2.0 (local copy) dataset guide

This document explains the dataset bundled in this repo under `Fakeddit datasetv2.0/` and how to interpret its fields and labels. It is based on the paper **“r/Fakeddit: Fine‑grained Fake News Detection”** (`/Users/gaozichuan/Downloads/1911.03854v2.pdf`) plus inspection of the TSV files in this repo.

## 1) What is this dataset?
Fakeddit is a large-scale Reddit dataset for fake news detection. It is **multimodal** (text + image + metadata) and provides three label granularities per sample:
- **2‑way**: true vs. fake
- **3‑way**: true vs. two kinds of fake (see below)
- **6‑way**: true + 5 types of fake

The labels are assigned **by subreddit theme** (distant supervision). The authors filter and manually check subreddits for topical consistency and then assign each subreddit a fixed 2/3/6‑way label.

## 2) Directory layout
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

- **all_samples**: includes both multimodal and text‑only posts.
- **multimodal_only_samples**: only posts that have images (multimodal).

## 3) Split sizes (as stored here)
> Counts below are **samples**, not including the header row.

**All samples**
| Split | File | Samples |
|---|---|---:|
| Train | `Fakeddit datasetv2.0/all_samples (also includes non multimodal)/all_train.tsv` | 878,570 |
| Validate | `Fakeddit datasetv2.0/all_samples (also includes non multimodal)/all_validate.tsv` | 92,482 |
| Test (public) | `Fakeddit datasetv2.0/all_samples (also includes non multimodal)/all_test_public.tsv` | 92,484 |

**Multimodal only**
| Split | File | Samples |
|---|---|---:|
| Train | `Fakeddit datasetv2.0/multimodal_only_samples/multimodal_train.tsv` | 564,211 |
| Validate | `Fakeddit datasetv2.0/multimodal_only_samples/multimodal_validate.tsv` | 59,364 |
| Test (public) | `Fakeddit datasetv2.0/multimodal_only_samples/multimodal_test_public.tsv` | 59,338 |

## 4) File format
- TSV (tab‑separated values), UTF‑8.
- First row is the header.
- `all_samples` includes extra index-like columns (`Unnamed: ...`) that can be dropped.

### 4.1 Multimodal headers (16 columns)
From `multimodal_train.tsv`:
```
author, clean_title, created_utc, domain, hasImage, id, image_url,
linked_submission_id, num_comments, score, subreddit, title,
upvote_ratio, 2_way_label, 3_way_label, 6_way_label
```

### 4.2 All-samples headers (20 columns)
From `all_train.tsv`:
```
<empty>, Unnamed: 0, Unnamed: 0.1, Unnamed: 0.1.1,
author, clean_title, created_utc, domain, hasImage, id, image_url,
linked_submission_id, num_comments, score, subreddit, title,
upvote_ratio, 2_way_label, 3_way_label, 6_way_label
```
**Note:** The leading empty column and the `Unnamed:*` columns are artifacts from exporting index columns. You can safely drop them.

## 5) Column meanings (practical interpretation)
| Column | Meaning | Notes |
|---|---|---|
| `author` | Reddit username | May be empty if deleted.
| `clean_title` | Cleaned title text | Lowercased, punctuation/numbers removed, and subreddit‑revealing words removed (per paper). For `savedyouaclick`, text after a specific delimiter is removed.
| `created_utc` | Unix timestamp (seconds) | Post creation time in UTC.
| `domain` | Link domain | e.g., `i.imgur.com`.
| `hasImage` | Boolean | `True/False`. In multimodal splits it should always be `True`.
| `id` | Reddit submission ID | Unique per submission.
| `image_url` | Image preview URL | May be empty in `all_samples` when `hasImage=False`.
| `linked_submission_id` | Linked submission ID | Often empty.
| `num_comments` | Number of comments | Integer or empty.
| `score` | Reddit score | Integer.
| `subreddit` | Subreddit name | Used for label assignment.
| `title` | Original title text | Raw (not cleaned).
| `upvote_ratio` | Upvote ratio | 0–1.
| `2_way_label` | Binary label | See mapping below.
| `3_way_label` | 3‑class label | See mapping below.
| `6_way_label` | 6‑class label | See mapping below.

## 6) Label definitions (from the paper)
The dataset provides three label granularities per sample:

### 6.1 2‑way label
- **True** vs **Fake**.

### 6.2 3‑way label
- **True**
- **Fake with true text** (e.g., fake post that quotes true text)
- **Fake with false text**

### 6.3 6‑way label
The 6‑way labels are **True + 5 types of fake** (adapted from Wardle, 2017):
- **True**: content is accurate.
- **Satire/Parody**: satirical or intentionally humorous distortion.
- **Misleading Content**: information intentionally structured to mislead.
- **Imposter Content**: content that impersonates or imitates real sources.
- **False Connection**: title/visuals do not match or support the content.
- **Manipulated Content**: altered images or content.

## 7) Numeric label mapping (derived from this repo’s TSVs)
The paper defines label meanings but **does not specify numeric IDs**. Below is the **numeric mapping inferred from `multimodal_train.tsv` in this repo** by grouping labels by subreddit.

### 7.1 2‑way mapping
| `2_way_label` | Meaning | Subreddits in this repo |
|---|---|---|
| `1` | True | mildlyinteresting, neutralnews, nottheonion, photoshopbattles, pic, upliftingnews, usanews, usnews |
| `0` | Fake | confusing_perspective, fakealbumcovers, fakefacts, fakehistoryporn, misleadingthumbnails, pareidolia, propagandaposters, psbattle_artwork, satire, savedyouaclick, subredditsimulator, subsimulatorgpt2, theonion, waterfordwhispersnews |

### 7.2 3‑way mapping
| `3_way_label` | Meaning | Subreddits in this repo |
|---|---|---|
| `0` | True | mildlyinteresting, neutralnews, nottheonion, photoshopbattles, pic, upliftingnews, usanews, usnews |
| `1` | Fake with true text | propagandaposters |
| `2` | Fake with false text | confusing_perspective, fakealbumcovers, fakefacts, fakehistoryporn, misleadingthumbnails, pareidolia, psbattle_artwork, satire, savedyouaclick, subredditsimulator, subsimulatorgpt2, theonion, waterfordwhispersnews |

### 7.3 6‑way mapping
| `6_way_label` | Meaning | Subreddits in this repo |
|---|---|---|
| `0` | True | mildlyinteresting, neutralnews, nottheonion, photoshopbattles, pic, upliftingnews, usanews, usnews |
| `1` | Satire/Parody | fakealbumcovers, satire, theonion, waterfordwhispersnews |
| `2` | False Connection | confusing_perspective, fakehistoryporn, misleadingthumbnails, pareidolia |
| `3` | Imposter Content | subredditsimulator, subsimulatorgpt2 |
| `4` | Manipulated Content | psbattle_artwork |
| `5` | Misleading Content | fakefacts, propagandaposters, savedyouaclick |

**Note on subreddit name differences:**
- The paper lists **“photoshopbattles comments”** for Manipulated Content; the TSVs use `psbattle_artwork` for that category.
- The paper lists **“confusing perspective”**; the TSVs use `confusing_perspective`.

## 8) How labels are assigned (important for interpretation)
The paper uses **distant supervision**:
- Each subreddit is mapped to a label category.
- Every post from that subreddit inherits the same 2/3/6‑way labels.
- Quality control steps include filtering low‑score posts (score < 1) and manually checking random samples from each subreddit to validate topical consistency.

This means labels are **subreddit‑level**, not manual per‑post annotations. Expect some noise at the sample level.

## 9) Text cleaning (affects `clean_title`)
The paper states that title text is cleaned by:
- removing punctuation and numbers,
- lowercasing all text,
- removing words that reveal the subreddit (e.g., “PsBattle”, “colorized”),
- applying a special rule for `savedyouaclick` to remove text after a specific delimiter.

If you need exact preprocessing, rely on `clean_title` rather than reproducing it.

## 10) Practical tips
- If you want to compare with the paper’s reported results, use `multimodal_only_samples` (the official README says the paper uses multimodal samples only).
- Prefer `multimodal_only_samples` if you need paired text‑image inputs.
- For `all_samples`, drop the empty and `Unnamed:*` columns.
- Use `clean_title` for text‑only baselines; use `title` if you want raw text.
- `image_url` is a preview URL; you may need to handle missing or expired links.
- `hasImage` is a string in TSV; coerce to boolean in your loader.

---

If you want, I can add a small loader script (Python) that reads these TSVs, drops junk columns, and returns structured samples for training.
