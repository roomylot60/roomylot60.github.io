---
title: Transformer Algorithm
author: Patrick
date: 2023-12-20 18:18:00 +0900
tags: [Concept]
categories: [NLP]
render_with_liquid: false
---
## 1. Transformer Model
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/nlp/Pasted%20image%2020240105144425.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

- 2017년 Google에서 발표한 "Attention is all you need" 논문에서 제시한 모델
- **Attention Machanism** 만을 사용하여 seq2seq의 구조인 *encoder-decoder*로 구현
	- seq2seq : Encoder, Decoder에서 각각 하나의 RNN이 t 개의 시점을 가지는 구조
	- transformer : Encoder, Decoder 단위(Layer)가 N 개로 구성되는 구조
- RNN을 사용하지 않았음에도 성능적인 우수성
	- Positional information : RNN은 단어의 위치에 따라 단어를 순차적(Sequential) 처리를 하여 단어의 위치 정보를 보유
	- Positional Encoding : Transformer는 단어의 위치 정보(sin, cos)를 각 단어의 Embedding vector에 더하여 Model의 입력으로 사용
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/nlp/Pasted%20image%2020240104161019.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/nlp/Pasted%20image%2020240104162143.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

```python
import numpy as np
import matplotlib.pyplot as plt
import tensorflow as tf

class PositionalEncoding(tf.keras.layers.Layer):
	def __init__(self, position, d_model):
		super(PositionalEncoding, self).__init__()
		self.pos_encoding = self.positional_encoding(position, d_model)

	# position encoding에서 사용할 sin, cos의 각도 지정 함수
	def get_angles(self, position, i, d_model):
		angles = 1 / tf.pow(10000, (2 * (i // 2)) / tf.cast(d_model, tf.float32))
		return position * angles
	
	def positional_encoding(self, position, d_model):
		angle_rads = self.get_angles(
			position=tf.range(position, dtype=tf.float32)[:, tf.newaxis],
			i=tf.range(d_model, dtype=tf.float32)[tf.newaxis, :],
			d_model=d_model)
		
		# Even : 2i
		sines = tf.math.sin(angle_rads[:, 0::2])
		
		# Odd : 2i+1
		cosines = tf.math.cos(angle_rads[:, 1::2])
	
		angle_rads = np.zeros(angle_rads.shape) # 홀수, 짝수에 따라 나누어진 각도들에 대한 정보를 통합
		angle_rads[:, 0::2] = sines
		angle_rads[:, 1::2] = cosines
		pos_encoding = tf.constant(angle_rads)
		pos_encoding = pos_encoding[tf.newaxis, ...]
		print(pos_encoding.shape)
		
		return tf.cast(pos_encoding, tf.float32)

	def call(self, inputs):
		return inputs + self.pos_encoding[:, :tf.shape(inputs)[1], :]
```

### seq2seq Model의 문제
- Input sequence를 하나의 벡터(context vector)로 압축하는 과정에서 정보 손실 발생

### Hyper-parameter in Transformer
1. d_{model} : Encoder, Decoder, Embedding vector에서의 차원
2. num_layers : Layer(Encoder+Decoder)의 층 수
3. num_heads : Transformer에서 Attention을 사용할 때 분할 및 병렬 수행, 결과값을 통합하는 방식을 사용하는데, 이 때의 병렬 수
4. d_{ff} : Transformer 내부에 존재하는 Feed Forward Neural Network의 크기(이 때의 FFNN의 입출력층의 크기는 d_{model})

### Attention in TM
1. Encoder Self-Attention
2. Masked Decoder Self-Attention
3. Encoder-Decoder Attention

---

## 2. Encoder
### First sublayer : Multi-head Self-Attention
#### Self-Attention
- Attention : Query에 대해서, Key와의 유사도를 mapping된 각각의 Value에 반영, Value의 가중합을 return

