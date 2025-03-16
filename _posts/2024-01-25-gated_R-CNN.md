---
title: 
author: Patrick
date: 2024-01-25 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
## Gated Recurrent Convolution Neural Network
![](../Attatched/Pasted%20image%2020240130202300.png)
#### [GRCNN-BLSTM Model Paper](https://proceedings.neurips.cc/paper_files/paper/2017/file/c24cd76e1ce41366a4bbe8a49b02a028-Paper.pdf)
- OCR의 목표는 일반적인 이미지에서 문자를 인식하는 것
- 일반 이미지 분류에 대해 최근 제기된 RCNN 모델에 대해 GRCNN을 제안
- Gate는 Recurrent Convolution Layer의 문맥 모듈화를 제어하고, 순전(feed-forward), 순환(recurrent)정보의 균형을 제어
- BiLSTM(양방향 장단기 메모리)와의 결합을 통해 이미지에서의 문자 인식