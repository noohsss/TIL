# RAG 기초

## RAG란?

RAG(Retrieval-Augmented Generation)는 질문과 관련된 문서를 먼저 검색한 뒤, 검색 결과를 LLM에 전달해 답변을 만드는 방식이다.

모델을 다시 학습하지 않고도 최신 문서, 사내 규정, 비공개 자료를 답변에 활용할 수 있다. 단, 검색 결과가 틀리거나 관련성이 낮으면 답변도 부정확해진다.

```text
인덱싱: 문서 → 로드 → 분할 → 임베딩 → 벡터 저장
질의: 질문 → 질문 임베딩 → 유사 문서 검색 → LLM에 전달 → 답변
```

## Document Loader

문서를 LangChain의 `Document` 객체로 읽어 온다.

| Loader | 대상 |
|---|---|
| `TextLoader` | `.txt` |
| `PyPDFLoader` | `.pdf` |
| `CSVLoader` | `.csv` |
| `WebBaseLoader` | 웹페이지 |

```python
from langchain_community.document_loaders import TextLoader, PyPDFLoader

text_docs = TextLoader("data/rules.txt", encoding="utf-8").load()
pdf_docs = PyPDFLoader("data/manual.pdf").load()
```

`Document`는 본문과 메타데이터를 가진다.

```python
Document(
    page_content="문서 내용",
    metadata={"source": "파일 경로", "page": 0},
)
```

`page_content`는 임베딩할 텍스트이고 `metadata`는 출처, 페이지 등 추적 정보다. 스캔 PDF나 복잡한 표는 로더만으로 텍스트가 완벽히 복원되지 않을 수 있다.

## Text Splitter

긴 문서를 통째로 임베딩하면 검색 정확도가 떨어지고 입력 크기 제한에 걸릴 수 있다. 문서를 작은 청크로 나눠 저장한다.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
)
chunks = splitter.split_documents(text_docs)
```

- `chunk_size`: 청크의 최대 크기
- `chunk_overlap`: 인접 청크가 겹치는 크기

`RecursiveCharacterTextSplitter`는 문단, 줄바꿈, 공백, 문자 순서로 나누며 문맥이 끊기는 것을 줄인다.

문서에 따라 `TokenTextSplitter`, `MarkdownHeaderTextSplitter`, `HTMLHeaderTextSplitter`, `SemanticChunker`, Parent-Child Chunking을 선택할 수 있다.

## Embedding

텍스트를 숫자 벡터로 바꾸는 과정이다. 의미가 비슷한 문장은 벡터 공간에서도 가까운 위치에 놓이는 경향이 있다.

```python
from langchain_google_genai import GoogleGenerativeAIEmbeddings

embeddings = GoogleGenerativeAIEmbeddings(model="gemini-embedding-2")

query_vector = embeddings.embed_query("휴가 신청 방법")
document_vectors = embeddings.embed_documents([
    "연차 신청은 그룹웨어에서 합니다.",
    "재택근무는 팀장 승인이 필요합니다.",
])
```

- `embed_query()`: 검색 질문 하나를 벡터로 변환
- `embed_documents()`: 문서 여러 개를 벡터로 변환

질문과 문서는 같은 임베딩 모델과 차원 설정을 사용해야 비교할 수 있다.

## 유사도 검색

### 코사인 유사도

두 벡터가 같은 방향을 향하는 정도를 계산한다. 일반적인 공식은 다음과 같다.

```text
cosine_similarity(A, B) = (A · B) / (|A| × |B|)
```

값이 1에 가까울수록 방향이 비슷하다. 다만 특정 점수 이상이면 무조건 관련 있다는 공통 기준은 없으므로 실제 데이터로 기준을 정해야 한다.

```python
def cosine_similarity(a, b):
    if len(a) != len(b):
        raise ValueError("벡터 차원이 다릅니다.")

    dot = sum(x * y for x, y in zip(a, b))
    norm_a = sum(x * x for x in a) ** 0.5
    norm_b = sum(y * y for y in b) ** 0.5
    if norm_a == 0 or norm_b == 0:
        raise ValueError("0 벡터는 비교할 수 없습니다.")
    return dot / (norm_a * norm_b)
