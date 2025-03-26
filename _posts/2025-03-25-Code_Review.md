---
title: MoeGoe TTS Code Review
author: Patrick
date: 2025-03-25 21:15:00 +0900
tags: [PRQST, Librosa]
categories: [Algorithm, Concept]
render_with_liquid: false
---
## MoeGoe Project
### 구성
MoeGoe/
utils.py
transforms.py
mel_processing.py

### utils.py
- `HParams Class`: 하이퍼파라미터를 관리하며, 딕셔너리 형태의 데이터를 객체 속성으로 변환하여 모델의 설정을 체계적으로 관리
- `load_checkpoint` 함수: 저장된 체크포인트 파일에서 모델의 가중치와 학습 상태를 불러와 모델에 로드하고, 이를 통해 이전 학습 상태를 복원하거나, 사전 학습된 가중치를 활용하여 학습을 이어서 진행 가능
- `get_hparams_from_file` 함수: JSON 형식의 설정 파일에서 하이퍼파라미터를 읽어와 HParams 객체로 변환해 모델의 설정을 파일로부터 쉽게 불러와 적용 가능
- `load_audio_to_torch` 함수: 지정된 오디오 파일을 로드하고, 목표 샘플링 레이트로 변환한 후, PyTorch의 텐서로 변환하는데, 이는 오디오 데이터를 모델의 입력으로 사용하기 위한 전처리 과정

---

### transforms.py
- PyTorch를 활용하여 Rational Quadratic Spline 변환을 구현한 것입니다.
- 이는 복잡한 분포를 모델링하거나 정규화 흐름(normalizing flows)에서 비선형 변환을 수행하는 데 사용됩니다.

#### 구성 요소
1. 상수 정의
코드의 시작 부분에서 세 개의 상수를 정의:​ 스플라인의 최소 구간 너비, 높이, 그리고 도함수 값을 설정하여 수치적 안정성을 보장

```python
DEFAULT_MIN_BIN_WIDTH = 1e-3
DEFAULT_MIN_BIN_HEIGHT = 1e-3
DEFAULT_MIN_DERIVATIVE = 1e-3
```

2. piecewise_rational_quadratic_transform 함수
- 입력 데이터에 대해 piecewise rational quadratic spline 변환을 적용
- 주요 매개변수:​
    - `inputs`: 변환할 입력 데이터.​
    - `unnormalized_widths`, `unnormalized_heights`, `unnormalized_derivatives`: 스플라인의 구간 너비, 높이, 도함수에 대한 미정규화된 값들
    - `inverse`: 변환의 역함수를 적용할지 여부를 결정. 기본값은 False
    - `tails` 및 `tail_bound`: 입력 값이 정의된 구간을 벗어날 경우의 처리 방식을 지정
    - `min_bin_width`, `min_bin_height`, `min_derivative`: 각 구간의 최소 너비, 높이, 도함수 값을 설정
- 이 함수는 내부적으로 rational_quadratic_spline 또는 unconstrained_rational_quadratic_spline 함수를 호출하여 변환을 수행

3. searchsorted 함수
- 주어진 `bin_locations`에서 `inputs`가 속하는 구간의 인덱스를 탐색
- `NumPy`의 `searchsorted`와 유사한 기능(`eps`는 수치적 안정성을 위한 작은 값)​

4. unconstrained_rational_quadratic_spline 함수
- 입력 값이 특정 구간을 벗어날 경우에도 적용 가능한 유리 이차 스플라인 변환을 수행
- `tails='linear'`로 설정되어 있으면, 구간 외부에서는 선형 변환을 적용
- 내부적으로 rational_quadratic_spline 함수를 호출하여 구간 내의 변환을 수행

5. rational_quadratic_spline 함수
- 실제 유리 이차 스플라인 변환을 수행
    - 입력 검증: 입력 값이 정의된 구간 [left, right] 내에 있는지 확인
    - 구간 너비 및 높이 계산: `unnormalized_widths`와 `unnormalized_heights`를 소프트맥스 함수를 통해 정규화하고, 최소 너비와 높이를 고려하여 실제 구간 너비와 높이를 계산​
    - 누적 너비 및 높이 계산: 각 구간의 누적 너비(cumwidths)와 높이(cumheights)를 계산하여, 입력 값이 속하는 구간을 찾는 데 사용
    - 도함수 계산: `unnormalized_derivatives`를 소프트플러스 함수를 통해 양수로 변환하여 각 구간의 도함수를 계산
    - 입력 값이 속하는 구간 찾기: searchsorted 함수를 사용하여 각 입력 값이 속하는 구간의 인덱스를 탐색​
    - 변환 수행: 각 구간에 대해 유리 이차 함수의 계수를 계산하고, 이를 사용하여 입력 값을 변환하며 inverse 매개변수에 따라 순방향 또는 역방향 변환을 수행
    - 자코비안 행렬식의 로그 계산: 변환의 자코비안 행렬식의 절댓값의 로그를 계산하여, 확률 분포의 밀도 추정 등에서 사용

#### 전반적인 역할
- 조각별 유리 이차 스플라인을 사용하여 비선형 변환을 수행하는 기능을 제공
- 정규화 흐름과 같은 확률 모델에서 복잡한 분포를 효과적으로 모델링하는 데 활용 가능
- 특히, 역변환이 명시적으로 정의되어 있어 샘플링과 밀도 추정 모두에서 유용하게 사용 가능

