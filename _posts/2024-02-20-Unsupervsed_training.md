---
title: 
author: Patrick
date: 2024-02-20 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
## Unsupervised Training
### Auto-Encoder
- 정답이 없는 x 데이터만으로 의미를 추출하기 위하여 x를 입력받아 다시 x를 복원할 수 있는 신경망을 학습
- 해당 입력 데이터의 의미있는 정보들이 쌓이고, 복원한 결과물; 예측값을 원본 데이터와 비교하며 근사치를 출력
- 입, 출력의 크기는 동일하나 중간으로 갈수록 신경망의 차원이 줄어들면서, 입력 특징이 압축되는 latent variable(잠재 변수)를 획득
- 주요 정보만이 압축된 값; 잠재 변수는 불필요한 정보에 대해 정보 손실이 발생
- 주로 복잡한 비선형 데이터의 차원을 줄이는 용도로 사용
#### Reconstructionn Loss
- 복원 오차를 손실값으로 사용하여 네트워크 학습에 사용
- Cross Entropy loss, L1, L2 loss 등을 사용