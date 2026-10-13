---
title: PyTorch Environment Setup
author: Patrick
date: 2024-02-19 13:18:00 +0900
tags: [Grammar]
categories: [PyTorch]
description: PyTorch 설치 명령어와 CUDA 버전에 맞는 PyTorch 버전을 확인·설치하는 방법을 정리합니다.
render_with_liquid: false
---
## Pytorch

```powershell
pip install pytorch
pip install torchvision
pip install torch torchvision torchaudio
```

- GPU 사용을 위해 cuda version 확인 필요
	- `nvcc --version` 명령어를 통해 cuda version이 11.3임을 확인
	- 다음 페이지에서 이전 버전의 pytorch를 다운 : https://pytorch.org/get-started/previous-versions/