```python
Q = Query # t 시점의 decoder cell에서의 hidden state = t 시점의 encoder에서의 context vector
K = Keys # 모든 시점의 encoder cell의 hidden states = dot product의 대상이 되는 set
V = Values # 모든 시점의 encoder cell의 hidden states = weight과 곱해지는 vector의 set
```

- Self-Attention : Q, K, V가 모두 입력 문장의 모든 단어 벡터들을 의미
	- d_{model}의 차원을 갖는 단어 벡터들을 num_heads로 나눈 값을 Q, K, V의 벡터의 차원으로 결정
- Scaled dot-product Attention : 내적만을 사용하는 Attention Function에 대해서 특정값 √n으로 나누어 scaling 하여 각 벡터의 모음; 행렬에 대해 연산하여 일괄 계산`score(q, k) = q · k / √n (n = d_{model} / num_heads = 각 Q, K, V의 차원값 d_{k})`
	- Padding Mask : Scaled dot-product attention에서 `<PAD>` token은 실질적인 의미를 갖지 않는 단어이므로, 이를 제외하기 위해 매우 작은 음수를 넣어 유사도에 반영되는 것을 막는 연산<br>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/nlp/Pasted%20image%2020240104175031.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/nlp/Pasted%20image%2020240104175125.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

```python
def scaled_dot_product_attention(query, key, value, mask):
  # query 크기 : (batch_size, num_heads, query의 문장 길이, d_model/num_heads)
  # key 크기 : (batch_size, num_heads, key의 문장 길이, d_model/num_heads)
  # value 크기 : (batch_size, num_heads, value의 문장 길이, d_model/num_heads)
  # padding_mask : (batch_size, 1, 1, key의 문장 길이)

  # Q와 K의 곱. 어텐션 스코어 행렬.
  matmul_qk = tf.matmul(query, key, transpose_b=True)

  # 스케일링
  # dk의 루트값으로 나눠준다.
  depth = tf.cast(tf.shape(key)[-1], tf.float32)
  logits = matmul_qk / tf.math.sqrt(depth)

  # 마스킹. 어텐션 스코어 행렬의 마스킹 할 위치에 매우 작은 음수값을 넣는다.
  # 매우 작은 값이므로 소프트맥스 함수를 지나면 행렬의 해당 위치의 값은 0이 된다.
  if mask is not None:
    logits += (mask * -1e9)

  # 소프트맥스 함수는 마지막 차원인 key의 문장 길이 방향으로 수행된다.
  # attention weight : (batch_size, num_heads, query의 문장 길이, key의 문장 길이)
  attention_weights = tf.nn.softmax(logits, axis=-1)

  # output : (batch_size, num_heads, query의 문장 길이, d_model/num_heads)
  output = tf.matmul(attention_weights, value)

  return output, attention_weights
```

- Multi-head Attention : Self Attention을 병렬적으로 사용하여 각각의 Attention head가 갖는 가중치 행렬이 다르게 설정;<br> 
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/nlp/Pasted%20image%2020240104180009.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

여러 시점으로 정보를 수집하여 sequence를 구성하는 단어들간의 연관도를 측정<br>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/nlp/Pasted%20image%2020240104180133.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

