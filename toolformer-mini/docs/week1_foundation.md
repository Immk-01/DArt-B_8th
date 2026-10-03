# Week 1 — Toolformer Foundation

> **Paper:** Toolformer: Language Models Can Teach Themselves to Use Tools

## Week 1 목표

Week 1의 목표는 Toolformer를 직접 구현하기 전에 필요한 **Language Model의 기본 개념과 Toolformer 논문의 핵심 구조를 이해하는 것**이다.

또한 논문의 전체 구조를 바로 구현하기보다는 가장 간단한 **Calculator Tool**을 구현하고, GPT-2를 이용하여 **Tool 사용 전후의 Loss를 비교하는 기본 Filtering 구조**까지 구현한다.

### 이번 주 학습 및 구현 범위

- Toolformer 논문의 전체 구조 이해
- 기존 LLM의 한계와 Tool 사용의 필요성 이해
- Self-Supervised Learning 이해
- Token / Vocabulary / Logit / Softmax 이해
- Probability / Next Token Prediction 이해
- Cross Entropy / Perplexity 이해
- In-Context Learning 이해
- Calculator Tool 구현
- GPT-2를 이용한 Loss 계산
- Loss Improvement 계산
- 간단한 Threshold 기반 Filtering 구현

---

# 1. Toolformer 논문 전체 구조

## 1.1 Toolformer는 왜 제안되었는가?

기존 Large Language Model(LLM)은 문장 생성과 다양한 자연어 처리 문제를 해결하는 능력이 뛰어나지만, **모델 내부의 지식과 계산 능력만으로 해결하기 어려운 영역이 존재한다.**

대표적인 한계는 다음과 같다.

- 정확한 계산을 수행하기 어려움
- 최신 정보에 직접 접근할 수 없음
- 잘못된 사실을 생성하는 Hallucination 문제
- 일부 저자원 언어에 대한 이해 능력의 한계
- 현재 날짜와 같은 시간 정보 인식의 한계
- 단순한 사실 검색에서도 오류가 발생할 수 있음

Toolformer는 이러한 문제를 해결하기 위해 LLM이 필요한 상황에서 외부 Tool을 사용할 수 있도록 학습시키는 방법을 제안한다.

논문에서는 다음과 같은 Tool을 사용한다.

- Calculator
- Search Engine
- Question Answering System
- Translation System
- Calendar

즉, 모델 자체가 모든 문제를 해결하도록 만드는 것이 아니라 필요한 경우 외부 Tool을 호출하여 부족한 능력을 보완하도록 한다.

```text
LLM 자체 능력의 한계
        ↓
계산 / 검색 / 번역 / 시간 정보 등의 문제 발생
        ↓
외부 Tool 호출
        ↓
Tool 결과 활용
        ↓
LLM의 부족한 능력 보완
```

---

# 2. 기존 Tool 사용 방식의 문제점

Toolformer 이전에도 Language Model이 Calculator나 Search Engine 등의 Tool을 사용하는 연구는 존재했다.

하지만 기존 방식에는 크게 두 가지 문제가 있었다.

## 2.1 많은 Human Annotation이 필요함

기존 방식에서는 사람이 직접 다음과 같은 Tool 사용 데이터를 만들어야 하는 경우가 많았다.

```text
이 상황에서는 Calculator를 사용한다.

이 질문에서는 Search Engine을 사용한다.

Calculator에는 이 값을 Input으로 전달한다.
```

이 방식은 다음과 같은 문제를 가진다.

- 데이터 구축 비용이 많이 든다.
- 많은 Human Annotation이 필요하다.
- 사람이 유용하다고 판단한 Tool Call과 실제 모델에게 유용한 Tool Call이 다를 수 있다.

---

## 2.2 특정 Task에 Tool 사용 방법이 종속됨

기존 방식에서는 특정 Task에 맞춰 사용할 Tool이나 Tool 사용 방법이 미리 정의되는 경우가 많았다.

즉,

```text
특정 문제
    ↓
미리 지정된 Tool 사용
```

과 같은 구조였다.

모델이 스스로

- Tool이 필요한지
- 어떤 Tool이 필요한지
- Tool에 무엇을 입력해야 하는지

판단하기보다는 사람이 정한 규칙에 의존하는 경우가 많았다.

Toolformer는 이러한 문제를 **Self-Supervised Learning**을 이용하여 해결하고자 한다.

---

# 3. Self-Supervised Learning

