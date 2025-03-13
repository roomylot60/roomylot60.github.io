---
title: Subword Tokenizer
author: Patrick
date: 2023-11-28 19:50:00 +0900
tags: [Concept]
categories: [NLP]
render_with_liquid: false
---
## Subword Tokenizer
### Subword
- OOV(Out-Of Vocabulary) : 새로 제시된 단어에 대해 기존의 dictionary 내의 단어가 아닌 경우
- Subword : 하나의 단어를 구성하는 더 작은 단위의 의미를 가진 어구
- Subword segmenation : 하나의 단어를 여러 서브 워드로 분리해서 단어를 인코딩 및 임베딩하는 전처리 작업

#### Byte Pair Encoding;BPE
- 연속적으로 가장 많이 등장한 `글자의 쌍(Byte Pair)`을 찾아서 하나의 글자로 병합하는 방식을 수행하는 알고리즘        
1. 글자 단위에서 단어 집합을 만들어 내는 Bottom up 방식의 접근을 사용
2. Dictionary 내의 단어에 대해 사용자가 정한 반복횟수만큼 가장 빈도수가 높은 Unigram의 쌍을 하나의 Unigram으로 통합하는 과정을 반복
3. 반복을 통해 얻은 Unigram을 dictionary에 update
4. 새로 제시된 단어를 글자 단위로 분할하여 dictionary 내의 Unigram으로 구성되었는지 여부를 확인
    * WordPiece Tokenizer : BPE의 변형 알고리즘 중 하나로, BPE가 빈도수에 기반하여 가장 많이 등장한 쌍을 병합하는 것과는 달리, 병합되었을 때 corpus의 Likelihood(우도)를 가장 높이는 쌍을 병합; 띄어쓰기를 사용하여 subword들을 구분하고, `'_'`를 사용하여 기존 문장의 띄어쓰기를 구분

```python
import re, collections
from IPython.display import display, Markdown, Latex

num_merges = 10 # BPE 수행횟수

dictionary = {'l o w <\w>' : 5,
            'l o w e r <\w>' : 2,
            'n e w e s t <\w>' : 6,
            'w i d e s t <\w>' : 3}

def get_stats(dictionary):
    # Unigram의 pair들의 빈도수를 카운트
    pairs = collections.defaultdict(int)
    for word, freq in dictionary.items():
        symbols = word.split()
        for i in range(len(symbols)-1):
            pairs[symbols[i],symbols[i+1]] += freq
    print('현재 pair들의 빈도수 :', dict(pairs))
    return pairs

def merge_dictionary(pair, v_in):
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    for word in v_in:
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

bpe_codes = {}
bpe_codes_reverse = {}

for i in range(num_merges):
    display(Markdown("### Iteration {}".format(i + 1)))
    pairs = get_stats(dictionary)
    best = max(pairs, key=pairs.get)
    dictionary = merge_dictionary(best, dictionary)

    bpe_codes[best] = i
    bpe_codes_reverse[best[0] + best[1]] = best

    print("new merge: {}".format(best))
    print("dictionary: {}".format(dictionary))

# OOV 대응
def get_pairs(word):
    """Return set of symbol pairs in a word.
    Word is represented as a tuple of symbols (symbols being variable-length strings).
    """
    pairs = set()
    prev_char = word[0]
    for char in word[1:]:
        pairs.add((prev_char, char))
        prev_char = char
    return pairs


def encode(orig):
    """Encode word based on list of BPE merge operations, which are applied consecutively"""

    word = tuple(orig) + ('</w>',)
    display(Markdown("__word split into characters:__ <tt>{}</tt>".format(word)))

    pairs = get_pairs(word)    

    if not pairs:
        return orig

    iteration = 0
    while True:
        iteration += 1
        display(Markdown("__Iteration {}:__".format(iteration)))

        print("bigrams in the word: {}".format(pairs))
        bigram = min(pairs, key = lambda pair: bpe_codes.get(pair, float('inf')))
        print("candidate for merging: {}".format(bigram))
        if bigram not in bpe_codes:
            display(Markdown("__Candidate not in BPE merges, algorithm stops.__"))
            break
        first, second = bigram
        new_word = []
        i = 0
        while i < len(word):
            try:
                j = word.index(first, i)
                new_word.extend(word[i:j])
                i = j
            except:
                new_word.extend(word[i:])
                break

            if word[i] == first and i < len(word)-1 and word[i+1] == second:
                new_word.append(first+second)
                i += 2
            else:
                new_word.append(word[i])
                i += 1
        new_word = tuple(new_word)
        word = new_word
        print("word after merging: {}".format(word))
        if len(word) == 1:
            break
        else:
            pairs = get_pairs(word)

    # 특별 토큰인 </w>는 출력하지 않는다.
    if word[-1] == '</w>':
        word = word[:-1]
    elif word[-1].endswith('</w>'):
        word = word[:-1] + (word[-1].replace('</w>',''),)

    return word

```

#### BPE Packages
1. Sentence Piece : 내부 단어 분리를 위한 패키지로, pretokenization 없이 전처리를 하지 않은 raw data에 바로 단어 분리 토크나이저를 사용할 수 있어 언어에 종속되지 않도록 BPE 알고리즘과 Unigram LM Tokenizer를 구현
2. Subword Text Encoder : Tensorflow를 통해 사용할 수 있는 Subword Tokenizer로 Wordpiece Model을 채택
3. Huggingface's Tokenizers : 자주 등장하는 서브워드들을 하나의 토큰으로 취급하는 다양한 Subword Tokenizer를 제공

    |Tokenizer|Descriptions|
    |-----|----------|
    |BertWordPieceTokenizer|허깅페이스에서 구현한 BERT 모델의 WordPiece Tokenizer로 tokenizers 패키지를 통해 제공|
    |CharBPETokenizer|오리지널 BPE|
    |ByteLevelBPETokenizer|BPE의 Byte level 버전|
    |SentencePieceBPETokenizer|SentencePiece BPE 구현체|