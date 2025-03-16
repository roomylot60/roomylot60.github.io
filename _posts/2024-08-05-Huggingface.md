---
title: 
author: Patrick
date: 2024-08-05 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
# Datasets(feat. Hugging Face)

## 0. Log-in
```bash
# Create Repository
hugginface-cli login

huggingface-cli repo create my-cool-dataset --type dataset

huggingface-cli repo create my-cool-dataset --type dataset --organization your-org-name
```
---
## 1. Download

### load_dataset

```python
from datasets import load_dataset

dataset = load_dataset("dataset_name", split="train or test", trust_remote_code=True or False)
```

- load_dataset() 을 통해 불러온 데이터셋은 dict 형태
- [각 parameter에 대한 참조](https://huggingface.co/docs/datasets/v2.20.0/en/package_reference/loading_methods#datasets.load_dataset)
---
