---
title: 
author: Patrick
date: 2024-03-02 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
## Custom Dataset

- Pytorch에서는 데이터셋을 좀 더 쉽게 다룰 수 있도록 유용한 도구로서 `torch.utils.data.Dataset`과 `torch.utils.data.DataLoader`를 제공
- 이를 사용하면 미니 배치 학습, 데이터 셔플(shuffle), 병렬 처리까지 간단히 수행 가능
- 기본적인 사용 방법은 Dataset을 정의하고, 이를 DataLoader에 전달
- `torch.utils.data.Dataset`을 상속받아 직접 커스텀 데이터셋(Custom Dataset)을 제작
- `torch.utils.data.Dataset`은 Pytorch에서 데이터셋을 제공하는 Abstract(추상) 클래스

```python
from torch.utils.data import Dataset

class CustomDataset(Dataset): 
  def __init__(self): # 데이터셋의 전처리를 해주는 부분

  def __len__(self): # 데이터셋의 길이. 즉, 총 샘플의 수를 적어주는 부분

  def __getitem__(self, idx): # 데이터셋에서 특정 1개의 샘플을 가져오는 함수
```

- 커스텀 데이터셋을 만들 때, 일단 가장 필요한 define : `__init__()`, `__len__()`, `__getitem__()`
	- `len(dataset)` : Dataset의 크기를 return
	- `__getitem__(self, idx)` : `dataset[i]` i번째 샘플을 가져올 때 사용하는 인덱싱