```

문서 벡터와 질문 벡터의 유사도를 계산한 뒤 점수가 높은 문서를 선택한다.

```python
scores = []
for document, vector in zip(documents, document_vectors):
    score = cosine_similarity(query_vector, vector)
    scores.append((score, document))

scores.sort(key=lambda item: item[0], reverse=True)
top_documents = scores[:2]
```

## RAG에서 확인할 것

- 문서가 제대로 로드됐는지
- 청크 크기와 겹침이 문서에 맞는지
- 질문과 문서에 같은 임베딩 모델을 사용했는지
- 검색 결과가 질문과 실제로 관련 있는지
- 답변에 출처 metadata를 함께 전달할지
- 검색 결과가 없을 때 어떻게 답할지

RAG는 검색 품질이 핵심이다. 검색된 문서가 답변의 근거가 되므로, LLM 프롬프트보다 먼저 로더·청킹·임베딩·검색 결과를 확인한다.

## Vector DB

임베딩 벡터와 원본 텍스트, metadata를 저장하고 유사도 검색을 수행하는 저장소다.

일반 DB가 정확한 값이나 범위 검색에 강하다면, Vector DB는 질문과 의미가 가까운 벡터를 찾는 데 사용한다. 대규모 데이터에서는 모든 벡터를 비교하지 않고 ANN(Approximate Nearest Neighbor) 인덱스를 사용해 검색 속도를 높인다.

```text
저장: 문서 → 청크 → 임베딩 → 벡터 + 원문 + metadata 저장
검색: 질문 → 임베딩 → 유사 벡터 검색 → 원문 반환
```

### Chroma 사용

```bash
pip install langchain-chroma
```

```python
from langchain_chroma import Chroma

vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    collection_name="company_rules",
    persist_directory="./chroma_db",
)
```

`persist_directory`를 지정하면 로컬 디스크에 저장되어 다시 연결할 수 있다.

```python
existing_store = Chroma(
    embedding_function=embeddings,
    collection_name="company_rules",
    persist_directory="./chroma_db",
)
```

임베딩 모델을 바꾸면 벡터 공간과 차원이 달라질 수 있으므로 기존 DB를 그대로 섞어 쓰지 말고 다시 구축한다.

### Metadata

벡터 DB의 한 청크는 보통 다음 세 가지로 구성된다.

- 벡터: 유사도 검색에 사용
- 원본 텍스트: LLM에 전달할 내용
- metadata: 출처, 페이지, 문서 분류, 사용자 ID 등

metadata는 임베딩되지 않는다. 의미 검색 대상이 아니라 필터링과 출처 확인에 사용한다.

```python
results = vectorstore.similarity_search(
    "보안 규정은?",
    k=3,
    filter={"category": "보안"},
)
```

반드시 지켜야 하는 날짜, 권한, 사용자 범위는 프롬프트가 아니라 metadata filter로 제한하는 편이 안전하다.

### 유사도 점수

Chroma의 `similarity_search_with_score()`는 기본 설정에서 L2 거리를 반환할 수 있다. 이 경우 값이 작을수록 가깝다. 점수의 의미와 방향은 벡터 DB의 metric 설정을 확인해야 한다.

### Retriever

Vector Store를 체인에 연결하려면 Retriever로 변환한다.

```python
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 4}
)
docs = retriever.invoke("재택근무 규정은?")
```

`Retriever.invoke(질문)`은 관련 `Document` 목록을 반환한다.

## RAG Chain

검색 결과를 `context`로 만들고 프롬프트에 넣어 LLM을 호출한다.

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

prompt = ChatPromptTemplate.from_template(
    "다음 context만 사용해서 질문에 답해줘.\n"
    "모르는 내용은 모른다고 답해.\n\n"
    "context:\n{context}\n\n질문: {question}"
)

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain = (
    {
        "context": retriever | format_docs,
        "question": RunnablePassthrough(),
    }
    | prompt
    | llm
    | StrOutputParser()
)

answer = rag_chain.invoke("연차는 어디에서 신청해?")
```