## 3.1 개념

Self-Supervised Learning은 사람이 정답 Label을 직접 만들어주지 않고, **원본 데이터 자체에서 학습에 필요한 정답 또는 학습 신호를 만들어 사용하는 방식**이다.

### Supervised Learning

예를 들어 다음 문제가 있다고 하자.

```text
입력:
나는 오늘 학교에 _____

정답:
갔다
```

Supervised Learning에서는 사람이 직접 `갔다`를 정답 Label로 만들어줘야 한다.

---

### Self-Supervised Learning

원본 문장이 다음과 같이 존재한다고 하자.

```text
나는 오늘 학교에 갔다.
```

원본 문장 자체를 이용하여 다음과 같은 학습 데이터를 만들 수 있다.

```text
입력:
나는 오늘 학교에 _____

정답:
갔다
```

즉, 사람이 별도로 정답을 작성하지 않아도 **원래 존재하는 데이터에서 학습 신호를 자동으로 생성할 수 있다.**

| 구분 | Supervised Learning | Self-Supervised Learning |
|---|---|---|
| 입력 | 나는 오늘 학교에 ___ | 나는 오늘 학교에 ___ |
| 정답 | 갔다 | 갔다 |
| 정답 생성 | 사람이 직접 생성 | 원본 데이터에서 생성 |
| 사람의 Labeling | 필요 | 상대적으로 적음 |
| 핵심 | 사람이 만든 정답 학습 | 데이터 자체에서 학습 신호 생성 |

Toolformer 역시 이러한 아이디어를 이용하여 대규모 Human Annotation 없이 Tool 사용 데이터를 만들어낸다.

---

# 4. Toolformer가 학습하는 것

Toolformer의 핵심은 단순히 **API를 호출하는 방법만 학습하는 것**이 아니다.

모델은 다음 네 가지를 학습해야 한다.

1. **어떤 Tool을 사용할 것인가?**
2. **언제 Tool을 사용할 것인가?**
3. **Tool에 어떤 Input을 넣을 것인가?**
4. **Tool의 결과를 이후 문장 생성에 어떻게 활용할 것인가?**

예를 들어 다음 문제가 있다고 하자.

```text
Out of 1400 participants, 400 passed the test.
This is approximately ____%.
```

모델은 다음과 같은 과정을 수행해야 한다.

```text
계산이 필요한 상황인지 판단
        ↓
Calculator 선택
        ↓
400 / 1400 Input 생성
        ↓
Calculator 호출
        ↓
계산 결과 획득
        ↓
결과를 이후 Token 생성에 활용
```

Toolformer에서는 API Call Candidate를 먼저 생성한 뒤 실제 Tool을 실행한다.

그리고 **Tool 결과가 이후 Token을 예측하는 데 실제로 도움이 되는지 Loss를 이용하여 평가한다.**

Tool 사용 후 Loss가 충분히 감소한 API Call만 최종 Training Dataset에 남긴다.

---

# 5. Toolformer 전체 학습 흐름

Toolformer의 전체적인 학습 구조는 다음과 같다.

```text
Raw Text
    ↓
API Call Candidate 생성
    ↓
실제 Tool 실행
    ↓
Tool Result 삽입
    ↓
Tool 사용 전/후 Loss 비교
    ↓
유용한 API Call Filtering
    ↓
Augmented Dataset 생성
    ↓
Language Model Fine-tuning
```

핵심은 단순히 Tool을 사용하는 것이 아니라,

> 모델이 직접 Tool Call 후보를 만들고, 실제로 도움이 되는 Tool Call만 골라 다시 학습에 사용한다는 것이다.

---

# 6. Language Model 기초

Toolformer를 이해하기 위해서는 Language Model의 기본적인 동작 방식에 대한 이해가 필요하다.

---

## 6.1 Token

Token은 Language Model이 텍스트를 처리하는 기본 단위이다.

Tokenizer는 입력된 문장을 여러 Token으로 나누고 각각을 숫자 ID로 변환한다.

예를 들어 다음 문장이 있다고 하자.

```text
I love artificial intelligence.
```

Tokenizer에 따라 다음과 같이 분리될 수 있다.

```text
I
love
artificial
intelligence
.
```

하지만 실제 LLM에서는 Token이 반드시 하나의 단어와 일치하지 않는다.

Token은 다음과 같은 형태가 될 수 있다.

