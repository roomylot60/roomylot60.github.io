---
title: Model evaluation metric
author: Patrick
date: 2024-03-22 13:18:00 +0900
tags: [BELU, ROUGE, Perplexity]
categories: [NLP, Metric]
render_with_liquid: false
---
## 모델 평가 지표 개요

|지표|핵심 목적|주요 적용 분야|
|-----|----------|----------|
|BLEU|정밀도 기반 n-gram 매칭|기계 번역, 챗봇|
|ROUGE|재현율 기반 n-gram 매칭|텍스트 요약, 대화 응답|
|Perplexity|언어 모델의 문장 생성 능력 측정|언어 모델링, 챗봇|
|Accuracy|전체 정답률|문장 분류, 이진 태스크|
|F1 Score|정밀도-재현율 조화 평균|감정 분석, QA 시스템|
|METEOR|어순, 동의어까지 고려한 n-gram 매칭|번역, 챗봇|
|CIDEr / SPICE|시각 설명 등 의미론적 매칭|이미지 캡셔닝|

### 1. BLEU (Bilingual Evaluation Understudy)
- 기계 번역 품질을 평가하기 위해 고안된 정밀도 중심의 지표
- n-gram 정확도를 측정 (예: unigram, bigram, trigram 등)
- 공식 : \\( BLEU = BP⋅exp \sum_{n=1}^{N} omega_{n} log p_{n} \\)
    - \\( BP \\): brevity penalty (짧은 번역 패널티)
    - \\( p_{n} \\): n-gram precision
- 해석 : 1.0 (100%)에 가까울수록 좋고 짧은 문장에서 낮게 나올 수 있음

### 2. ROUGE (Recall-Oriented Understudy for Gisting Evaluation)
- 요약 평가용으로 고안됨
- 중요 단어가 얼마나 많이 맞았는가 (재현율 기반)
- 종류:
    - ROUGE-1: 단어 매칭
    - ROUGE-2: bigram 매칭
    - ROUGE-L: longest common subsequence 기반
- 해석 : 1.0에 가까울수록 좋고 BLEU보다 자연어 응답 평가에 적합

### 3. Perplexity (혼란도)
- 언어 모델이 얼마나 “자연스럽게” 다음 단어를 예측했는가
- Cross-Entropy Loss를 기반으로 계산
- 공식 : \\( Perplexity = e^{Loss} \\)
- 해석 : 
    - 낮을수록 좋음 (→ 모델이 예측을 "자신 있게" 함)
    - 1.0에 가까우면 거의 완벽하게 맞췄다는 뜻
    - 너무 낮으면 과적합 가능성 있음

### 4. Accuracy, Precision, Recall, F1 Score
- 정답 여부가 명확한 분류 과제에 적합
- 챗봇 태스크보단 분류 모델, 감정 분석 등에서 사용

|지표|의미|
|----|----------|
|Accuracy|전체 중 맞춘 비율|
|Precision|맞다고 예측한 것 중 실제로 맞은 비율|
|Recall|실제 정답 중 맞췄던 비율|
|F1 Score|Precision과 Recall의 조화 평균|

### 5. METEOR (Metric for Evaluation of Translation with Explicit ORdering)
- BLEU 보완 지표
- 동의어/어순 등도 평가하며 더 인간 중심 평가
- 특징 :
    - BLEU보다 더 유연한 평가
    - WordNet 기반 매핑 포함 가능
    - 챗봇이나 번역 평가에 효과적

### 6. CIDEr / SPICE
- 주로 이미지 캡셔닝과 관련
- 문장과 문장의 의미론적 일치를 평가
- 챗봇에서도 "상황 설명" 중심이라면 사용 가능

✅ 비교 요약
지표	핵심 특징	사용 분야	설명
BLEU	정밀도 기반 n-gram	번역, 챗봇	단어 순서 엄격
ROUGE	재현율 기반 n-gram	요약, 대화	핵심 단어 중심
Perplexity	언어 모델 예측 정확도	언어 생성	낮을수록 좋음
METEOR	의미 기반 유사도	번역, 챗봇	어순·동의어까지 반영
Accuracy	전체 정답률	분류	이진/다중 클래스
F1 Score	P-R 균형	감정, QA	불균형 데이터 적합
✅ 챗봇에서 추천 조합
목적	추천 지표
일반 대화 챗봇	BLEU, ROUGE-L, Perplexity
의도 맞춤 응답 챗봇	BLEU + METEOR
QA / 정답형 챗봇	Accuracy, F1
요약·설명 중심 챗봇	ROUGE-1, ROUGE-L