검색 결과의 metadata를 함께 포맷하면 출처를 답변에 표시할 수 있다.

## 고급 검색 전략

### MMR

Similarity Search는 질문과 가까운 문서를 고르지만 결과가 서로 비슷할 수 있다. MMR(Maximal Marginal Relevance)은 관련성과 결과 간 다양성을 함께 고려한다.

```python
mmr_docs = vectorstore.max_marginal_relevance_search(
    question,
    k=6,
    fetch_k=20,
    lambda_mult=0.5,
)
```

- `k`: 최종 반환 개수
- `fetch_k`: 후보로 검토할 개수
- `lambda_mult`: 1에 가까울수록 관련성, 0에 가까울수록 다양성 중시

여러 관점이 필요한 동향 요약에는 유용하지만, 정답 문서 하나를 찾는 질문에는 기본 유사도 검색이 더 나을 수 있다.

### BM25

문서와 질문에 실제로 등장하는 단어를 비교하는 키워드 검색이다. 제품명, 모델명, 코드, 약어처럼 철자가 중요한 검색에 강하다.

```python
from langchain_community.retrievers import BM25Retriever

bm25 = BM25Retriever.from_documents(
    documents,
    k=5,
)
keyword_docs = bm25.invoke("Qwen3-Next 모델")
```

한국어는 조사와 어미 때문에 단순 공백 토큰화의 한계가 있다. 형태소 분석기를 사용하면 문서와 질문에 같은 전처리를 적용할 수 있다.

### Metadata Filter

본문 의미가 아니라 metadata 조건으로 검색 대상을 먼저 제한한다.

```python
docs = vectorstore.similarity_search(
    "AI 정책 동향",
    k=5,
    filter={"month": 11},
)
```

필터는 관련성 점수를 높이는 기능이 아니라 검색 범위를 강제하는 기능이다. metadata가 누락되거나 잘못 저장되면 필요한 문서가 제외될 수 있다.

### Hybrid Search

벡터 검색과 BM25를 함께 사용한다. 벡터 검색은 표현이 달라도 의미가 비슷한 문서를 찾고, BM25는 정확한 단어를 찾는다.

```python
from langchain.retrievers import EnsembleRetriever

hybrid = EnsembleRetriever(
    retrievers=[vector_retriever, bm25],
    weights=[0.5, 0.5],
)
docs = hybrid.invoke("Qwen3-Next 성능 개선")
```

검색 방식마다 점수 범위가 다르므로 점수를 단순히 더하지 않고 RRF(Reciprocal Rank Fusion)처럼 순위를 결합한다.

### Re-ranking

먼저 검색한 후보 문서를 질문과 다시 비교해 순서를 바꾸는 단계다.

```text
빠른 검색 → 후보 문서 여러 개 → 관련성 재평가 → 상위 문서 선택
```

LLM으로 후보를 재평가하면 유연하지만 후보마다 추가 호출이 필요해 비용과 시간이 늘어난다. 이미 검색된 후보의 순서만 바꿀 수 있고, 처음 검색에서 빠진 문서를 되살리지는 못한다.

## 검색 품질 확인

검색 방법은 모두 적용하는 것이 아니라 문제에 맞게 선택한다.

| 문제 | 방법 |
|---|---|
| 비슷한 문서가 반복됨 | MMR |
| 정확한 모델명·코드가 중요함 | BM25 |
| 날짜·권한·분류를 제한해야 함 | Metadata Filter |
| 의미 검색과 키워드 검색을 같이 사용 | Hybrid Search |
| 관련 문서는 찾았지만 순서가 아쉬움 | Re-ranking |

평가할 때는 한 질문의 결과만 보고 판단하지 않는다. 여러 질문에 대해 관련 문서 포함 여부, 순위, 중복, 출처를 비교한다. 청크 크기, `k`, `fetch_k`, `lambda_mult`, 토큰화 방식도 함께 기록한다.