- 단어
- 단어의 일부(Subword)
- 문자
- 숫자
- 기호
- Special Token

Toolformer에서는 API Call을 표현하기 위한 특별한 Token을 사용하여 일반 텍스트와 Tool 호출을 구분한다.

---

## 6.2 Vocabulary

Vocabulary는 모델이 사용할 수 있는 모든 Token의 집합이다.

LLM은 정확히 말하면 다음 **단어**를 예측하는 것이 아니라 Vocabulary 안에 있는 다음 **Token**을 예측한다.

```text
현재까지의 Token
        ↓
Vocabulary의 모든 Token 후보 평가
        ↓
다음 Token 선택
```

---

## 6.3 Logits

Logit은 Language Model이 다음 Token 후보들에게 부여하는 **원시 점수(Raw Score)**이다.

예를 들어 다음 문장이 있다고 하자.

```text
나는 오늘 학교에
```

모델은 다음 Token 후보에 대해 다음과 같은 값을 출력할 수 있다.

| 다음 Token | Logit |
|---|---:|
| 갔다 | 5.2 |
| 먹었다 | 1.3 |
| 잤다 | 0.7 |
| 오늘 | -1.2 |
| 학교에 | -2.1 |

`갔다`의 Logit이 가장 높기 때문에 모델은 `갔다`가 다음 Token으로 적절할 가능성이 높다고 판단한다.

하지만 다음과 같이 해석하면 안 된다.

```text
Logit 5.2 = 확률 52%
```

Logit은 확률이 아니라 아직 변환되지 않은 점수이다.

---

## 6.4 Softmax

Softmax는 Logit 값을 확률 분포로 변환한다.

예를 들어 다음과 같이 변환될 수 있다.

| 다음 Token | Probability |
|---|---:|
| 갔다 | 0.95 |
| 먹었다 | 0.02 |
| 잤다 | 0.02 |
| 오늘 | 0.005 |
| 학교에 | 0.005 |

전체 확률의 합은 1이 된다.

```text
Logits
   ↓
Softmax
   ↓
Probability Distribution
```

---

## 6.5 Probability

Probability는 현재까지 등장한 Token들이 주어졌을 때 특정 Token이 다음 Token으로 등장할 가능성을 의미한다.

예를 들어

```text
나는 오늘 학교에
```

라는 Context가 있을 때 모델은 다음과 같은 확률을 계산한다.

```text
P("갔다" | "나는 오늘 학교에")
```

즉,

```text
이전 Token들
    ↓
다음 Token이 나올 확률
```

을 계산하는 것이다.

---

## 6.6 Next Token Prediction

LLM의 기본적인 학습 방식은 **Next Token Prediction**이다.

예를 들어 다음 문장이 있다고 하자.

```text
나는 오늘 학교에 갔다.
```

모델은 다음과 같은 방식으로 학습한다.

```text
나는
↓
오늘

나는 오늘
↓
학교에

나는 오늘 학교에
↓
갔다

나는 오늘 학교에 갔다
↓
.
```

즉, 이전 Token을 기반으로 다음 Token을 계속 예측한다.

---

# 7. Cross Entropy와 Loss

## 7.1 Cross Entropy

Cross Entropy는 Language Model이 **정답 Token을 얼마나 잘 예측했는지 측정하는 Loss**이다.

예를 들어 다음 문제가 있다고 하자.

```text
나는 오늘 학교에 _____
```

실제 정답 Token이 `갔다`라고 가정한다.

### 경우 1

모델이 다음과 같이 예측했다.

```text
P("갔다") = 0.90
```

정답 Token에 높은 확률을 주었기 때문에 Loss가 낮다.

### 경우 2

```text
P("갔다") = 0.01
```

정답 Token에 매우 낮은 확률을 주었기 때문에 Loss가 높다.

따라서 다음과 같이 이해할 수 있다.

```text
정답 Token 확률 ↑
        ↓
Cross Entropy ↓
        ↓
좋은 예측
```

반대로

```text
정답 Token 확률 ↓
        ↓
Cross Entropy ↑
        ↓
나쁜 예측
```

---

## 7.2 Loss

Loss는 모델의 예측이 실제 정답과 얼마나 다른지를 수치로 나타낸 값이다.

Language Model에서는 실제 다음 Token에 모델이 얼마나 높은 확률을 부여했는지를 이용하여 Loss를 계산한다.

```text
Loss 낮음
→ 예측을 잘함

Loss 높음
→ 예측이 좋지 않음
```

