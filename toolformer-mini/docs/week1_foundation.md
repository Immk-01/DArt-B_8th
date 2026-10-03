# Week 1 — Toolformer 기초 학습 및 핵심 모듈 구현

## 1. Toolformer 논문 전체 구조 파악

### 1.1 Toolformer는 왜 제안되었는가?

### 1.2 기존 Tool 사용 방식의 문제점

### 1.3 Self-Supervised Learning

### 1.4 Toolformer는 무엇을 학습하는가?


## 2. Language Model 기초

### 2.1 Token

### 2.2 Vocabulary

### 2.3 Logits

### 2.4 Softmax

### 2.5 Probability

### 2.6 Next Token Prediction


## 3. Cross Entropy와 Perplexity


## 4. In-Context Learning


## 5. Calculator Tool 구현


## 6. 기본 Loss Filtering 구현


## 7. Week 1 결과

Week 1에서는 Toolformer 전체 Pipeline을 바로 구현하기보다,
Toolformer를 이해하기 위한 이론적 기초와 각 핵심 구성 요소를
개별적으로 구현하고 검증하는 것을 목표로 하였다.

### 완료한 내용

- Toolformer 논문의 전체 구조 이해
- 기존 LLM과 기존 Tool 사용 방식의 한계 이해
- Self-Supervised Learning 개념 이해
- Token / Vocabulary / Logits / Softmax 이해
- Next Token Prediction 이해
- Cross Entropy와 Perplexity 이해
- In-Context Learning 이해
- Calculator Tool 구현
- Rule-based API Call 생성
- Calculator API 실행
- GPT-2 기반 Loss 계산
- 기본 Loss Filtering 구현

Week 1에서 구현한 기능들은 Week 2에서 서로 연결하고 확장하여,
End-to-End Mini Toolformer Pipeline으로 발전시킨다.
