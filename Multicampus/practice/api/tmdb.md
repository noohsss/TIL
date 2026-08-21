# TMDB API 활용 예제

## 함수 정의

```python
import requests
from pprint import pprint
import os
from dotenv import load_dotenv

load_dotenv()

def get_movies(what="now_playing", page=1):
    url = f"https://api.themoviedb.org/3/movie/{what}"

    token = os.getenv("TMDB_TOKEN")

    my_params = {
        "language": "ko-kr",
        "page": page
    }

    my_headers = {
        "Authorization": f"Bearer {token}",
    }

    response = requests.get(url, headers=my_headers, params=my_params)

    response.raise_for_status()

    data = response.json()

    results = data.get("results")
    return results

def get_best_movie(movies):
    best_movie = max(movies, key=lambda movie: movie["vote_average"])

    return best_movie

def get_simple_movie(movie):
    title = movie["title"]
    vote_average = movie["vote_average"]
    release_date = movie["release_date"]

    simple_movie = {
        "title": title,
        "vote_average": vote_average,
        "release_date": release_date,
    }
    return simple_movie

def get_movie_by_id(movie_id):
    url = f"https://api.themoviedb.org/3/movie/{movie_id}"

    token = os.getenv("TMDB_TOKEN")

    my_params = {
    }

    my_headers = {
        "Authorization": f"Bearer {token}",
    }

    response = requests.get(url, headers=my_headers, params=my_params)

    response.raise_for_status()

    data = response.json()

    return data

def fetch_tmdb_api(path, my_params={}):
    url = f"https://api.themoviedb.org/3/{path}"

    token = os.getenv("TMDB_TOKEN")
    my_headers = {
        "Authorization": f"Bearer {token}",
    }

    response = requests.get(url, headers=my_headers, params=my_params)

    response.raise_for_status()

    data = response.json()
    return data
```

## 문제 1

현재 상영 중인 영화 중 평점이 가장 높은 영화 출력

```python
try:
    movies = get_movies()

    best_movie = get_best_movie(movies)
    print(best_movie["title"], best_movie["vote_average"])
except Exception as e:
    print("오류 발생!")
    print(e)
```

## 문제 2

인기 영화 중 평점이 가장 높은 영화 출력

```python
try:
    movies = get_movies("popular")

    best_movie = get_best_movie(movies)
    print(best_movie["title"], best_movie["vote_average"])
except Exception as e:
    print("오류 발생!")
    print(e)
```

## 문제 3

현재 상영 중인 영화에서 제목, 평점, 개봉일만 추출

```python
try:
    movies = get_movies()

    simple_movies = [get_simple_movie(movie) for movie in movies]

    pprint(simple_movies)
except Exception as e:
    print(e)
```

## 문제 4

현재 상영 중인 영화 1~4페이지를 합친 뒤 평점순으로 정렬

```python
try:
    pages = [1, 2, 3, 4]
    many_movies = []
    for page in pages:
        movies = get_movies(page=page)
        many_movies.extend(movies)

    sorted_movies = sorted(
        many_movies,
        key=lambda movie: movie["vote_average"],
        reverse=True
    )

    sorted_movies_title = [
        (movie["title"], movie["vote_average"])
        for movie in sorted_movies
    ]

    pprint(sorted_movies_title)

except Exception as e:
    print(e)
```

## 문제 5

평점이 가장 높은 영화의 수익 조회

```python
try:
    movies = get_movies()
    movie = get_best_movie(movies)

    movie = movies[0]

    movie_id = movie.get("id")
    movie_detail = get_movie_by_id(movie_id)

    revenue = movie_detail.get("revenue")

    print(revenue)

except Exception as e:
    print(e)
```

## 문제 6

평점이 가장 높은 영화의 포스터 경로 조회

```python
try:
    movies = get_movies()
    movie = get_best_movie(movies)
    pprint(movie)

    pprint(movie["poster_path"])

except Exception as e:
    print(e)
```

## 문제 7

평점이 가장 높은 영화의 배우 정보 조회

```python
try:
    movies = get_movies()

    best_movie = get_best_movie(movies)

    movie_id = best_movie["id"]

    data = fetch_tmdb_api(f"movie/{movie_id}/credits")

    cast = data.get("cast")

    cast_name = [item.get("original_name") for item in cast]
    print(cast_name)

except Exception as e:
    print("오류발생!")
    print(e)
```

## 문제 8

현재 상영 중인 영화의 제목과 장르명 추출

```python
try:
    movies = get_movies()

    genres_data = fetch_tmdb_api("genre/movie/list")
    genres = genres_data.get("genres")

    my_movie_list = []

    count = 0

    for movie in movies:
        movie_genre_ids = movie.get("genre_ids")

        movie_genre_list = []
        for genre_id in movie_genre_ids:
            for genre in genres:
                if genre["id"] == genre_id:
                    movie_genre_list.append(genre["name"])
                    break
                count += 1

        my_movie_list.append((movie.get("title"), movie_genre_list))
    print(count)
    pprint(my_movie_list)

except Exception as e:
    print("오류발생!")
    print(e)
```

### 장르 매핑 딕셔너리 사용

```python
try:
    movies = get_movies()

    genres_data = fetch_tmdb_api("genre/movie/list")
    genres = genres_data.get("genres")
    my_movie_list = []

    count = 0

    genre_mapping = {}

    for genre in genres:
        genre_id = genre.get("id")
        genre_name = genre.get("name")

        genre_mapping[genre_id] = genre_name

        count += 1
    pprint(genre_mapping)

    my_movie_list = []

    for movie in movies:
        movie_genre_ids = movie.get("genre_ids")

        movie_genre_list = []
        for genre_id in movie_genre_ids:
            genre_name = genre_mapping[genre_id]
            movie_genre_list.append(genre_name)

            count += 1
        my_movie_list.append((movie.get("title"), movie_genre_list))

    print(count)
    pprint(my_movie_list)

except Exception as e:
    print("오류발생!")
    print(e)
```