```python
# Multi-head Attention Implementation
# 각 Q, K, V를 만들기 위한 가중치 행렬과 결합 후에 사용하는 가중치 행렬 생성
# 구현 시에 입력을 전결합층; 밀집층(Dense layer)을 통과해 구현
Dense(units)

class MultiHeadAttention(tf.keras.layers.Layer):

  def __init__(self, d_model, num_heads, name="multi_head_attention"):
    super(MultiHeadAttention, self).__init__(name=name)
    self.num_heads = num_heads
    self.d_model = d_model

    assert d_model % self.num_heads == 0

    # d_model을 num_heads로 나눈 값.
    # 논문 기준 : 64
    self.depth = d_model // self.num_heads

    # WQ, WK, WV에 해당하는 밀집층 정의
    self.query_dense = tf.keras.layers.Dense(units=d_model)
    self.key_dense = tf.keras.layers.Dense(units=d_model)
    self.value_dense = tf.keras.layers.Dense(units=d_model)

    # WO에 해당하는 밀집층 정의
    self.dense = tf.keras.layers.Dense(units=d_model)

  # num_heads 개수만큼 q, k, v를 split하는 함수
  def split_heads(self, inputs, batch_size):
    inputs = tf.reshape(
        inputs, shape=(batch_size, -1, self.num_heads, self.depth))
    return tf.transpose(inputs, perm=[0, 2, 1, 3])

  def call(self, inputs):
    query, key, value, mask = inputs['query'], inputs['key'], inputs[
        'value'], inputs['mask']
    batch_size = tf.shape(query)[0]

    # 1. WQ, WK, WV에 해당하는 밀집층 지나기
    # q : (batch_size, query의 문장 길이, d_model)
    # k : (batch_size, key의 문장 길이, d_model)
    # v : (batch_size, value의 문장 길이, d_model)
    # 참고) 인코더(k, v)-디코더(q) 어텐션에서는 query 길이와 key, value의 길이는 다를 수 있다.
    query = self.query_dense(query)
    key = self.key_dense(key)
    value = self.value_dense(value)

    # 2. 헤드 나누기
    # q : (batch_size, num_heads, query의 문장 길이, d_model/num_heads)
    # k : (batch_size, num_heads, key의 문장 길이, d_model/num_heads)
    # v : (batch_size, num_heads, value의 문장 길이, d_model/num_heads)
    query = self.split_heads(query, batch_size)
    key = self.split_heads(key, batch_size)
    value = self.split_heads(value, batch_size)

    # 3. 스케일드 닷 프로덕트 어텐션. 앞서 구현한 함수 사용.
    # (batch_size, num_heads, query의 문장 길이, d_model/num_heads)
    scaled_attention, _ = scaled_dot_product_attention(query, key, value, mask)
    # (batch_size, query의 문장 길이, num_heads, d_model/num_heads)
    scaled_attention = tf.transpose(scaled_attention, perm=[0, 2, 1, 3])

    # 4. 헤드 연결(concatenate)하기
    # (batch_size, query의 문장 길이, d_model)
    concat_attention = tf.reshape(scaled_attention,
                                  (batch_size, -1, self.d_model))

    # 5. WO에 해당하는 밀집층 지나기
    # (batch_size, query의 문장 길이, d_model)
    outputs = self.dense(concat_attention)

    return outputs
```

