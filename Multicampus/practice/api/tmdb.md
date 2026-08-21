# TMDB API 학습 정리

## 1. 준비 및 인증

TMDB(The Movie Database) API는 영화 목록, 상세 정보, 출연진 등의 데이터를 제공한다. 파이썬에서는 `requests`로 HTTP 요청을 보내고 응답 JSON을 처리한다.

```bash
pip install requests python-dotenv
```

API 토큰은 소스 코드에 직접 작성하지 않고 프로젝트 루트의 `.env`에 저장한다.

```dotenv
TMDB_TOKEN=발급받은_토큰
```

```python
import os
from dotenv import load_dotenv

load_dotenv()
token = os.getenv("TMDB_TOKEN")
```

Bearer 토큰 인증은 요청 헤더에 넣는다.

```python
headers = {"Authorization": f"Bearer {token}"}
```

## 2. 영화 목록 조회

현재 상영 중인 영화는 `movie/now_playing`, 인기 영화는 `movie/popular` 엔드포인트로 조회한다.

```python
import requests

BASE_URL = "https://api.themoviedb.org/3"

def get_movies(category="now_playing", page=1):
    response = requests.get(
        f"{BASE_URL}/movie/{category}",
        headers={"Authorization": f"Bearer {token}"},
        params={"language": "ko-KR", "page": page},
    )
    response.raise_for_status()
    return response.json().get("results", [])
```

- `params`: 쿼리 파라미터를 딕셔너리로 전달한다.
- `language`: 응답 언어를 지정한다.
- `page`: 여러 페이지의 데이터를 조회할 때 사용한다.
- 영화 목록 데이터는 응답 JSON의 `results`에 담긴다.

## 3. 응답 오류 처리

`raise_for_status()`는 상태 코드가 4xx 또는 5xx이면 예외를 발생시킨다. API 호출 함수에서는 오류를 발생시키고, 실제 실행부에서 `try-except`로 처리할 수 있다.

```python
try:
    movies = get_movies()
except requests.exceptions.RequestException as error:
    print(f"API 요청 실패: {error}")
```

`pprint()`를 사용하면 중첩된 딕셔너리나 리스트를 읽기 쉽게 출력할 수 있다.

```python
from pprint import pprint

pprint(movies)
```

## 4. 평점이 가장 높은 영화 찾기

`max()`의 `key`에 평점 기준을 지정하면 가장 높은 평점의 영화를 찾을 수 있다.

```python
def get_best_movie(movies):
    return max(movies, key=lambda movie: movie["vote_average"])

movies = get_movies("popular")
best_movie = get_best_movie(movies)
print(best_movie["title"], best_movie["vote_average"])
```

## 5. 필요한 정보만 추출하기

API 응답 전체를 그대로 사용하지 않고 필요한 필드만 새로운 딕셔너리로 구성할 수 있다.

```python
def get_simple_movie(movie):
    return {
        "title": movie["title"],
        "vote_average": movie["vote_average"],
        "release_date": movie["release_date"],
    }

simple_movies = [get_simple_movie(movie) for movie in movies]
```

리스트 컴프리헨션은 목록의 각 요소에 같은 변환을 적용할 때 유용하다.

## 6. 여러 페이지 조회 및 정렬

한 페이지의 결과만으로 부족하면 페이지를 반복 조회하여 하나의 리스트로 합친다. `extend()`는 리스트에 여러 요소를 추가한다.

```python
many_movies = []
for page in range(1, 5):
    many_movies.extend(get_movies(page=page))

sorted_movies = sorted(
    many_movies,
    key=lambda movie: movie["vote_average"],
    reverse=True,
)
movie_titles = [
    (movie["title"], movie["vote_average"])
    for movie in sorted_movies
]
```

- `sorted()`는 원본을 변경하지 않고 정렬된 새 리스트를 반환한다.
- `reverse=True`를 사용하면 내림차순으로 정렬된다.
- 여러 페이지를 요청하면 요청 횟수와 API 제한을 고려해야 한다.

## 7. 영화 상세 정보 조회

목록 응답에는 영화 ID가 포함되어 있다. 이 ID를 상세 조회 엔드포인트에 전달하면 수익(`revenue`) 같은 추가 정보를 얻을 수 있다.

```python
def get_movie_by_id(movie_id):
    response = requests.get(
        f"{BASE_URL}/movie/{movie_id}",
        headers={"Authorization": f"Bearer {token}"},
        params={"language": "ko-KR"},
    )
    response.raise_for_status()
    return response.json()

best_movie = get_best_movie(get_movies())
movie_detail = get_movie_by_id(best_movie["id"])
print(movie_detail.get("revenue"))
```

`poster_path`는 포스터 이미지의 상대 경로다. 이미지 URL을 만들 때는 TMDB 이미지 기본 주소와 결합한다.

```python
image_url = f"https://image.tmdb.org/t/p/w500{best_movie['poster_path']}"
```

## 8. 출연진(Credits) 조회

영화 ID를 사용해 `movie/{movie_id}/credits` 엔드포인트를 호출한다.

```python
def fetch_tmdb_api(path, params=None):
    response = requests.get(
        f"{BASE_URL}/{path}",
        headers={"Authorization": f"Bearer {token}"},
        params=params or {},
    )
    response.raise_for_status()
    return response.json()

best_movie = get_best_movie(get_movies())
credits = fetch_tmdb_api(f"movie/{best_movie['id']}/credits")
cast_names = [person["original_name"] for person in credits.get("cast", [])]
```

공통 API 호출 함수를 만들면 엔드포인트가 달라져도 인증, 요청, 오류 처리를 반복해서 작성하지 않아도 된다.

## 9. 장르 ID를 장르명으로 변환하기

영화 목록에는 장르명이 아니라 `genre_ids`가 들어 있다. `genre/movie/list`에서 장르 목록을 한 번 가져온 뒤, ID와 이름의 매핑 딕셔너리를 만든다.

```python
genres = fetch_tmdb_api("genre/movie/list", {"language": "ko-KR"})["genres"]
genre_mapping = {genre["id"]: genre["name"] for genre in genres}

movie_list = []
for movie in get_movies():
    movie_genres = [
        genre_mapping[genre_id]
        for genre_id in movie.get("genre_ids", [])
    ]
    movie_list.append({
        "title": movie["title"],
        "genres": movie_genres,
    })
```

영화마다 장르 목록을 순회하며 ID를 찾는 대신 매핑 딕셔너리를 사용하면 불필요한 반복을 줄일 수 있다.

## 핵심 정리

- 토큰과 같은 민감한 정보는 `.env`와 환경변수로 관리한다.
- `requests.get()`의 `headers`로 인증하고 `params`로 쿼리 파라미터를 전달한다.
- `raise_for_status()`로 HTTP 오류를 예외 처리한다.
- 목록의 `results`, 상세 정보의 `id`, 크레딧의 `cast` 등 응답 구조를 확인하며 데이터를 꺼낸다.
- `max()`, `sorted()`, 리스트 컴프리헨션, `lambda`로 데이터를 가공한다.
- 반복 호출을 줄이고, 공통 요청 함수와 ID-이름 매핑 딕셔너리를 활용한다.

