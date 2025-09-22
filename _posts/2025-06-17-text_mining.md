---
title: "Text_mining 실습"
date: 2025-06-17 21:51:00 +0900
categories: [Blog]
tags: [Sungil Software Study, Blog]
---

# 한국어 텍스트 마이닝 & 워드클라우드

> 이 문서는 대한민국헌법 텍스트를 예시로, 한국어 텍스트 분석과 시각화를 단계별로 설명합니다.
> 분석 과정에서는 **명사 추출 → 단어 정제 → 빈도 계산 → 시각화**의 흐름을 따라가며, Python과 Jupyter Notebook을 활용합니다.

![대표 이미지](/assets/images/TextCloud.png)

> 💡 설명: 이 이미지는 분석 결과를 직관적으로 보여주는 워드클라우드 예시입니다. 실제 데이터와 형태는 달라질 수 있습니다.

---

## 1️⃣ 환경 준비

먼저, 분석에 필요한 **환경을 설정**합니다. 한국어 텍스트를 제대로 처리하기 위해서는 한글 폰트와 형태소 분석 라이브러리가 필요합니다.

```bash
sudo apt-get install -y fonts-nanum
sudo fc-cache -fv
rm ~/.cache/matplotlib -rf

pip install konlpy jpype1 wordcloud
```

> 💡 설명:
>
> * `fonts-nanum` 설치 → 한글 폰트 설치
> * `fc-cache` → 폰트 캐시를 갱신
> * `konlpy` → 한국어 형태소 분석
> * `jpype1` → Java 기반 라이브러리를 파이썬에서 사용 가능하게 연결
> * `wordcloud` → 단어 구름 시각화

이 과정을 통해 한글이 깨지지 않고, 한국어 텍스트 분석과 시각화가 원활하게 진행됩니다.

---

## 2️⃣ 텍스트 읽기 및 전처리

분석할 텍스트를 불러오고, **필요 없는 문자나 기호, 숫자**를 제거합니다.

```python
import re
import konlpy

# 텍스트 읽기
constitution = open('대한민국헌법.txt', encoding='UTF-8').read()

# 숫자와 특수기호 제거
constitution = re.sub('[0-9①-⑨]', ' ', constitution)

# 형태소 분석 준비
hannanum = konlpy.tag.Hannanum()

# 명사만 추출
nouns = hannanum.nouns(constitution)
```

> 💡 설명:
>
> * 텍스트를 깨끗하게 정제하면 분석 정확도가 올라갑니다.
> * Hannanum 분석기를 사용하여 명사만 뽑음으로써 핵심 키워드를 추출합니다.
> * 명사 추출 후, 불필요한 조사나 단어는 후처리에서 걸러낼 수 있습니다.

---

## 3️⃣ 데이터 정리 및 단어 빈도 계산

추출한 명사 리스트를 **데이터프레임**으로 변환하고, 단어 길이와 등장 빈도를 기준으로 정리합니다.

```python
import pandas as pd

# 리스트를 데이터프레임으로 변환
df_word = pd.DataFrame({'word': nouns})

# 단어 길이 계산
df_word['count'] = df_word['word'].str.len()

# 길이 2 이상인 단어만 남김
df_word = df_word.query('count >= 2')

# 단어별 등장 횟수 계산
df_word = df_word.groupby('word', as_index=False)\
                 .agg(n=('word', 'count'))\
                 .sort_values('n', ascending=False)

# 상위 20개 단어 선택
top20 = df_word.head(20)
```

> 💡 설명:
>
> * 길이가 1인 단어는 의미가 약한 경우가 많으므로 제거합니다.
> * 그룹화와 집계(aggregate)를 통해 각 단어가 텍스트에서 몇 번 등장했는지 계산합니다.
> * 상위 20개의 단어를 선택하면, 핵심 키워드를 빠르게 파악할 수 있습니다.

---

## 4️⃣ 막대 그래프 시각화

상위 단어를 막대 그래프로 표현하면, 어떤 단어가 텍스트에서 **가장 많이 등장하는지** 한눈에 확인할 수 있습니다.

```python
import matplotlib.pyplot as plt
import seaborn as sns

# 한글 폰트 설정
plt.rcParams['font.family'] = 'NanumGothic'

# 막대 그래프 생성
sns.barplot(data=top20, y='word', x='n')
plt.title('상위 20 단어 빈도')
plt.xlabel('등장 횟수')
plt.ylabel('단어')
plt.show()
```

> 💡 설명:
>
> * 시각화로 단어 빈도를 직관적으로 이해할 수 있습니다.
> * y축에는 단어, x축에는 빈도 수를 나타내어 쉽게 비교할 수 있습니다.

---

## 5️⃣ 워드 클라우드 생성

단어 빈도를 **워드 클라우드 형태로 시각화**하면, 단어의 빈도와 중요도를 크기와 색상으로 직관적으로 보여줍니다.

```python
from wordcloud import WordCloud

FONT_PATH = '/usr/share/fonts/truetype/nanum/NanumSquareRoundB.ttf'
dic_word = df_word.set_index('word').to_dict()['n']

# 워드 클라우드 객체 생성
wc = WordCloud(
    random_state=1234,
    font_path=FONT_PATH,
    width=400,
    height=400,
    background_color='white'
)

# 단어 빈도로 이미지 생성
img_wordcloud = wc.generate_from_frequencies(dic_word)

plt.figure(figsize=(10,10))
plt.axis('off')
plt.imshow(img_wordcloud)
plt.show()
```

> 💡 설명:
>
> * 각 단어의 크기는 등장 빈도와 비례합니다.
> * 배경색, 이미지 크기, 폰트 등 다양한 옵션으로 시각화 스타일을 조정할 수 있습니다.
> * 워드 클라우드는 텍스트 데이터에서 핵심 키워드를 **직관적이고 시각적으로 파악**할 때 매우 유용합니다.

---

