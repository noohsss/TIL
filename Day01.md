# ⌨️ TIL - Git/Terminal 기본 명령어 및 Markdown 문법

## ❗️ Git 작업 흐름

```text
Working Directory
      │
  git add
      ▼
Staging Area
      │
 git commit
      ▼
Local Repository
      │
  git push
      ▼
Remote Repository
```

---

## ❗️ 자주 사용하는 명령어

| 명령어 | 설명 |
|--------|------|
| `git init` | Git 저장소 생성 |
| `git status` | 현재 상태 확인 |
| `git add .` | 모든 변경 사항을 Staging Area에 추가 |
| `git commit -m {commit message}` | 변경 사항을 Commit |
| `git log --oneline` | Commit 기록 확인 |
| `git remote origin {URL}` | 원격 저장소 등록 |
| `git remote -v` | 원격 저장소 확인 |
| `git push` | 원격 저장소에 업로드 |
| `git clone {URL}` | 원격 저장소 복제 |

---

## ❗️ 기본 작업 순서

```bash
# 저장소 생성
git init

# 변경 사항 확인
git status

# Staging Area에 추가
git add .

# Commit
git commit -m {commit message}

# 원격 저장소 연동
git remote origin {URL}

# 원격 저장소 업로드
git push
```

---

## ❗️Terminal 기본 명령어

| 명령어 | 설명 |
|--------|------|
| `mkdir {폴더명}` | 새 폴더 생성 |
| `ls` | 현재 폴더의 파일 및 폴더 목록 조회 |
| `cd {경로/폴더명}` | 해당 경로의 폴더로 이동 |
| `rm {파일명}` | 파일 삭제 |
| `rm -r {폴더명}` | 폴더와 내부 파일 삭제 |
| `start {파일명}` / `open {파일명}` | 파일 실행 및 열기 |

---

## ❗️사용 예시

```bash
# project 폴더 생성
mkdir project

# 현재 폴더 목록 확인
ls

# project 폴더로 이동
cd project

# 상위 폴더로 이동
cd ..

# 파일 삭제
rm test.txt

# 폴더 삭제
rm -r project

# 파일 열기
start test.txt   # Windows
open test.txt    # macOS
```

---

## ❗️ Markdown 문법

### 1. 제목

```text
# 제목 1
## 제목 2
### 제목 3
```

---

### 2. 목록

-  순서 없는 목록

```text
- 항목
- 항목
  - 하위 항목
```

-  순서 있는 목록

```text
1. 첫 번째
2. 두 번째
```

---

### 3. 코드

- 인라인 코드

```bash
`git status`
```

- 코드 블록

```bash
git status
```

### 4. 링크

```text
[Google](https://google.com)
```

---

### 5. 이미지

```text
![Git Logo](https://git-scm.com/images/logo@2x.png)
```

---

### 6. 강조

```text
*기울임*
**굵게**
~~취소선~~
```

---

### 7. 수평선

```text
---
***
___
```

---