#### Rational Quadratic Spline 변환 
- 복잡한 데이터 분포를 모델링하기 위해 사용되는 비선형 변환 기법
- 이 방법은 구간별로 정의된 유리 이차 함수(rational quadratic functions)를 활용하여 입력 공간을 변환하며, 특히 정규화 흐름(normalizing flows)과 같은 생성 모델에서 밀도 추정과 샘플링 효율성을 높이는 데 활용
- 주요 특징 및 기능:
    - 유연한 비선형 변환: Rational Quadratic Spline은 각 구간(knots) 사이에서 유리 이차 함수를 사용하여 데이터를 변환. 이를 통해 단순한 선형 변환보다 훨씬 복잡한 데이터 구조를 효과적으로 캡처 가능​
    - 역변환의 효율성: 이 변환은 역함수가 명시적으로 정의되어 있어, 샘플링과 밀도 추정 모두에서 효율적인 계산이 가능하며 이는 정규화 흐름과 같은 모델에서 중요한 특성
    - 정확한 밀도 추정: 복잡한 분포를 정확하게 모델링할 수 있어, 다양한 데이터 분포에 대한 밀도 추정의 정확도를 향상
- 적용 분야:
    - 정규화 흐름(Normalizing Flows): 복잡한 분포를 학습하고 샘플링하는 데 사용되며, 이미지 생성, 음성 합성 등 다양한 생성 모델에 적용​
    - 밀도 추정 및 변분 추론(Variational Inference): 데이터의 복잡한 분포를 정확하게 추정하여, 베이지안 추론 등에서 활용

---

### mel_processing.py
- PyTorch와 Librosa를 활용하여 오디오 신호를 처리하고 멜 스펙트로그램을 생성하는 함수들을 정의

#### 1. 상수 및 전역 변수 정의
- `MAX_WAV_VALUE = 32768.0`: 오디오 샘플의 최대 진폭 값을 정의(16비트 PCM 오디오의 최대값​)
- 전역 딕셔너리 `mel_basis` 및 `hann_window`: 멜 필터 뱅크와 해닝 윈도우를 저장하여, 동일한 파라미터의 재계산을 방지하고 성능을 향상

#### 2. 동적 범위 압축 및 복원 함수
- 함수명: `dynamic_range_compression_torch(x, C=1, clip_val=1e-5)`
- 기능: 입력 신호 x에 대해 동적 범위 압축(dynamic range compression)을 수행하여 신호의 진폭 범위를 줄여서 작은 신호를 증폭시키고 큰 신호를 억제하여 전체적인 신호의 균형을 조절
- 구현: x의 값을 clip_val 이상으로 제한한 후, C 값을 곱한 값의 로그값
- 참고: 동적 범위 압축은 오디오 신호 처리에서 흔히 사용되며, 소리의 일관된 볼륨을 유지

- 함수명: `dynamic_range_decompression_torch(x, C=1)`:
- 기능: 압축된 신호를 복원
- 구현: 입력 x에 대해 지수 함수를 적용하고 C로 나눈 값​

- 함수명: `spectral_normalize_torch`(magnitudes) 및 `spectral_de_normalize_torch`(magnitudes):
- 기능: 위의 압축 및 복원 함수를 활용하여 구현하며 스펙트럼의 진폭을 정규화 및 복원

#### 3. 스펙트로그램 생성 함수
- 함수명: `spectrogram_torch(y, n_fft, sampling_rate, hop_size, win_size, center=False)`
- 기능: 입력 신호 y에 대해 스펙트로그램을 계산
- 구현:
    - 해닝 윈도우 준비: win_size와 데이터 타입, 디바이스 정보를 기반으로 전역 딕셔너리 hann_window에서 윈도우를 가져오거나 새로 생성
    - 패딩: 신호 y에 대해 n_fft와 hop_size를 기반으로 반사 패딩을 적용
    - STFT 적용: 패딩된 신호에 대해 단시간 푸리에 변환(STFT)
    - 진폭 스펙트럼 계산: STFT 결과의 실수부와 허수부를 사용하여 진폭 스펙트럼을 계산

#### 4. 멜 스펙트로그램 변환 함수
- 함수명: `spec_to_mel_torch(spec, n_fft, num_mels, sampling_rate, fmin, fmax)`
- 기능: 스펙트로그램 spec을 멜 스펙트로그램으로 변환
- 구현:
    멜 필터 뱅크 준비: n_fft, num_mels, sampling_rate, fmin, fmax를 기반으로 전역 딕셔너리 mel_basis에서 멜 필터를 가져오거나 새로 생성
    행렬 곱셈: 멜 필터와 스펙트로그램을 곱하여 멜 스펙트로그램을 생성
    정규화: 생성된 멜 스펙트로그램에 대해 스펙트럼 정규화를 수행

- 함수명: `mel_spectrogram_torch(y, n_fft, num_mels, sampling_rate, hop_size, win_size, fmin, fmax, center=False)`
- 기능: 입력 신호 y에 대해 멜 스펙트로그램을 직접 계산
- 구현:
    - 신호 범위 확인: y의 최소값과 최대값이 -1과 1 사이에 있는지 확인
    - 멜 필터 뱅크 및 해닝 윈도우 준비: 전역 딕셔너리에서 필요한 필터와 윈도우를 가져오거나 생성
    - 패딩 및 STFT 적용: 신호에 패딩을 적용하고 STFT를 수행하여 스펙트로그램을 도출
    - 멜 변환 및 정규화: 스펙트로그램을 멜 스펙트로그램으로 변환하고 정규화