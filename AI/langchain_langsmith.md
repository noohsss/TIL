# LangChain과 LangSmith

## LangChain

LangChain은 LLM, 프롬프트, 출력 파서, 메모리, Tool 등을 연결해 LLM 애플리케이션을 만들기 위한 Python 프레임워크다.

### 주요 패키지

| 패키지 | 역할 |
|---|---|
| `langchain-core` | Runnable, LCEL, 메시지 등 핵심 기능 |
| `langchain` | 체인, 메모리, 에이전트 등 고수준 기능 |
| `langchain-google-genai` | Gemini 모델 연동 |
| `langchain-community` | 외부 Tool, 벡터 DB 등 연동 |
| `langgraph` | 상태 기반 Agent 구성 |

단순한 API 호출은 직접 작성하는 편이 간단할 수 있다. 프롬프트 재사용, 체인 연결, 메모리, Tool, RAG가 필요해지면 LangChain을 사용하기 편하다.

## ChatModel

`ChatGoogleGenerativeAI`는 Gemini를 LangChain의 ChatModel 형태로 사용할 수 있게 해 준다.

```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.messages import HumanMessage

llm = ChatGoogleGenerativeAI(model="gemini-3.6-flash")
response = llm.invoke([
    HumanMessage(content="Python의 장점 3가지를 알려줘.")
])

print(response.content)
```

`invoke()`의 결과는 문자열이 아니라 `AIMessage`다.

- `content`: 모델 답변
- `usage_metadata`: 입력·출력 토큰 수
- `response_metadata`: 모델명, 종료 이유 등
- `tool_calls`: 모델이 요청한 Tool 목록

## 메시지 타입

- `SystemMessage`: 모델의 역할과 공통 규칙
- `HumanMessage`: 사용자 입력
- `AIMessage`: 모델 응답

## PromptTemplate

프롬프트 안의 바뀌는 부분을 변수로 분리하면 같은 프롬프트를 반복해서 사용할 수 있다.

### `PromptTemplate`

역할 구분이 없는 단일 문자열 프롬프트에 사용한다.

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template(
    "너는 {role} 전문가야. 다음 질문에 답해줘.\n{question}"
)
```

### `ChatPromptTemplate`

system, human, ai처럼 메시지 역할을 나눌 때 사용한다.

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "너는 {role}야. 한국어로 답해."),
    ("human", "{question}"),
])
```

## OutputParser

LLM 응답을 원하는 자료형으로 바꾸는 역할을 한다.
`StrOutputParser`는 `AIMessage`에서 `content`만 꺼내 문자열로 반환한다.

```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()
```

## LCEL과 Runnable

LCEL(LangChain Expression Language)은 `|` 연산자로 구성 요소를 연결하는 문법이다.

```python
chain = prompt | llm | StrOutputParser()
answer = chain.invoke({
    "role": "Python 튜터",
    "question": "decorator가 뭐야?",
})
```

실행 흐름은 다음과 같다.

```text
입력 → PromptTemplate → ChatModel → OutputParser → 출력
```

프롬프트, 모델, 파서, 체인은 모두 `Runnable` 인터페이스를 따른다. 따라서 같은 방식으로 `invoke()`를 호출하고 `|`로 연결할 수 있다. 단, 앞 단계의 출력 자료형이 다음 단계의 입력 조건과 맞아야 한다.

### `RunnableLambda`

일반 Python 함수를 체인의 한 단계로 바꾼다. 문자열을 딕셔너리로 바꾸거나 외부 데이터를 다음 단계에 넘길 때 사용한다.

```python
from langchain_core.runnables import RunnableLambda

to_input = RunnableLambda(lambda text: {"text": text})
chain = first_chain | to_input | second_chain
```

### `RunnablePassthrough`

기존 입력을 그대로 전달하거나, `assign()`으로 값을 추가한다.

```python
from langchain_core.runnables import RunnablePassthrough

add_context = RunnablePassthrough().assign(
    context=lambda data: search(data["question"])
)
qa_chain = add_context | prompt | llm | StrOutputParser()
```

사용자는 질문만 보내고, 체인 내부에서 검색 결과를 `context`에 붙이는 구조에 사용할 수 있다.

## Runnable 실행 방법

| 목적 | 메서드 |
|---|---|
| 입력 하나 실행 | `invoke()` |
| 여러 입력 실행 | `batch()` |
| 응답 스트리밍 | `stream()` |
| 비동기 입력 하나 | `ainvoke()` |
| 비동기 여러 입력 | `abatch()` |
| 비동기 스트리밍 | `astream()` |

`batch()`는 여러 입력을 하나의 HTTP 요청으로 합치는 기능이 아니다. 입력마다 별도 호출을 만들고 동시에 처리한다. `max_concurrency`로 동시에 실행할 수를 제한한다.

```python
results = chain.batch(
    [
        {"question": "Python이 뭐야?"},
        {"question": "FastAPI가 뭐야?"},
    ],
    config={"max_concurrency": 2},
)
```

`stream()`에서는 결과가 chunk 단위로 전달된다. chunk가 항상 토큰 하나인 것은 아니다.

```python
for chunk in chain.stream({"question": "LangChain을 설명해줘."}):
    print(chunk, end="", flush=True)
```

## 토큰과 오류 처리

토큰 사용량은 `AIMessage.usage_metadata`에서 확인한다. 입력과 출력이 길어질수록 비용과 응답 시간이 늘어난다.

```python
response = llm.invoke("Python이 뭐야?")
print(response.usage_metadata)
```

