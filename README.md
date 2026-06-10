[README.md](https://github.com/user-attachments/files/28781279/README.md)
<div align="center">

# 📚 BOOK FOREST

### 카카오 도서 검색 API를 연동한 온라인 서점 클론 프로젝트

[![GitHub](https://img.shields.io/badge/GitHub-imchoi2828--arch%2Folist__ai__project-181717?style=for-the-badge&logo=github)](https://github.com/imchoi2828-arch/olist_ai_project)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![KakaoAPI](https://img.shields.io/badge/Kakao_API-FFE812?style=for-the-badge&logo=kakao&logoColor=black)

**🔗 https://github.com/imchoi2828-arch/olist_ai_project**

</div>

---

## 📌 프로젝트 개요

교보문고 UI를 참고해 제작한 온라인 서점 스타일 클론 프로젝트입니다.
**카카오 REST API**를 직접 호출해 실제 도서 데이터를 가져오고, 메인 페이지와 상세 페이지 간 데이터 흐름을 `localStorage`로 연결했습니다.
순수 HTML / CSS / Vanilla JS만으로 동적 렌더링, 탭 전환, 검색, 슬라이더 등을 구현했습니다.

| 항목 | 내용 |
|------|------|
| 프로젝트명 | BOOK FOREST |
| 참고 서비스 | 교보문고 (kyobobook.co.kr) |
| 핵심 기술 | Kakao 도서 검색 API · localStorage · Vanilla JS |
| 구성 페이지 | 메인 (`index.html`) + 상세 (`detail.html`) |
| CSS 규모 | 약 1,760줄 (Flexbox + Grid 기반) |
| JS 규모 | main.js + detail.js (약 600줄) |

---

## 🏗️ 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                        사용자 브라우저                        │
│                                                             │
│  ┌──────────────────────┐       ┌──────────────────────┐   │
│  │    index.html        │       │    detail.html       │   │
│  │    (메인 페이지)      │       │    (상세 페이지)      │   │
│  │                      │       │                      │   │
│  │  ┌────────────────┐  │       │  ┌────────────────┐  │   │
│  │  │   main.js      │  │       │  │   detail.js    │  │   │
│  │  │ - 초기 렌더링   │  │       │  │ - LS 읽기      │  │   │
│  │  │ - 검색 처리     │  │       │  │ - 상세 렌더링  │  │   │
│  │  │ - 탭 전환       │  │       │  │ - 관련 도서    │  │   │
│  │  │ - 슬라이더      │  │       │  │ - URL 복구     │  │   │
│  │  └───────┬────────┘  │       │  └───────┬────────┘  │   │
│  └──────────┼───────────┘       └──────────┼───────────┘   │
│             │  카드 클릭 → book 데이터 저장  │               │
│             └──────────────────────────────►               │
│                                                             │
│  ┌──────────────────────────────────────┐                   │
│  │  localStorage  key: "selectedBook"   │                   │
│  │  { title, authors, thumbnail, ... }  │                   │
│  └──────────────────────────────────────┘                   │
└─────────────────────────┬───────────────────────────────────┘
                          │ fetch() + Authorization: KakaoAK
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                  Kakao REST API (외부 서버)                   │
│  GET /v3/search/book?query={keyword}&size={n}               │
│  Response: { documents: [...도서 배열], meta: {...} }        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 데이터 흐름

```
[사용자 검색 입력 or 페이지 진입]
          │
          ▼
  fetchKakaoBooks(query, size)
          │
          ▼
  Kakao API 호출 ──► 응답 수신
          │
          ▼
  sanitizeBooks()   ← 불완전 데이터 필터링
  (제목 없음 / 썸네일 없음 / 가격 0원 제거)
          │
          ▼
  renderBooks() + createBookCard()
          │
          ▼
  DOM에 카드 렌더링 + 클릭 이벤트 바인딩
          │
          ▼ (카드 클릭 시)
  saveSelectedBook()  → localStorage 저장
          │
          ▼
  detail.html?title={encoded} 로 이동
          │
          ▼
  getSelectedBook()  ← localStorage 읽기
          │
          └─ (데이터 없거나 title 불일치 시)
                    ▼
          fetchKakaoBooks(title, 1)  ← API 재호출로 자동 복구
                    │
                    ▼
  renderDetailHero() + renderDetailSections()
                    │
                    ▼
  loadRelatedBooks()  ← 저자명으로 관련 도서 API 재호출
```

---

## 📊 기능 구성 현황

```
메인 페이지 (index.html)
├── ✅ 히어로 슬라이더        3개 배너, 3초 자동 전환, 마우스오버 시 일시정지
├── ✅ 전체 메뉴 드롭다운      도서 / 디지털 / 라이프 카테고리
├── ✅ 호버 팝업 패널          로그인 / 회원가입 / 고객센터
├── ✅ 도서 검색               카카오 API 실시간 호출, 결과 위치로 스크롤
├── ✅ 베스트셀러              탭 4개 (종합 / 소설 / 에세이 / 인문)
├── ✅ 추천 도서               탭 4개 (문학 / 경제경영 / 자기계발 / 인문)
├── ✅ AI Picks                인문학 키워드 기반 큐레이션 섹션
├── ✅ 신간 도서               탭 3개 (전체 / 국내도서 / 외국도서)
├── ✅ 읽을거리 (Magazine)     고정 콘텐츠 카드 5개
└── ✅ 하단 프로모션 배너      3단 컬러 배너 섹션

상세 페이지 (detail.html)
├── ✅ 도서 표지 / 제목 / 저자  API 데이터 기반 렌더링
├── ✅ 가격 / 할인율 / 포인트   정가 대비 자동 계산 (5% 적립)
├── ✅ ISBN / 출간일 / 상태     API 응답 필드 활용
├── ✅ 책 소개 / 작가 정보      contents 필드 + 보조 텍스트
├── ✅ 목차                     제목 기반 더미 목차 5개
├── ✅ 독자 리뷰                더미 리뷰 카드 3개
├── ✅ 관련 도서 추천           저자명으로 API 재호출 (4권)
├── ✅ 상세 페이지 내 검색      검색 후 바로 상세 이동
└── ✅ URL 파라미터 복구        새로고침 시 API 재호출로 데이터 유지
```

---

## 🛠️ 기술 스택

| 구분 | 기술 | 사용 목적 |
|------|------|-----------|
| 마크업 | HTML5 | 시맨틱 구조, 접근성 (`aria-label`, `role`, `tabindex`) |
| 스타일 | CSS3 | Flexbox / Grid 레이아웃, CSS 변수, 반응형 설계 |
| 스크립트 | Vanilla JS (ES6+) | async/await, DOM 조작, 이벤트 처리 |
| API | Kakao REST API | 실시간 도서 데이터 검색 및 렌더링 |
| 저장소 | localStorage | 페이지 간 선택 도서 데이터 전달 |
| 버전 관리 | Git / GitHub | 커밋 이력 관리, 원격 저장소 배포 |

---

## 📁 파일 구조

```
kyobo-clone/
│
├── index.html              # 메인 페이지 (검색 + 다중 섹션)
├── detail.html             # 도서 상세 페이지
│
├── css/
│   └── style.css           # 전체 스타일시트 (약 1,760줄)
│                           # CSS 변수, 헤더, GNB, 카드, 슬라이더,
│                           # 상세 페이지 레이아웃 전부 포함
│
├── js/
│   ├── main.js             # 메인 페이지 스크립트
│   │                       #   fetchKakaoBooks()  — API 호출
│   │                       #   sanitizeBooks()    — 데이터 정제
│   │                       #   renderBooks()      — 카드 렌더링
│   │                       #   handleSearch()     — 검색 처리
│   │                       #   bindBookTabs()     — 탭 이벤트
│   │                       #   bindHeroSlider()   — 슬라이더
│   │                       #   bindHeaderMenus()  — 드롭다운
│   │
│   └── detail.js           # 상세 페이지 스크립트
│                           #   getSelectedBook()       — LS 읽기
│                           #   renderDetailHero()      — 메인 정보
│                           #   renderDetailSections()  — 하위 섹션
│                           #   loadRelatedBooks()      — 관련 도서
│                           #   initDetailPage()        — 초기화
│
└── img/                    # 이미지 리소스 폴더
```

---

## 🔌 Kakao 도서 검색 API 상세

### 엔드포인트 및 인증

```
GET https://dapi.kakao.com/v3/search/book
Authorization: KakaoAK {REST_API_KEY}
```

### 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `query` | String | ✅ | 검색어 |
| `target` | String | | 검색 필드 (`title` / `isbn` / `publisher` / `person`) |
| `size` | Integer | | 결과 수 (기본 10, 최대 50) |
| `page` | Integer | | 페이지 번호 (기본 1, 최대 50) |

### 응답 주요 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `title` | String | 도서 제목 |
| `authors` | Array | 저자 목록 |
| `publisher` | String | 출판사 |
| `thumbnail` | String | 표지 이미지 URL |
| `price` | Integer | 정가 |
| `sale_price` | Integer | 판매가 (`-1` = 판매 중지) |
| `isbn` | String | ISBN |
| `contents` | String | 책 소개 (최대 200자) |
| `datetime` | String | 출간일 (ISO 8601) |
| `url` | String | 카카오 상세 페이지 링크 |
| `status` | String | 판매 상태 (`정상판매` / `품절` 등) |

### 실제 호출 코드

```javascript
const KAKAO_REST_API_KEY = "YOUR_API_KEY";

async function fetchKakaoBooks(query, size = 12) {
  const url = `https://dapi.kakao.com/v3/search/book`
            + `?target=title`
            + `&query=${encodeURIComponent(query)}`
            + `&size=20`;

  const response = await fetch(url, {
    headers: { Authorization: `KakaoAK ${KAKAO_REST_API_KEY}` }
  });

  if (!response.ok) throw new Error("API 요청 실패");

  const data = await response.json();
  const cleanedBooks = sanitizeBooks(data.documents || []);
  return cleanedBooks.slice(0, size);
}
```

### 응답 예시

```json
{
  "documents": [
    {
      "title": "채식주의자",
      "authors": ["한강"],
      "publisher": "창비",
      "thumbnail": "https://search1.kakaocdn.net/...",
      "price": 12000,
      "sale_price": 10800,
      "isbn": "9788936434588",
      "contents": "어느 날 갑자기 채식을 선언한 여자의 이야기...",
      "datetime": "2007-10-30T00:00:00.000+09:00",
      "url": "https://search.daum.net/...",
      "status": "정상판매"
    }
  ],
  "meta": {
    "total_count": 284,
    "pageable_count": 45,
    "is_end": false
  }
}
```

---

## ⚙️ 핵심 구현 포인트

### 1. 방어 코딩 — 불완전한 API 데이터 처리

```javascript
// 제목·썸네일·가격 없는 항목 필터링
function sanitizeBooks(books) {
  return books.filter(book => {
    const hasTitle     = book.title && book.title.trim() !== "";
    const hasThumbnail = book.thumbnail && book.thumbnail.trim() !== "";
    const validPrice   = Number(book.sale_price || book.price) > 0;
    return hasTitle && hasThumbnail && validPrice;
  });
}

// 썸네일 없을 때 대체 이미지 자동 생성
function safeThumbnail(url, title) {
  if (url && url.trim() !== "") return url;
  return `https://placehold.co/300x420/ececec/666?text=${encodeURIComponent(title)}`;
}
```

### 2. 페이지 간 데이터 전달 — localStorage + URL 파라미터

```javascript
// 메인 → 상세 이동
function goToBookDetail(book) {
  localStorage.setItem("selectedBook", JSON.stringify(book));
  window.location.href = `./detail.html?title=${encodeURIComponent(book.title)}`;
}

// 상세 페이지 — 새로고침 / 직접 URL 진입 시에도 데이터 복구
async function initDetailPage() {
  let book = getSelectedBook();
  const title = new URLSearchParams(location.search).get("title");

  // localStorage 불일치 시 API 재호출로 자동 복구
  if ((!book || book.title !== title) && title) {
    const result = await fetchKakaoBooks(title, 1);
    if (result.length) book = result[0];
  }

  renderDetailHero(book);
  renderDetailSections(book);
  await loadRelatedBooks(book);
}
```

### 3. 탭 클릭 → API 재호출 → DOM 교체

```javascript
const categoryMap = {
  all: "베스트셀러",   novel: "소설",
  essay: "에세이",     humanities: "인문학",
  literature: "한국문학", business: "경제경영",
  selfhelp: "자기계발", domestic: "국내소설", foreign: "영미소설"
};

bestTabs.forEach(btn => {
  btn.addEventListener("click", async function () {
    activateTab(this);
    const books = await fetchKakaoBooks(categoryMap[this.dataset.category], 6);
    renderBooks("#bestSellerList", books, { showRank: true });
  });
});
```

### 4. 가격 자동 계산

```javascript
// 할인율 — 정가 vs 판매가 비교
const discountRate = (price > salePrice && salePrice > 0)
  ? Math.round(((price - salePrice) / price) * 100)
  : 0;

// 포인트 적립 — 판매가의 5% (최소 100P 보장)
const point = Math.max(100, Math.round((salePrice || price) * 0.05));
```

### 5. XSS 방지 — API 데이터 이스케이프

```javascript
function escapeAttr(value) {
  return String(value ?? "")
    .replace(/&/g, "&amp;").replace(/</g, "&lt;")
    .replace(/>/g, "&gt;").replace(/"/g, "&quot;")
    .replace(/'/g, "&#39;");
}
// 모든 API 응답 데이터는 DOM 삽입 전 escapeAttr() / escapeHtml() 처리
```

---

## 🚀 로컬 실행 방법

```bash
# 1. 저장소 클론
git clone https://github.com/imchoi2828-arch/olist_ai_project.git

# 2. 폴더 이동
cd kyobo-clone

# 3. VS Code Live Server로 실행
#    index.html 우클릭 → "Open with Live Server"
```

> ⚠️ **CORS 주의** — `file://` 경로로 직접 열면 카카오 API 호출 시 CORS 오류 발생합니다.
> VS Code **Live Server** 확장 프로그램 사용을 권장합니다.

---

## 📈 기술적 성장 포인트

| 주제 | 배운 내용 |
|------|-----------|
| 비동기 처리 | `async/await` 기반 API 호출 흐름과 try/catch 에러 핸들링 |
| 상태 전달 | `localStorage` + URL 파라미터 병용으로 새로고침에도 상태 유지 |
| 방어 코딩 | API 응답의 불완전 케이스를 사전 필터링하는 `sanitizeBooks()` 패턴 |
| 동적 렌더링 | 탭마다 API 재호출 후 DOM 교체하는 컴포넌트 패턴 |
| 보안 처리 | `escapeAttr()` / `escapeHtml()`로 XSS 방지 |
| CSS 설계 | CSS 변수(`--green`, `--radius-lg` 등)로 디자인 일관성 유지 |

---

## 🔗 참고

- [Kakao Developers — 도서 검색 API 공식 문서](https://developers.kakao.com/docs/latest/ko/daum-search/dev-guide#search-book)
- [교보문고](https://www.kyobobook.co.kr)

---

<div align="center">
  <sub>© 2026 BOOK FOREST Clone Project</sub>
</div>