---

# 8. Perplexity

Perplexity는 Language Model이 다음 Token을 얼마나 잘 예측하는지를 나타내는 지표이다.

Cross Entropy와 밀접한 관계가 있다.

```text
Cross Entropy ↓
→ Perplexity ↓

Cross Entropy ↑
→ Perplexity ↑
```

따라서 일반적으로 **낮은 Perplexity는 모델이 다음 Token을 더 잘 예측하고 있다는 의미**이다.

Toolformer에서는 직접적으로 Tool Call Filtering을 위해 Tool Result가 이후 Token Prediction Loss를 얼마나 감소시키는지를 이용한다.

---

# 9. In-Context Learning

In-Context Learning은 모델의 Parameter를 다시 학습하지 않고 **Prompt 안에 제공된 예시를 보고 수행해야 하는 작업을 파악하는 방식**이다.

예를 들어 다음 Prompt를 제공할 수 있다.

```text
Input: 2 + 3
Output: Calculator(2 + 3)

Input: 5 * 4
Output: Calculator(5 * 4)

Input: 10 + 20
Output:
```

모델은 앞의 예시에서 패턴을 파악하여 다음과 같은 결과를 생성할 수 있다.

```text
Calculator(10 + 20)
```

즉,

```text
Example 제공
    ↓
Pattern 파악
    ↓
새로운 Input에 Pattern 적용
```

과 같은 방식이다.

Few-shot Prompting은 대표적인 In-Context Learning 방식 중 하나이다.

Toolformer에서는 API Call Candidate를 생성할 때 이러한 In-Context Learning 아이디어를 활용한다.

---

# 10. Calculator Tool 구현

Toolformer 전체 구조를 바로 구현하기 전에 가장 간단한 Calculator Tool부터 구현하였다.

현재 단계에서는 실제 외부 API를 호출하지 않고 Python 내부에서 계산하도록 구현하였다.

---

## 10.1 Calculator API Call 생성

```python
import re


def generate_api_call(text):
    numbers = re.findall(r"\d+", text)

    if len(numbers) >= 2:
        num1 = numbers[0]
        num2 = numbers[1]

        return f"[Calculator({num1} * {num2})]"

    return None
```

예시 입력:

```text
A box contains 12 items and there are 8 boxes.
```

정규표현식을 이용해 다음 숫자를 추출한다.

```text
12
8
```

그리고 다음과 같은 Calculator Call을 생성한다.

```text
[Calculator(12 * 8)]
```

---

## 10.2 Calculator API Call 실행

```python
def execute_api_call(api_call):
    match = re.search(
        r"\[Calculator\((\d+)\s*([+\-*/])\s*(\d+)\)\]",
        api_call
    )

    if not match:
        return None

    num1 = int(match.group(1))
    operator = match.group(2)
    num2 = int(match.group(3))

    if operator == "*":
        result = num1 * num2

    elif operator == "+":
        result = num1 + num2

    elif operator == "-":
        result = num1 - num2

    elif operator == "/":
        result = num1 / num2

    else:
        return None

    return result
```

현재 Calculator는 다음 네 가지 연산을 처리할 수 있다.

```text
+
-
*
/
```

---

## 10.3 Calculator 전체 실행

```python
if __name__ == "__main__":

    text = "A box contains 12 items and there are 8 boxes."

    # Step 1. API Call 생성
    api_call = generate_api_call(text)

    # Step 2. Calculator 실행
    result = execute_api_call(api_call)

    print("Calculator 계산 결과:")
    print(result)

    # Step 3. Toolformer 형태로 결과 표현
    final_api_call = api_call[:-1] + f" -> {result}]"

    print("\nAPI Call + 결과:")
    print(final_api_call)

    print("\n최종 정답:")
    print(result)
```

실행 결과:

```text
Calculator 계산 결과:
96

API Call + 결과:
[Calculator(12 * 8) -> 96]

최종 정답:
96
```

현재 구현 흐름은 다음과 같다.

```text
Input Text
    ↓
숫자 추출
    ↓
Calculator API Call 생성
    ↓
Calculator 실행
    ↓
Result 반환
    ↓
API Call + Result 생성
```

---

# 11. 현재 Calculator 구현의 한계

현재 구현에서는 문장에서 발견되는 처음 두 숫자를 이용하여 무조건 곱셈을 수행한다.

```python
return f"[Calculator({num1} * {num2})]"
```

