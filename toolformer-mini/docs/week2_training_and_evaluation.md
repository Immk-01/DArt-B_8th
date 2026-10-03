# Week 2 — Toolformer Pipeline 통합, 학습 및 평가

## 1. Week 2 목표

Week 1에서는 Toolformer 구현에 필요한 핵심 개념을 학습하고,
Calculator, API Call, Language Model Loss, Filtering과 같은
각 구성 요소를 개별적으로 구현하였다.

Week 2에서는 이를 하나의 Pipeline으로 통합하고,
Week 1의 단순화된 구현을 실제 Toolformer 구조에 더 가깝게 발전시킨다.

최종적으로 다음 Pipeline을 구현하는 것을 목표로 한다.

Raw Text
↓
LLM 기반 API Call 생성
↓
Calculator 실행
↓
Toolformer 방식 Loss Filtering
↓
Augmented Dataset 생성
↓
LoRA Fine-tuning
↓
Tool Calling Inference
↓
Model Evaluation
↓
Failure Case Analysis
