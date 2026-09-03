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
