# Mini Toolformer 구현 프로젝트

본 프로젝트는 논문  
**Toolformer: Language Models Can Teach Themselves to Use Tools**  
를 읽고 핵심 아이디어를 직접 구현해보는 학습 프로젝트이다.

## 프로젝트 목표

이 프로젝트의 목표는 Toolformer 논문의 모든 실험 결과를 그대로 재현하는 것이 아니다.

궁극적인 목표는 다음과 같은 LLM 논문 학습 과정을 직접 경험하는 것이다.

논문 읽기  
↓  
핵심 개념 이해  
↓  
알고리즘 분석  
↓  
코드 구현  
↓  
모델 학습  
↓  
실험  
↓  
결과 분석

즉, 단순히 논문 내용을 정리하는 것이 아니라  
**논문의 아이디어를 실제 코드와 실험으로 옮기는 경험**을 하는 것을 목표로 한다.


## 구현 범위

원래 Toolformer에서는 다음과 같은 다양한 외부 Tool을 사용한다.

- Calculator
- Question Answering
- Wikipedia Search
- Machine Translation
- Calendar

본 프로젝트에서는 전체 Tool을 구현하지 않고  
**Calculator Tool을 중심으로 Toolformer의 핵심 학습 구조를 소규모로 구현한다.**


## 전체 Pipeline

Raw Text  
↓  
API Call 생성  
↓  
Calculator 실행  
↓  
Loss 기반 Filtering  
↓  
Tool이 포함된 Dataset 생성  
↓  
LoRA Fine-tuning  
↓  
Tool Calling  
↓  
평가 및 분석


# 학습 일정

## Week 0 — 프로젝트 방향 설정

### 목표

프로젝트의 범위와 구현 전략을 결정한다.

### 진행 내용

- Toolformer 논문 선정
- 프로젝트 최종 목표 설정
- Calculator Tool을 중심으로 구현 범위 결정
- 2주 학습 계획 수립


## Week 1 — Toolformer 기초 이해 및 핵심 모듈 구현

### 목표

Toolformer를 구현하기 위해 필요한 개념을 이해하고,
전체 Pipeline을 구성하는 개별 기능을 먼저 구현하고 검증한다.

### 논문 학습

- Toolformer가 제안된 이유
- 기존 LLM의 한계
- 기존 Tool 사용 방식의 한계
- Self-Supervised Learning
- Toolformer의 Tool 사용 의사결정 과정

### Language Model 기초

- Token
- Vocabulary
- Logits
- Softmax
- Probability
- Next Token Prediction
- Cross Entropy
- Perplexity
- In-Context Learning

### 구현

- Calculator Tool 구현
- 정규표현식 기반 API Call 생성
- Calculator API Call 실행
- GPT-2 기반 Language Model Loss 계산
- 기본 Loss Filtering 구현

### Week 1 Pipeline

Input Text  
↓  
Rule-based Calculator Call 생성  
↓  
Calculator 실행  
↓  
Language Model Loss 계산  
↓  
기본 Loss Filtering


## Week 2 — Pipeline 통합 및 Toolformer 방식 고도화

### 목표

Week 1에서 개별적으로 구현한 기능들을 연결하고,
Toolformer 논문의 구조에 더 가까운 End-to-End Pipeline으로 발전시킨다.

### 진행 내용

1. LLM 기반 API Call 생성
2. 실제 Loss Filtering 실험
3. Toolformer 방식 Future Token Loss 구현
4. Augmented Dataset 생성
5. Dataset 통계 분석
6. LoRA Fine-tuning
7. Tool Calling Inference 구현
8. Base Model과 성능 비교
9. Failure Case 분석


## 최종 목표 Pipeline

Raw Dataset  
↓  
LLM 기반 API Call 생성  
↓  
Calculator 실행  
↓  
Toolformer 방식 Loss Filtering  
↓  
Augmented Dataset  
↓  
LoRA Fine-tuning  
↓  
Tool 사용이 가능한 Language Model  
↓  
Tool Calling Inference  
↓  
Evaluation  
↓  
Failure Case Analysis


## 사용 기술

- Python
- PyTorch
- Hugging Face Transformers
- GPT-2 / Small Causal Language Model
- PEFT
- LoRA


## 최종 목표

이번 프로젝트를 통해 다음 과정을 직접 경험한다.

Paper  
↓  
Concept  
↓  
Algorithm  
↓  
Implementation  
↓  
Training  
↓  
Experiment  
↓  
Analysis

이를 통해 향후 다른 LLM 논문을 읽을 때도
핵심 아이디어를 분석하고 직접 구현할 수 있는 능력을 기르는 것을 목표로 한다.
