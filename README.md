# Mini AI Pipeline Project: Financial Sentiment Analysis

**Name:** [박동준]
**Student ID:** [2022148094]
**Date:** 2025-12-10

---

## 1. Introduction & Problem Definition
### Task Description
본 프로젝트의 목표는 금융 뉴스 텍스트(Financial News Headlines)를 입력받아 해당 뉴스가 시장에 **긍정(Positive)**인지, **부정(Negative)**인지, 혹은 **중립(Neutral)**인지 분류하는 **감성 분석(Sentiment Analysis)** 파이프라인을 구축하는 것입니다.

### Motivation
금융 시장에서 뉴스는 주가에 즉각적인 영향을 미치는 핵심 요소입니다. 쏟아지는 뉴스 속에서 호재와 악재를 실시간으로 파악하는 자동화된 시스템은 퀀트 트레이딩(Quantitative Trading) 및 시장 분석에 필수적입니다.

### Input / Output
* **Input:** 영문 금융 뉴스 문장 (예: "Operating profit rose to EUR 13.1 mn.")
* **Output:** 감성 레이블 (0: Negative, 1: Neutral, 2: Positive)

---

## 2. Dataset
### Source & Description
* **Dataset Name:** `atrost/financial_phrasebank`
* **Description:** 핀란드 알토 대학교(Aalto University)에서 구축한 금융 뉴스 데이터셋으로, 금융 전문가들이 라벨링한 고품질 데이터입니다.
* **Data Split:** 전체 데이터 중 80%를 훈련(Train)용으로 사용하려 했으나, 사전 학습된 모델을 활용하므로 전체 데이터를 로드한 뒤 **20%를 테스트(Test) 셋**으로 분리하여 평가에 사용했습니다.
* **Size:** 테스트 데이터셋 총 **970개** 문장

---

## 3. Methods

### 3.1 Naïve Baseline (Rule-based)
복잡한 모델 없이 금융 분야에서 자주 쓰이는 긍정/부정 키워드 사전을 구축하여 단순 매칭하는 방식을 사용했습니다.

* **Algorithm:**
    * 문장에 `profit`, `growth`, `rise` 등의 긍정 단어가 더 많으면 **Positive**
    * `loss`, `fall`, `decline` 등의 부정 단어가 더 많으면 **Negative**
    * 개수가 같거나 키워드가 없으면 **Neutral**
* **Limitations:** 문맥을 보지 않고 단순 단어 유무만 확인하므로, "Profit dropped" 같이 긍정/부정 단어가 섞여 있는 경우 오판할 가능성이 높습니다.

### 3.2 AI Pipeline (FinBERT)
단순 규칙의 한계를 극복하기 위해 금융 텍스트에 특화되어 사전 학습된(Pre-trained) **FinBERT** 모델을 사용했습니다.

* **Model:** `ProsusAI/finbert` (Hugging Face Transformers)
* **Pipeline Design:**
    1.  **Preprocessing:** 텍스트를 토큰화(Tokenization)
    2.  **Inference:** BERT 모델을 통해 문맥 정보(Context)가 포함된 임베딩을 추출하고 분류 수행
    3.  **Post-processing:** 모델의 출력 로짓(Logit)을 0, 1, 2 라벨로 변환

---

## 4. Experiments & Results

### 4.1 Quantitative Results (Accuracy)
동일한 테스트 데이터(970개)에 대해 정확도를 측정한 결과입니다.

| Model | Accuracy |
| :--- | :--- |
| **Naïve Baseline** | **60.52%** |
| **AI Pipeline (FinBERT)** | **86.49%** |

AI 파이프라인이 베이스라인 대비 약 **26%p** 높은 성능을 보였습니다.

### 4.2 Qualitative Analysis (Success Cases)
베이스라인은 틀렸지만, AI 모델이 정답을 맞힌 주요 사례 분석입니다.

**Case 1:**
> *Text:* "L&T 's net **profit** for the whole 2010 **dropped** to EUR 36 million from EUR 45 million for 2009."
> * **Label:** Negative (0)
> * **Baseline Prediction:** Neutral (1)
> * **AI Prediction:** Negative (0)
> * **Analysis:** 베이스라인은 긍정 단어인 'profit'과 부정 단어인 'dropped'가 동시에 등장하여 서로 상쇄되면서 'Neutral'로 잘못 판단했습니다. 반면, AI는 "이익이 떨어졌다(Profit dropped)"는 문맥을 정확히 이해하여 'Negative'로 분류했습니다.

**Case 2:**
> *Text:* "Net sales **fell** by 5 % from the previous accounting period."
> * **Label:** Negative (0)
> * **Baseline Prediction:** Neutral (1)
> * **Analysis:** 베이스라인의 키워드 사전에는 'fall'은 있었으나 과거형인 'fell'이 포함되지 않았거나 매칭되지 않아 부정적인 의미를 놓쳤습니다. BERT 모델은 단어의 형태 변화(fell -> fall)와 문맥을 이해하고 있어 정확히 분류했습니다.

---

## 5. Reflection & Limitations

* **Success:** 단순한 키워드 매칭으로는 해결할 수 없는 '문맥(Context)'의 중요성을 확인했습니다. 특히 긍정 단어(profit)와 부정 서술어(dropped)가 결합될 때 AI 모델의 성능이 압도적이었습니다.
* **Challenges:** Hugging Face 라이브러리 버전 호환성 문제와 서버 용량 문제(OS Error 28)로 인해 모델 로딩에 어려움이 있었으나, `transformers` 버전을 조정하고 캐시 경로를 변경하여 해결했습니다.
* **Future Work:** 현재는 문장 단위 분류만 수행하지만, 추후에는 주가 데이터와 연동하여 뉴스가 실제 가격 변동에 미치는 상관관계를 분석하는 파이프라인으로 확장해보고 싶습니다.

---

## 6. How to Run
```bash
# 1. Install dependencies
pip install transformers datasets scikit-learn pandas torch

# 2. Run the notebook
jupyter notebook pipeline_project.ipynb
