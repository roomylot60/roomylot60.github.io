---
title: 
author: Patrick
date: 2024-07-22 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
Pytorch에서 데이터를 다룰 때 사용하는 모듈
- `torch.utils.data.Dataset`
- `torch.utils.data.DataLoader`

### Collate_fn
- Dataset이 고정된 길이가 아닐 경우, batch_size를 2 이상으로 dataloader를 호출 시 에러 발생
- Variable length data를 다룰 경우에 batch_size를 2 이상으로 하기 위해서 사용하는 함수
- Custom collate_fn()은 variable length input을 padding 작업을 거친 뒤, batch로 묶어서 dataloader로 넘겨주는 역할을 담당

```python
def my_collate_fn(samples):
	inputs = [sample['input'] for sample in samples]
    labels = [sample['label'] for sample in samples]
    padded_inputs = torch.nn.utils.rnn.pad_sequence(inputs, batch_first=True) #padding
    return {'input': padded_inputs.contiguous(),
            'label': torch.stack(labels).contiguous()}
```