### Second sublayer : FFNN
- Residual connection(잔차 연결) : Sublayer의 입력과 출력을 합(같은 차원을 가지므로 가능)을 구하는 과정 [입력과 출력의 합과 모델의 학습 관련 논문](https://arxiv.org/pdf/1512.03385.pdf)
- Layer Normalization(층 정규화) : Tensor의 마지막 차원; d_{model} 차원에 대해서 평균과 분산을 구하고, 이를 가지고 정규화하여 학습에 활용 [층 정규화 논문](https://arxiv.org/pdf/1607.06450.pdf)

#### Encoder layer code

```python
def encoder_layer(dff, d_model, num_heads, dropout, name="encoder_layer"):
  inputs = tf.keras.Input(shape=(None, d_model), name="inputs")

  # 인코더는 패딩 마스크 사용
  padding_mask = tf.keras.Input(shape=(1, 1, None), name="padding_mask")

  # 멀티-헤드 어텐션 (첫번째 서브층 / 셀프 어텐션)
  attention = MultiHeadAttention(
      d_model, num_heads, name="attention")({
          'query': inputs, 'key': inputs, 'value': inputs, # Q = K = V
          'mask': padding_mask # 패딩 마스크 사용
      })

  # 드롭아웃 + 잔차 연결과 층 정규화
  attention = tf.keras.layers.Dropout(rate=dropout)(attention)
  attention = tf.keras.layers.LayerNormalization(
      epsilon=1e-6)(inputs + attention)

  # 포지션 와이즈 피드 포워드 신경망 (두번째 서브층)
  outputs = tf.keras.layers.Dense(units=dff, activation='relu')(attention)
  outputs = tf.keras.layers.Dense(units=d_model)(outputs)

  # 드롭아웃 + 잔차 연결과 층 정규화
  outputs = tf.keras.layers.Dropout(rate=dropout)(outputs)
  outputs = tf.keras.layers.LayerNormalization(
      epsilon=1e-6)(attention + outputs)

  return tf.keras.Model(
      inputs=[inputs, padding_mask], outputs=outputs, name=name)
```

### Encoder Code

```python
def encoder(vocab_size, num_layers, dff,
            d_model, num_heads, dropout,
            name="encoder"):
  inputs = tf.keras.Input(shape=(None,), name="inputs")

  # 인코더는 패딩 마스크 사용
  padding_mask = tf.keras.Input(shape=(1, 1, None), name="padding_mask")

  # 포지셔널 인코딩 + 드롭아웃
  embeddings = tf.keras.layers.Embedding(vocab_size, d_model)(inputs)
  embeddings *= tf.math.sqrt(tf.cast(d_model, tf.float32))
  embeddings = PositionalEncoding(vocab_size, d_model)(embeddings)
  outputs = tf.keras.layers.Dropout(rate=dropout)(embeddings)

  # 인코더를 num_layers개 쌓기
  for i in range(num_layers):
    outputs = encoder_layer(dff=dff, d_model=d_model, num_heads=num_heads,
        dropout=dropout, name="encoder_layer_{}".format(i),
    )([outputs, padding_mask])

  return tf.keras.Model(
      inputs=[inputs, padding_mask], outputs=outputs, name=name)
```

---

## 3. Decoder
### First sublayer : Masked Multi-head Self-Attention
- 기본적으로 Transformer에서는 Multi-head Attention을 수행하고, 내부에서 Scaled dot-product Attention Function을 호출하여 Masking을 적용
	1. **Encoders**' Self-Attention : Padding Mask
	2. **Decoders**' Masked Self-Attention : Look-ahead Mask
	3. **Decoders**' Encoder-Decoder Attention : Padding Mask

#### Look-ahead Mask
- Input : Embedding layer + Positional Encoding(with Teacher Forcing)
- 번역할 문장을 문장 행렬로 입력 받을 때, RNN과는 달리 *한꺼번에* 입력되어 현재 시점의 단어를 예측하는 데에 미래 시점의 단어까지 참고
- Look-ahead Mask : 현재 시점보다 미래에 있는 단어를 참고하지 못하도록 하는 역할
- Masked Self-Attention : Self-Attention을 통해 Attention score를 얻고, Scaled dot-product Attention Function에 Look-ahead mask를 전달 후 Masking을 실시; 자기 자신과 이전 단어들 만을 참고할 수 있음

```python
# 디코더의 첫번째 서브층(sublayer)에서 미래 토큰을 Mask하는 함수 
def create_look_ahead_mask(x): 
	seq_len = tf.shape(x)[1]
	look_ahead_mask = 1 - tf.linalg.band_part(tf.ones((seq_len, seq_len)), -1, 0)
	padding_mask = create_padding_mask(x) # 패딩 마스크도 포함
	return tf.maximum(look_ahead_mask, padding_mask)
```

### Second sublayer : Encoder-Decoder Attention
#### Q : Decoder, K = V : Encoder
- Q, K, V의 출처가 같을 경우 Self-Attention이라고 함
- Query는 Decoder의 1st sublayer로부터, Key, Value는 Encoder의 마지막 layer로부터 받음

#### Decode layer code

```python
def decoder_layer(dff, d_model, num_heads, dropout, name="decoder_layer"):
  inputs = tf.keras.Input(shape=(None, d_model), name="inputs")
  enc_outputs = tf.keras.Input(shape=(None, d_model), name="encoder_outputs")

  # 룩어헤드 마스크(첫번째 서브층)
  look_ahead_mask = tf.keras.Input(
      shape=(1, None, None), name="look_ahead_mask")

  # 패딩 마스크(두번째 서브층)
  padding_mask = tf.keras.Input(shape=(1, 1, None), name='padding_mask')

  # 멀티-헤드 어텐션 (첫번째 서브층 / 마스크드 셀프 어텐션)
  attention1 = MultiHeadAttention(
      d_model, num_heads, name="attention_1")(inputs={
          'query': inputs, 'key': inputs, 'value': inputs, # Q = K = V
          'mask': look_ahead_mask # 룩어헤드 마스크
      })

  # 잔차 연결과 층 정규화
  attention1 = tf.keras.layers.LayerNormalization(
      epsilon=1e-6)(attention1 + inputs)

  # 멀티-헤드 어텐션 (두번째 서브층 / 디코더-인코더 어텐션)
  attention2 = MultiHeadAttention(
      d_model, num_heads, name="attention_2")(inputs={
          'query': attention1, 'key': enc_outputs, 'value': enc_outputs, # Q != K = V
          'mask': padding_mask # 패딩 마스크
      })

  # 드롭아웃 + 잔차 연결과 층 정규화
  attention2 = tf.keras.layers.Dropout(rate=dropout)(attention2)
  attention2 = tf.keras.layers.LayerNormalization(
      epsilon=1e-6)(attention2 + attention1)

  # 포지션 와이즈 피드 포워드 신경망 (세번째 서브층)
  outputs = tf.keras.layers.Dense(units=dff, activation='relu')(attention2)
  outputs = tf.keras.layers.Dense(units=d_model)(outputs)

  # 드롭아웃 + 잔차 연결과 층 정규화
  outputs = tf.keras.layers.Dropout(rate=dropout)(outputs)
  outputs = tf.keras.layers.LayerNormalization(
      epsilon=1e-6)(outputs + attention2)

  return tf.keras.Model(
      inputs=[inputs, enc_outputs, look_ahead_mask, padding_mask],
      outputs=outputs,
      name=name)
```

### Third sublayer : FFNN
### Decoder Code

```python
def decoder(vocab_size, num_layers, dff,
            d_model, num_heads, dropout,
            name='decoder'):
  inputs = tf.keras.Input(shape=(None,), name='inputs')
  enc_outputs = tf.keras.Input(shape=(None, d_model), name='encoder_outputs')

  # 디코더는 룩어헤드 마스크(첫번째 서브층)와 패딩 마스크(두번째 서브층) 둘 다 사용.
  look_ahead_mask = tf.keras.Input(
      shape=(1, None, None), name='look_ahead_mask')
  padding_mask = tf.keras.Input(shape=(1, 1, None), name='padding_mask')

  # 포지셔널 인코딩 + 드롭아웃
  embeddings = tf.keras.layers.Embedding(vocab_size, d_model)(inputs)
  embeddings *= tf.math.sqrt(tf.cast(d_model, tf.float32))
  embeddings = PositionalEncoding(vocab_size, d_model)(embeddings)
  outputs = tf.keras.layers.Dropout(rate=dropout)(embeddings)

  # 디코더를 num_layers개 쌓기
  for i in range(num_layers):
    outputs = decoder_layer(dff=dff, d_model=d_model, num_heads=num_heads,
        dropout=dropout, name='decoder_layer_{}'.format(i),
    )(inputs=[outputs, enc_outputs, look_ahead_mask, padding_mask])

  return tf.keras.Model(
      inputs=[inputs, enc_outputs, look_ahead_mask, padding_mask],
      outputs=outputs,
      name=name)
```

---

## 4. Position-wise FFNN
- Encoder, Decoder에서 공통적으로 가지고 있는 FFNN형태의 sublayer

## 5. Transformer Implementation
### Transformer Code

```python
def transformer(vocab_size, num_layers, dff,
                d_model, num_heads, dropout,
                name="transformer"):

  # 인코더의 입력
  inputs = tf.keras.Input(shape=(None,), name="inputs")

  # 디코더의 입력
  dec_inputs = tf.keras.Input(shape=(None,), name="dec_inputs")

  # 인코더의 패딩 마스크
  enc_padding_mask = tf.keras.layers.Lambda(
      create_padding_mask, output_shape=(1, 1, None),
      name='enc_padding_mask')(inputs)

  # 디코더의 룩어헤드 마스크(첫번째 서브층)
  look_ahead_mask = tf.keras.layers.Lambda(
      create_look_ahead_mask, output_shape=(1, None, None),
      name='look_ahead_mask')(dec_inputs)

  # 디코더의 패딩 마스크(두번째 서브층)
  dec_padding_mask = tf.keras.layers.Lambda(
      create_padding_mask, output_shape=(1, 1, None),
      name='dec_padding_mask')(inputs)

  # 인코더의 출력은 enc_outputs. 디코더로 전달된다.
  enc_outputs = encoder(vocab_size=vocab_size, num_layers=num_layers, dff=dff,
      d_model=d_model, num_heads=num_heads, dropout=dropout,
  )(inputs=[inputs, enc_padding_mask]) # 인코더의 입력은 입력 문장과 패딩 마스크

  # 디코더의 출력은 dec_outputs. 출력층으로 전달된다.
  dec_outputs = decoder(vocab_size=vocab_size, num_layers=num_layers, dff=dff,
      d_model=d_model, num_heads=num_heads, dropout=dropout,
  )(inputs=[dec_inputs, enc_outputs, look_ahead_mask, dec_padding_mask])

  # 다음 단어 예측을 위한 출력층
  outputs = tf.keras.layers.Dense(units=vocab_size, name="outputs")(dec_outputs)

  return tf.keras.Model(inputs=[inputs, dec_inputs], outputs=outputs, name=name)
```

### Hyper-parameter Setting

```python
small_transformer = transformer(
    vocab_size = 9000,
    num_layers = 4,
    dff = 512,
    d_model = 128,
    num_heads = 4,
    dropout = 0.3,
    name="small_transformer")

tf.keras.utils.plot_model(
    small_transformer, to_file='small_transformer.png', show_shapes=True)
```

### Loss Function

```python
def loss_function(y_true, y_pred):
  y_true = tf.reshape(y_true, shape=(-1, MAX_LENGTH - 1))

  loss = tf.keras.losses.SparseCategoricalCrossentropy(
      from_logits=True, reduction='none')(y_true, y_pred)

  mask = tf.cast(tf.not_equal(y_true, 0), tf.float32)
  loss = tf.multiply(loss, mask)

  return tf.reduce_mean(loss)
```

### Learning rate
#### Learning rate Scheduler
- 미리 학습 일정을 정해두고 그 일정에 따라 학습률이 조정되는 방법
- Transformer의 경우, 사용자가 정한 단계까지는 학습률을 증가시키고, 일정 단계에 이르면 학습률을 점차적으로 하강하여 적용

#### Cumstom Scheduler Code

```python
class CustomSchedule(tf.keras.optimizers.schedules.LearningRateSchedule):

  def __init__(self, d_model, warmup_steps=4000):
    super(CustomSchedule, self).__init__()
    self.d_model = d_model
    self.d_model = tf.cast(self.d_model, tf.float32)
    self.warmup_steps = warmup_steps

  def __call__(self, step):
    step = tf.cast(step, tf.float32)
    arg1 = tf.math.rsqrt(step)
    arg2 = step * (self.warmup_steps**-1.5)

    return tf.math.rsqrt(self.d_model) * tf.math.minimum(arg1, arg2)
```