| 오류 | 의미 | 대응 |
|---|---|---|
| `429` | 사용량 또는 요청 제한 | 잠시 기다리거나 재시도 |
| `401` | API 키 인증 실패 | `.env`와 키 권한 확인 |
| `408`, `504` | 응답 지연 | timeout과 재시도 확인 |
| `500` | 서버 오류 | 잠시 후 재시도 또는 fallback |

`max_retries`로 일시적인 오류를 자동 재시도할 수 있다. 계속 실패하면 fallback 모델을 사용하고, 마지막 오류는 `try/except`로 처리한다. API 키나 내부 오류 내용을 사용자에게 그대로 보여주지 않는다.

## 응답 캐싱

개발 중 같은 프롬프트를 반복 실행할 때 캐시를 사용하면 API 호출과 비용을 줄일 수 있다. 같은 입력에 대해 저장된 결과를 재사용하는 방식이다.

```python
from langchain_core.caches import InMemoryCache
from langchain_core.globals import set_llm_cache

set_llm_cache(InMemoryCache())
first = llm.invoke("Python이 뭐야?")  # 모델 호출
second = llm.invoke("Python이 뭐야?")  # 캐시 사용

set_llm_cache(None)
```

`InMemoryCache`는 프로세스가 종료되면 사라진다. 개발 중 반복 호출을 줄이는 용도로 사용한다.

## Prompt Chaining

하나의 큰 작업을 여러 체인으로 나누고 앞 체인의 결과를 다음 체인에 넘긴다.

```python
first_chain = first_prompt | llm | StrOutputParser()
second_chain = second_prompt | llm | StrOutputParser()

result = first_chain.invoke({"topic": "Python"})
final = second_chain.invoke({"text": result})
```

단계별 결과를 확인하기 쉽고, 각 단계의 프롬프트를 따로 수정할 수 있다. 대신 API 호출 횟수와 전체 응답 시간은 늘어날 수 있다.

## 구조화 출력

모델 응답을 Pydantic 모델 형태로 받고 싶을 때 `with_structured_output()`을 사용할 수 있다.

```python
from pydantic import BaseModel, Field

class LanguageInfo(BaseModel):
    language: str
    pros: list[str] = Field(min_length=2, max_length=2)
    cons: list[str] = Field(min_length=2, max_length=2)

structured_llm = llm.with_structured_output(LanguageInfo)
result = structured_llm.invoke("Python의 장단점을 알려줘.")
print(result.language)
```

프롬프트로 JSON 모양만 요청하는 것보다 필드와 자료형을 다루기 쉽다. 그래도 내용이 사실인지, 의미가 정확한지는 별도로 확인해야 한다.

## fallback과 재시도

모델 호출은 네트워크, 인증, 사용량 제한 때문에 실패할 수 있다.

```python
primary = ChatGoogleGenerativeAI(
    model="gemini-3.6-flash",
    max_retries=3,
)
backup = ChatGoogleGenerativeAI(model="gemini-2.5-flash")

safe_llm = primary.with_fallbacks([backup])
```

`max_retries`는 일시적인 오류를 다시 시도하고, 대기 시간이 점점 늘어나는 exponential backoff가 적용될 수 있다. `with_fallbacks()`는 주 모델이 계속 실패할 때 다른 모델로 넘긴다. 인증 오류처럼 설정을 고쳐야 하는 오류는 무조건 재시도하지 않는 편이 낫다.

```python
try:
    result = safe_llm.invoke("안녕하세요")
except Exception:
    print("현재 답변을 생성할 수 없습니다.")
```

## LangSmith

LangSmith는 LangChain 애플리케이션의 실행 과정을 기록하고 분석하는 관측성 도구다.

### 환경 설정

```bash
pip install langsmith
```

`.env`에 다음 값을 설정한다.

```dotenv
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_...
LANGSMITH_PROJECT=langchain-gemini-prac
```

API 키가 여러 workspace에 연결되어 있다면 `LANGSMITH_WORKSPACE_ID`도 설정할 수 있다.

### 기본 추적

Runnable 체인을 실행하면 별도의 callback 코드 없이 실행 기록이 남는다.

```python
result = chain.invoke({"question": "Python decorator가 뭐야?"})
```

LangSmith에서 확인할 수 있는 내용:

- 프롬프트 → Gemini → 파서 실행 순서
- 실제로 완성된 프롬프트
- 입력·출력 내용
- 토큰 사용량과 실행 시간
- 실패한 단계와 오류

### 태그와 메타데이터

`config`를 전달하면 실행 목적이나 실험 버전을 구분하기 쉽다.

```python
result = chain.invoke(
    {"question": "리스트와 튜플의 차이는?"},
    config={
        "run_name": "python-basic-question",
        "tags": ["gemini", "basic"],
        "metadata": {
            "lesson": "langchain",
            "model_family": "gemini",
        },
    },
)
```

### 선택적 추적

전체 실행을 기록하지 않고 특정 구간만 추적할 수 있다.

```python
import langsmith as ls

with ls.tracing_context(
    enabled=True,
    project_name="langchain-gemini-selective",
):
    result = chain.invoke({"question": "제너레이터가 뭐야?"})
```

환경 변수를 바꿨는데 반영되지 않으면 환경 설정을 다시 불러오고 커널을 재시작한다.

## Gemini API와 LangChain의 차이

| 방식 | API | 특징 |
|---|---|---|
| `client.interactions.create()` | Gemini Interactions API | Gemini 기능을 직접 사용 |
| `ChatGoogleGenerativeAI(...).invoke()` | Gemini `generateContent` 계열 | LangChain Runnable·LCEL 사용 |

LangChain은 모델 호출을 통일된 인터페이스로 감싸고 프롬프트, 파서, 메모리, Tool, LangSmith 추적을 연결하기 쉽게 한다.
