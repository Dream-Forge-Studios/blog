---
date: '2024-04-22'
title: 'vector DB 기반 판례 검색 시스템 제작기 1 - 소개 및 학습 데이터'
categories: ['LLM', 'Legal']
summary: 'LLM를 활용하여 판례 text embedding model을 만들어보자.'
thumbnail: './test.png'
---

<div id="개요"></div>

# 개요

<div id="목적과 중요성"></div>

## 목적과 중요성

현재 판례 검색 시스템은 키워드 기반의 단순 언어 매칭 방식을 채택하고 있습니다. 이러한 방식은 사용자가 특정 키워드를 입력했을 때 그 키워드를 포함하는 모든 판례를 검색 결과로 반환합니다.

<br>

이 과정에서 수많은 관련 없는 결과가 나타나, 사용자가 실제로 필요로 하는 판례를 찾기까지 많은 시간과 노력이 소모됩니다.

<img style="width: 50%; margin-top: 40px;" id="output" src="./precedentSerach1/figure1.PNG">

<br>

게다가, 법적 문서의 특성상 판례들은 그 내용이 복잡하고 이해하기 어려워 사용자의 부담을 더욱 가중시킵니다.

<img style="width: 50%; margin-top: 40px;" id="output" src="./precedentSerach1/figure2.PNG">

<br>

이러한 문제를 해결하기 위해, 저는 언어 모델(Large Language Model, LLM)을 fine-tuning하여 이를 활용해 판례를 벡터화하고, 이를 데이터베이스에 저장하는 방식의 새로운 판례 검색 시스템을 제안합니다.

<br>

이 시스템은 법률 전문가뿐만 아니라 일반 사용자들에게도 판례 검색을 통한 법적 지원을 보다 접근하기 쉽게 만들어 줄 것입니다.

<div id="시스템 개요"></div>

## 시스템 개요

본 판례 검색 시스템의 핵심 기술은 text embedding 입니다. 기존의 text embedding은 주로 BERT와 같은 Masked Language Model(MLM)에 의존해 왔습니다.

<br>

그러나 최근 Casual Language Model을 활용한 Large Language Models(LLMs)이 이러한 MLM 기반의 모델들보다 우수한 성능을 보여주고 있습니다. 

<br>

특히, 'Mistral' 모델을 활용한 텍스트 임베딩은 주목할 만한 성과를 나타내고 있습니다.

<br>

관련 논문: 

[LLM2Vec: Large Language Models Are Secretly Powerful Text Encoders](https://arxiv.org/abs/2404.05961), 

[Improving Text Embeddings with Large Language Models](https://arxiv.org/abs/2401.00368)

<br>

이러한 기술적 배경을 바탕으로, 저는 한국어에 특화된 'Mistral 7B' 기반의 기본 모델을 fine-tunging하여 판례에 최적화된 text embedding 모델을 개발하려고 합니다. 

<br>

본 연구는 현재 리소스를 고려할 때 모든 판례를 커버하긴 어려운 관계로, 소송 중 가장 많이 발생하는 민사의 손해배상 관련 사건을 대상으로 진행됩니다.

<div id="train data"></div>

# train data

본 판례 검색 시스템 개발의 첫 단계는 적절한 학습 데이터를 구성하는 것입니다. 처음에는 손해배상 관련 판례 데이터를 수집하고, 이를 기반으로 모델을 학습시킵니다.

<br>

이 데이터는 국가법령정보 공동활용 OPEN API를 통해 수집하였고, 2010년 1월 1일부터 2024년 4월 25일까지의 판례(2315개)를 포함하고 있습니다.

<br>

그 후 다양한 법적 도메인 데이터 및 법적 도메인 외 데이터를 조합하고 비교함으로써, 최적의 학습 데이터 구성하여 모델의 성능을 향상시키는 Data-Centric AI 연구를 진행할 예정입니다.

<div id="데이터 구성"></div>

## 데이터 구성

```
판례정보일련번호 precSeq
사건명 title
사건번호 preNum
선고일자 date
선고 select
법원명 lawName
법원종류코드 lawCode

사건종류명 caseType
사건종류코드 caseTypeCode
판결유형 judgeType
판시사항 decision
판결요지 summary
참조조문 referLaw
참조판례 referPre
판례내용 preDetails
```

평균 length: 9783
seed=42

1. 한국어 평가 지표 만들기
2. fine-tune
   - 한국어 qa 데이터를 활용하여 q의 유사도를 평가하여 비슷한 것들을 neg로 하여 학습시킨다.
   - 
3. RetroMAE
4. Unsup
5. 연구

- 실험: 기존 성능을 유지하며 필요한 부분만 성능 향상
  1. 법률 지식 주입
  3. task 특화
  

- 기능
  1. 판례 검색
  2. 법령 검색
  3. 법률상담사례 검색
     1. ANCE 
  4. open domain 성능 향상 80점
  5. 프로토타입
  6. swapMIM 프로토타입

- negative sampling 기법
    1. pos: 정답 neg: 그외 유사도 높은 기준 
        - 5개: {'ndcg@5': 0.8179}
        - 10개: {'ndcg@5': 0.8867}
        - 20개: {'ndcg@5': 0.8688}
    1. 질문과 문서 유사도 기반 pos 범위 neg 범위
        - 정답만 나머지 neg
    2. ance-다른 모델
        - 질문과 질문 유사도 기반 pos 범위 neg 범위
        - 문서와 문서 유사도 기반 pos 범위 neg 범위

1. 어댑터를 통한 법률사례검색 개발 ~ 7/5
   - 전체 fine tuning vs 마지막 어댑터 추가
2. 법률사례데이터 및 판례데이터 가공(학습, 평가) ~ 7/17
3. 법률사례, 판례 검색 개발 완료 ~ 7/25

7/1 ~ 7/6: 데이터 취합(법률 상담 사례, 판례)
7/8 ~ 7/12: 데이터 가공 및 평가 지표 개발
7/15 ~ 7/31: 모델 실험

<div id="데이터 취합"></div>

# 데이터 취합

<div id="법률 상담 사례"></div>

## 법률 상담 사례

### 출처

1. https://www.klac.or.kr/legalinfo/counsel.do: 법률 상담 사례 (10037개)
2. https://www.klac.or.kr/legalstruct/cyberConsultation/selectOpenArticleList.do?boardCode=3: 사이버 상담(2625(국내) + 49(재외))

### 평가 방법

1. train 데이터에서 20%

### negative sampling

1. 관련 문서 랭킹 자신을 제외한 나머지 10개
2. 몇번 반복
3. neg끼리에 영향

마스킹
beir 데이터도 함께


한 쿼리를 여러개로 나눠서 검색
쿼리에 특징에 따라 다른 파이프라인

### 신경써야할 부분

1. 모델
2. loss
   - neg 제약
   - 기존 임베딩에서 크게 안 바뀌도록
   - 강화학습
3. 검색 방법
   - rexical, coberta
   - 한 쿼리를 여러개로 나눠서 검색
   - 쿼리에 특징에 따라 다른 파이프라인
   - 키워드 검색
4. 데이터
    - 

질문 간의 유사도를 비교해서 그와 관련된 문서를 찾아주는 방법