따라서 다음과 같은 문장이 입력되면 문제가 발생한다.

```text
John has 10 apples and gives 3 apples away.
```

현재 구현에서는 다음과 같이 생성된다.

```text
Calculator(10 * 3)
```

하지만 실제 필요한 연산은 다음과 같다.

```text
Calculator(10 - 3)
```

즉, 현재 구현은 **Toolformer의 구조를 이해하기 위한 단순한 Prototype**이다.

아직 다음 기능은 구현되지 않았다.

- 문장의 의미 분석
- 필요한 Operator 판단
- Calculator가 필요한 문장인지 판단
- 여러 API Call Candidate 생성
- Candidate별 Loss 계산
- Loss 기반 Candidate Filtering

---

# 12. GPT-2 기반 Loss 계산

Tool Result가 Language Model의 예측에 도움이 되는지를 확인하기 위해 GPT-2를 이용하여 Loss를 계산하였다.

논문에서는 GPT-J를 사용하지만 현재 단계에서는 Toolformer의 구조를 이해하고 구현하기 위해 상대적으로 작은 GPT-2를 사용한다.

---

## 12.1 Model과 Tokenizer 불러오기

```python
import torch

from transformers import AutoTokenizer
from transformers import AutoModelForCausalLM


tokenizer = AutoTokenizer.from_pretrained("gpt2")

model = AutoModelForCausalLM.from_pretrained("gpt2")

model.eval()
```

### `tokenizer`

텍스트를 Language Model이 처리할 수 있는 Token ID로 변환한다.

```text
Text
 ↓
Token
 ↓
Token ID
```

### `model`

GPT-2 기반 Causal Language Model이다.

다음 Token Prediction을 수행하고 Cross Entropy Loss를 계산할 수 있다.

---

# 13. `calculate_loss()` 구현

```python
def calculate_loss(text):

    inputs = tokenizer(
        text,
        return_tensors="pt"
    )

    with torch.no_grad():

        outputs = model(
            **inputs,
            labels=inputs["input_ids"]
        )

    loss = outputs.loss.item()

    return loss
```

여기서 중요한 부분은 다음 코드이다.

```python
labels=inputs["input_ids"]
```

GPT-2와 같은 Causal Language Model은 내부적으로 Token을 한 칸 Shift하여 다음 Token Prediction Loss를 계산한다.

예를 들어 다음과 같이 동작한다.

```text
Token 1 → Token 2 예측
Token 2 → Token 3 예측
Token 3 → Token 4 예측
```

각 위치에서 정답 Token에 대한 Cross Entropy를 계산하고 전체 Loss를 얻는다.

---

# 14. Loss Filtering 구현

Tool Call이 실제로 도움이 되었는지 확인하기 위해 Tool 사용 전과 후의 Loss를 비교한다.

```python
def filter_api_call(
    loss_without_tool,
    loss_with_tool,
    threshold
):

    improvement = (
        loss_without_tool
        - loss_with_tool
    )

    if improvement >= threshold:
        return "KEEP", improvement

    else:
        return "REMOVE", improvement
```

Loss Improvement는 다음과 같이 계산한다.

```text
Loss Improvement
=
Loss Without Tool
-
Loss With Tool
```

---

## 14.1 KEEP 예시

다음과 같은 결과가 있다고 하자.

```text
Loss Without Tool = 2.81

Loss With Tool = 1.24

Threshold = 1.0
```

Loss Improvement를 계산하면

```text
2.81 - 1.24 = 1.57
```

이다.

```text
1.57 >= 1.0
```

이므로 해당 API Call은 유지한다.

```text
KEEP
```

---

## 14.2 REMOVE 예시

다음과 같은 경우를 생각해보자.

```text
Loss Without Tool = 2.0

Loss With Tool = 1.8

Threshold = 1.0
```

Loss Improvement는

```text
2.0 - 1.8 = 0.2
```

이다.

```text
0.2 < 1.0
```

이므로 Tool Result가 충분한 도움을 주었다고 판단하지 않고 해당 API Call을 제거한다.

```text
REMOVE
```

---

# 15. Loss Filtering의 의미

Toolformer에서 중요한 점은 **Tool을 사용할 수 있다는 이유만으로 모든 API Call을 학습시키지 않는다는 것**이다.

API Call을 실행한 후 실제 Tool Result가 이후 Token Prediction에 도움을 주었는지 평가한다.

