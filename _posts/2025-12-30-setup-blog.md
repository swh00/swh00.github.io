---
layout: post
title: "[Github] 블로그 구축 기록 (Theme: Chirpy)"
date: 2025-12-30 18:00:00 +0900
categories: [개발, 블로그]
tags: [github-pages, jekyll, chirpy, giscus]
---

GitHub Pages와 Jekyll을 이용하여 정적 블로그를 구축하였습니다. 테마 선정부터 Chirpy 테마 설정, 그리고 Giscus 댓글 연동까지의 과정을 글로 작성해보겠습니다.

## 1. 테마 선정 및 리소스

Jekyll 블로그의 가장 큰 장점은 다양한 테마를 적용할 수 있다는 점입니다. 테마를 선택할 때 참고할 수 있는 주요 사이트는 다음과 같습니다.

* **[Jekyll Themes](https://jekyllthemes.io/):** 가장 대중적인 테마 저장소.
* **[Jamstack Themes](https://jamstackthemes.dev/ssg/jekyll/):** 디자인 퀄리티가 높은 테마들이 많음.
* **[RubyGems](https://rubygems.org/search?query=jekyll-theme):** 배포가 용이한 Gem 기반 테마 검색 가능.

### 선정 기준
개인적으로 블로그를 운영함에 있어 다음 기능들을 필수 요소로 고려했습니다.
1.  **사이드바 구조:** 카테고리와 태그 접근이 용이해야 함.
2.  **검색 기능:** 본문 및 태그 검색이 가능해야 함.
3.  **가독성:** 코드 하이라이팅과 본문 폰트 가독성이 우수해야 함.
4.  **유지보수:** 업데이트가 활발하고 문서화가 잘 되어 있어야 함.

초기에는 `Simplex` 등의 테마를 고려했으나, 빌드 의존성 문제와 커스텀 플러그인 호환성 이슈 등 초보인 저에게는 어려움이 있었습니다. 그래서 최종적으로 개발자 블로그로서 기능이 충실하고 설정이 간편한 **[Chirpy](https://github.com/cotes2020/chirpy-starter)** 테마를 선택했습니다.

---

## 2. Chirpy Starter를 이용한 설치

Chirpy 테마를 밑바닥부터 설치하는 것은 Ruby 환경 설정 등 복잡한 과정이 수반되지만, 제작자가 제공하는 `Chirpy Starter`를 사용하면 초기 설정을 건너뛰고 바로 시작할 수 있습니다.

### 설치 과정
1.  **[Chirpy Starter 저장소](https://github.com/cotes2020/chirpy-starter)**에 접속한다.
2.  우측 상단의 `Use this template` > `Create a new repository`를 클릭한다.
3.  Repository 이름을 `username.github.io` 형식으로 설정하여 생성한다.

이 방식을 사용하면 GitHub Actions가 내장되어 있어, 글을 작성하고 Push만 하면 자동으로 빌드 및 배포가 이루어져 간단한 설정만으로 자신의 블로그 형태를 갖출 수 있습니다.

---

## 3. 환경 설정 (_config.yml)

블로그의 기본 정보는 루트 디렉토리의 `_config.yml` 파일에서 수정이 가능하고, 주요 설정 항목은 다음과 같습니다.

### 기본 정보 및 언어
한국어 사용자에게 맞는 시간대와 언어 설정을 적용할 수 있습니다.

```yaml
lang: ko
timezone: Asia/Seoul
title: "swh Log" # 블로그 제목
tagline: "주식과 코딩을 기록하는 공간" # 특수문자가 포함될 경우 반드시 따옴표("") 사용
url: "https://swh00.github.io"
github:
  username: swh00
```

### 주의사항: YAML 문법
`tagline`이나 `description` 작성 시 콜론(:) 등의 특수문자가 포함되면 빌드 에러(`mapping values are not allowed`)가 발생할 수 있으므로, 문자열은 항상 쌍따옴표(`""`)로 감싸는 것이 안전합니다.

### 카테고리 및 태그 사용
Chirpy 테마는 별도의 설정 없이 포스트의 머리말에 선언하는 것만으로 카테고리 페이지가 자동 생성되어 관리가 매우 편리합니다.

```yaml
---
title: 글 제목
categories: [대분류, 중분류]
tags: [태그1, 태그2]
---
```
그렇게 완성된 블로그는 다음과 같이 보입니다.

![화이트 모드](/assets/img/2025post/2025-12-30-setup-blog/2025-12-30-chirpy-white.jpg)
_화이트 모드_

![다크 모드](/assets/img/2025post/2025-12-30-setup-blog/2025-12-30-chirpy-dark.jpg)
_다크 모드_

## 4. 댓글 시스템 적용 (Giscus)

블로그에서 댓글 기능을 구현하기 위해 **Giscus**를 도입했습니다. Giscus는 GitHub Discussions를 기반으로 작동하며, 광고가 없고 GitHub 계정과 연동된다는 장점이 있습니다.

### 적용 순서
1. **저장소 설정:** 블로그 리포지토리의 Settings에서 `Discussions` 기능을 활성화한다.
2. **Giscus 앱 설치:** [GitHub Apps](https://github.com/apps/giscus)에서 저장소 접근 권한을 허용한다.
3. **설정값 생성:** [giscus.app](https://giscus.app/ko)에 접속하여 저장소 정보를 입력하고 `data-repo-id`와 `data-category-id`를 발급받는다.
4. **config 적용:** `_config.yml`의 comments 섹션에 발급받은 값을 입력한다.

```yaml
comments:
  provider: giscus
  giscus:
    repo: swh00/swh00.github.io
    repo_id: "발급받은 repo ID" #
    category: General
    category_id: "발급받은 category ID"
    mapping: pathname
    input_position: bottom
    lang: ko
```

## 5. 글 작성 가이드

Chirpy 테마에서 글을 작성하는 표준 절차는 다음과 같습니다.

### 파일 생성 규칙
모든 포스트는 `_posts` 디렉토리 안에 생성해야 하며, 파일명은 반드시 **`YYYY-MM-DD-제목.md`** 형식을 따라야 합니다. 형식을 지키지 않으면 사이트에 노출되지 않습니다.

* **올바른 예:** `2025-01-01-hello-world.md`
* **잘못된 예:** `hello-world.md` (날짜 누락), `2025-1-1-title.md` (자릿수 오류)

### Front Matter (머리말) 작성
파일 최상단에는 포스트의 메타 데이터를 정의하는 Front Matter가 위치해야 합니다.

```yaml
---
layout: post
title: "포스트 제목"
date: 2025-01-01 12:00:00 +0900
categories: [대분류, 중분류]
tags: [태그1, 태그2]
---
```

---

## 6. 이미지 첨부 방법

GitHub Pages는 정적 호스팅이므로 이미지를 저장소에 직접 업로드하여 경로를 연결해야 합니다.

### 1. 이미지 업로드
블로그 루트 디렉토리의 `assets/img/` 폴더에 이미지를 업로드합니다. (관리를 위해 연도별 폴더 등을 만들어 관리하는 것을 권장합니다.)

* 경로 예시: `assets/img/2025/chart_01.png`

### 2. 마크다운에 이미지 삽입
글 작성 시 절대 경로(`/`)를 사용하여 이미지를 불러옵니다.

```markdown
![이미지 설명](/assets/img/2025/chart_01.png)
```

**Tip:** 캡션을 달거나 사이즈를 조절하고 싶다면 HTML 태그를 직접 사용하거나 테마에서 제공하는 확장 문법을 사용할 수 있습니다.

---

## 7. 마무리

위 과정을 통해 블로그의 기본적인 골격을 완성했습니다. 생각보다 블로그 만드는 것은 쉬웠지만, 글을 작성하는 것은 이것저것 번거로운 느낌이 있었습니다.

앞으로 이것저것 올릴 수 있는 개인 블로그가 생겼으니, 어떤 내용을 채워 나갈지 차차 고민해보겠습니다.
