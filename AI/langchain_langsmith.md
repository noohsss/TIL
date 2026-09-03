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

비동기 코드에서는 `ainvoke()`, `abatch()`, `astream()`을 사용한다. 같은 Runnable에 여러 입력을 적용할 때는 `abatch()`가 편하고, 서로 다른 비동기 작업을 조합할 때는 `asyncio.gather()`를 사용할 수 있다.

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

## 대화 메모리

LLM 호출은 기본적으로 이전 요청을 기억하지 않는다. 이전 메시지를 다음 호출에도 함께 보내야 한다.

```python
messages = [
    SystemMessage(content="너는 친절한 상담사야."),
    HumanMessage(content="내 이름은 철수야."),
]

response = llm.invoke(messages)
messages.extend([
    response,
    HumanMessage(content="내 이름이 뭐야?"),
])
response = llm.invoke(messages)
```

### `InMemoryChatMessageHistory`

메시지를 세션별로 보관하는 LangChain Core 구현이다.

```python
from langchain_core.chat_history import InMemoryChatMessageHistory

history_store = {}

def get_history(session_id):
    if session_id not in history_store:
        history_store[session_id] = InMemoryChatMessageHistory()
    return history_store[session_id]

def chat(session_id: str, user_input: str) -> str:
    history = get_history(session_id)
    user_message = HumanMessage(content=user_input)
    response = llm.invoke([
        SystemMessage(content="너는 친절한 상담사야."),
        *history.messages,
        user_message,
    ])
    history.add_messages([user_message, response])
    return response.content
```

같은 session ID는 같은 대화를 사용하고, 다른 ID는 별도 대화가 된다. 인메모리 저장소는 프로세스가 종료되면 사라지므로 실제 서비스에서는 DB나 checkpointer가 필요하다.

### Context window 관리

전체 대화를 저장하는 것과 모델에 전체 대화를 보내는 것은 별개다. 대화가 길어지면 토큰 비용과 지연 시간이 늘어나므로 전달할 메시지를 줄인다.

- 트리밍: 최근 메시지만 전달
- 요약: 오래된 대화를 요약해서 전달
- 검색: 필요한 과거 정보만 검색해서 전달

```python
from langchain_core.messages import trim_messages

trimmer = trim_messages(
    max_tokens=60,
    strategy="last",
    token_counter=llm,
    include_system=True,
    start_on="human",
)
trimmed = trimmer.invoke(history.messages)
response = llm.invoke(trimmed)
```

`trim_messages()`는 원본 history를 삭제하지 않고, 모델에 전달할 새 메시지 목록만 만든다.

## Structured Output과 Output Parser

모델의 응답을 문자열이 아닌 Python 자료형이나 Pydantic 객체로 받는 방법이다.

| 방법 | 결과 | 용도 |
|---|---|---|
| `StrOutputParser` | `str` | 일반 텍스트 |
| `CommaSeparatedListOutputParser` | `list[str]` | 단순 목록 |
| `JsonOutputParser` | `dict` 또는 `list` | 유연한 JSON |
| `PydanticOutputParser` | Pydantic 객체 | schema 검증 |
| `with_structured_output()` | Pydantic 객체 등 | 모델의 네이티브 구조화 출력 |

### PydanticOutputParser

`get_format_instructions()`로 출력 규칙을 프롬프트에 넣고, 모델의 문자열 응답을 Pydantic으로 변환한다.

```python
from typing import Literal
from pydantic import BaseModel, Field
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import PromptTemplate

class SentimentResult(BaseModel):
    sentiment: Literal["긍정", "부정", "중립"]
    intensity: int = Field(ge=1, le=5)
    reason: str = Field(min_length=1)

parser = PydanticOutputParser(pydantic_object=SentimentResult)
prompt = PromptTemplate.from_template(
    "다음 문장의 감정을 분석해줘.\n문장: {text}\n{format_instructions}"
).partial(format_instructions=parser.get_format_instructions())

chain = prompt | llm | parser
result = chain.invoke({"text": "정말 만족스러운 기능입니다."})
```

`Literal`은 허용값을 제한하고, `Field`는 숫자 범위나 문자열 길이를 제한한다. 프롬프트의 설명만으로는 값을 강제할 수 없고, parser가 형식 또는 schema 오류를 확인한다.

### `with_structured_output()`

지원되는 모델에서는 schema를 모델 호출에 직접 전달해 Pydantic 객체를 받을 수 있다.

```python
structured_llm = llm.with_structured_output(SentimentResult)
result = structured_llm.invoke("배송은 빠르지만 포장이 찢어져 아쉬웠다.")
```

네이티브 구조화 출력을 지원하지 않는 모델이나 원본 JSON 문자열을 직접 다뤄야 하는 경우에는 `PydanticOutputParser`가 적합하다.

검증은 다음 순서로 나눠 생각한다.

```text
JSON 문법 → schema와 타입 → 업무 규칙 → 서비스 정책
```

Pydantic은 주로 앞의 두 단계를 처리한다. 값이 문법에 맞더라도 사실인지, 업무상 올바른지는 별도 검증이 필요하다.

## Tool과 Agent

Tool은 LLM이 외부 세계와 상호작용할 수 있도록 만든 함수다. 모델이 직접 함수를 실행하는 것은 아니고, 호출할 Tool과 인자를 반환하면 애플리케이션이 실행한다.

```text
사용자 질문 → 모델 판단 → tool_call
→ 애플리케이션에서 함수 실행 → ToolMessage 전달 → 최종 답변
```

### `@tool`

일반 Python 함수를 LangChain Tool로 바꾼다. 함수 이름, 타입 힌트, docstring이 모델에 전달되는 설명이 된다.

```python
from langchain_core.tools import tool

@tool(parse_docstring=True)
def search_weather(city: str) -> str:
    """도시의 현재 날씨를 검색한다.

    Args:
        city: 날씨를 검색할 도시 이름
    """
    return f"{city}의 날씨 정보"
```

Tool 이름은 명확하게 짓고, 설명·인자 타입·예시·오류 결과를 구체적으로 작성한다.

### Tool 바인딩과 실행

```python
llm_with_tools = llm.bind_tools([search_weather])
response = llm_with_tools.invoke("서울 날씨를 알려줘.")

for tool_call in response.tool_calls:
    result = search_weather.invoke(tool_call["args"])
    messages.append(response)
    messages.append(
        ToolMessage(
            content=str(result),
            tool_call_id=tool_call["id"],
        )
    )
```

`bind_tools()`는 Tool 목록을 모델에 알려줄 뿐 실제 함수를 실행하지 않는다. `AIMessage.tool_calls`를 확인하고 등록된 함수만 실행해야 한다.

### Agent loop

Tool 호출이 끝날 때까지 모델 호출과 함수 실행을 반복한다. 한 번에 여러 Tool을 요청할 수 있으므로 모든 호출을 처리한다.

```python
for _ in range(max_attempt):
    response = llm_with_tools.invoke(messages)
    messages.append(response)

    if not response.tool_calls:
        return response.content

    for tool_call in response.tool_calls:
        tool = tool_map[tool_call["name"]]
        result = tool.invoke(tool_call["args"])
        messages.append(ToolMessage(
            content=str(result),
            tool_call_id=tool_call["id"],
        ))
```

최대 반복 횟수를 두어 무한 호출을 막는다. Tool의 이름과 허용 목록을 확인하고, 실행 오류는 모델이 이해할 수 있는 결과로 돌려준다.