```text
Tool 없는 상태
    ↓
Future Token Loss 계산

Tool Result가 있는 상태
    ↓
Future Token Loss 계산

두 Loss 비교
    ↓
Tool이 충분히 도움이 되었는지 판단
```

Loss가 충분히 감소했다면

```text
KEEP
```

Loss가 충분히 감소하지 않았다면

```text
REMOVE
```

한다.

이를 통해 모델에게 실제로 유용한 Tool 사용 예제만 Training Dataset에 남길 수 있다.

---

# 16. 현재까지 구현한 Mini Toolformer 구조

Week 1에서 구현한 내용을 하나로 연결하면 다음과 같다.

```text
Input Text
    ↓
Calculator API Call 생성
    ↓
Calculator 실행
    ↓
Tool Result 생성
    ↓
Language Model Loss 계산
    ↓
Tool 사용 전/후 Loss 비교
    ↓
Loss Improvement 계산
    ↓
Threshold 비교
    ↓
KEEP / REMOVE
```

이를 Toolformer 논문의 구조와 연결하면 다음 부분에 해당한다.

```text
API Call Candidate Generation
            ↓
API Execution
            ↓
Loss-based Filtering
```

---

# 17. 논문과 현재 구현의 차이

현재 프로젝트는 Toolformer 논문의 전체 구조를 그대로 재현한 것이 아니라 **Toolformer의 핵심 Pipeline을 이해하기 위한 Mini Implementation**이다.

| 항목 | Toolformer 논문 | 현재 구현 |
|---|---|---|
| Language Model | GPT-J | GPT-2 |
| Tool 종류 | 여러 Tool | Calculator |
| API Call Candidate 생성 | Language Model | 정규표현식 |
| API Input 생성 | Language Model | 숫자 추출 |
| Tool 선택 | 모델이 판단 | Calculator 고정 |
| 연산 선택 | 모델이 생성 | 현재 `*` 고정 |
| API 실행 | 실제 Tool | Python Calculator |
| Filtering | Loss 기반 | 단순 Loss Difference |
| Training Dataset | 자동 생성 | 아직 미구현 |
| Fine-tuning | 수행 | 아직 미구현 |

현재 목표는 논문의 성능을 완전히 재현하는 것이 아니다.

먼저 작은 구조를 직접 구현하면서

```text
Candidate Generation
→ Tool Execution
→ Filtering
→ Dataset Generation
→ Fine-tuning
```

의 흐름을 이해하는 것을 목표로 한다.

---

# 18. Week 1에서 이해한 Toolformer 핵심

기존 Tool Learning 방식은 다음과 같이 볼 수 있다.

```text
사람이 Tool 사용 데이터를 생성
        ↓
모델 학습
```

Toolformer의 핵심 구조는 다음과 같다.

```text
Language Model이 API Call Candidate 생성
        ↓
실제 Tool 실행
        ↓
Tool Result 획득
        ↓
Tool 사용 전/후 Loss 비교
        ↓
유용한 API Call 선택
        ↓
Training Dataset 생성
        ↓
Fine-tuning
```

따라서 Toolformer의 핵심은 단순히 Tool을 호출하는 것이 아니다.

> **모델이 스스로 Tool 사용 데이터를 만들고, Tool 결과가 실제 Language Modeling에 도움이 되는지를 평가한 뒤 유용한 Tool Call만 다시 학습에 사용하는 구조이다.**

---

# 19. Week 1 최종 정리

Week 1에서는 Toolformer를 구현하기 위해 필요한 **이론적인 Foundation과 최소한의 Tool 사용 구조**를 구현하였다.

현재까지 만든 Mini Pipeline은 다음과 같다.

```text
Text
 ↓
Calculator Call 생성
 ↓
Calculator 실행
 ↓
Tool Result
 ↓
GPT-2 Loss 계산
 ↓
Tool 사용 전/후 Loss 비교
 ↓
KEEP / REMOVE
```

Week 1의 핵심 목표는 Toolformer를 완성하는 것이 아니라,

> **Toolformer가 Tool Call을 어떻게 만들고, Tool 결과가 실제로 유용한지를 어떻게 판단하는지 이해하는 것**

이었다.

Week 2에서는 이 개별 기능들을 실제 Dataset과 연결하여 **API Candidate Generation → Tool Execution → Loss Filtering → Dataset Generation**으로 이어지는 Mini Toolformer Pipeline을 구현한다.
