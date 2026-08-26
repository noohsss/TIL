# Gemini SDK

`google-genai` SDK를 사용하면 Python에서 Gemini에 요청을 보낼 수 있다.

## 기본 설정

```bash
pip install google-genai python-dotenv
```

`.env`에 API 키를 저장하고 코드에서 불러온다.

```python
import os
from dotenv import load_dotenv
from google import genai

load_dotenv()
api_key = os.getenv("GEMINI_API_KEY")
model = os.getenv("GEMINI_MODEL", "gemini-3.6-flash")

client = genai.Client(api_key=api_key)
```

## 요청 보내기

```python
response = client.interactions.create(
    model=model,
    input="LLM을 한 문장으로 설명해줘.",
    store=False,
)

print(response.output_text)
```

- `client`: Gemini 클라이언트
- `interactions.create()`: API 요청
- `output_text`: 최종 답변
- `steps`: 요청과 응답 과정

## 시스템 지시

`system_instruction`에 모델의 역할과 답변 규칙을 적는다. 실제 질문은 `input`에 넣는다.

```python
response = client.interactions.create(
    model=model,
    input="API 키를 코드에 적으면 왜 위험해?",
    system_instruction="보안 담당자처럼 짧게 답해.",
    store=False,
)
```

## 대화 이어가기

### 직접 history 관리

응답의 steps를 history에 넣고 다음 질문을 추가한다.

```python
history.extend(step.model_dump(exclude_none=True) for step in first.steps)
history.append({
    "type": "user_input",
    "content": [{"type": "text", "text": "내 이름이 뭐야?"}],
})
```

### `store=True`

서버에 interaction을 저장하고 ID로 대화를 이어간다.

```python
next_response = client.interactions.create(
    model=model,
    input="이름이 뭐야?",
    previous_interaction_id=first.id,
    store=True,
)
```

이전 interaction ID를 다시 사용하면 그 시점에서 새 대화로 분기할 수 있다.

## 생성 설정과 오류

```python
generation_config={
    "max_output_tokens": 1000,
    "thinking_level": "high",
}
```

- `ClientError`: API 키, 모델, 요청 형식, 사용량 문제
- `ServerError`: 서버 문제

오류 종류와 상태 코드를 확인한 뒤 API 키·모델·요청을 수정하거나 다시 요청한다.

## Interactions API의 `input`

`input`에 넣는 값은 문자열, Content 배열, Step 배열로 나뉜다.

| 형태 | 용도 | 주요 `type` |
|---|---|---|
| 문자열 | 간단한 텍스트 요청 | 없음 |
| Content 배열 | 한 번의 요청에 여러 콘텐츠 전달 | `text`, `image`, `audio`, `document` |
| Step 배열 | 대화 이력 직접 구성 | `user_input`, `model_output` 등 |

`user_input`은 대화의 단계(Step)이고, `text`나 `image`는 단계 안의 내용(Content)이다.

### 문자열

간단한 질문을 보낼 때 사용한다.

```python
input="LLM을 한 문장으로 설명해 줘."
```

### Content 배열

한 번의 요청에 여러 콘텐츠를 넣을 때 사용한다. `text`, `image`, `audio`, `document` 등을 넣을 수 있다.

```python
input=[
    {"type": "text", "text": "다음 단어를 묶어 줘."},
    {"type": "text", "text": "사과, 바나나"},
]
```

배열 전체는 하나의 `user_input` Step으로 처리된다.

### Step 배열

대화 기록을 직접 보낼 때 사용한다.

```python
history = [
    {
        "type": "user_input",
        "content": [
            {"type": "text", "text": "내 이름은 민수야."}
        ],
    }
]

first_turn = client.interactions.create(
    model=model,
    input=history,
    store=False,
)
```

모델 응답의 `steps`를 history에 추가한 뒤 다음 `user_input`을 붙인다.
응답 Step은 임의로 다시 만들지 말고 그대로 전달한다.

```python
history.extend(step.model_dump() for step in first_turn.steps)
history.append({
    "type": "user_input",
    "content": [
        {"type": "text", "text": "내 이름이 뭐였지?"}
    ],
})

second_turn = client.interactions.create(
    model=model,
    input=history,
    store=False,
)
```

### 정리

- 간단한 질문: 문자열
- 한 번의 요청에 여러 내용: Content 배열
- 대화 이력 직접 관리: Step 배열
- `user_input`: 대화 단계
- `content`: 단계 안의 내용
