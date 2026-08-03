# 웹 렌더링과 SEO

## CSR과 SSR

| 구분 | CSR(Client-Side Rendering) | SSR(Server-Side Rendering) |
| --- | --- | --- |
| HTML 생성 위치 | 브라우저 | 서버 |
| 첫 화면 | JavaScript 실행 후 표시 | 완성된 HTML을 먼저 표시 |
| 화면 전환 | 빠르고 자연스러움 | 요청에 따라 서버 작업이 필요할 수 있음 |
| SEO | 별도 대응이 필요할 수 있음 | 검색 엔진이 내용을 읽기 쉬움 |

- **CSR**: 서버에서 데이터와 JavaScript를 받아 브라우저가 화면을 만든다.
- **SSR**: 서버가 화면에 필요한 HTML을 만든 뒤 브라우저에 전달한다.

## SEO

SEO(Search Engine Optimization)는 웹 페이지가 검색 결과에 잘 노출되도록 최적화하는 작업이다.

핵심 요소는 의미 있는 HTML 구조, 정확한 제목과 설명, 빠른 로딩, 모바일 지원, 접근 가능한 링크와 콘텐츠이다. SSR이 SEO에 유리할 수 있지만, 렌더링 방식만으로 검색 순위가 결정되지는 않는다.
