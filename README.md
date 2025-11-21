# SCSS 스타일 가이드

## 모듈 시스템 (@use)을 사용

### Dart Sass 3.0에서는 `@import` 대신 `@use`를 권장

```scss
// ❌ 기존 방식 (deprecated)
@import "./variables.scss";
@import "./mixin.scss";

// ✅ 새로운 방식
@use "sass:math";
@use "sass:map";
@use "sass:list";
@use "./variables" as vars;
@use "./mixin" as mix;
```

- 중복 import 방지
- 네임스페이스 충돌 방지
- 더 나은 성능

## 네임스페이스

### 권장사항

`as *` 사용을 피하고 명시적인 네임스페이스를 사용하는 것을 권장합니다.  
⭐ variable, mixin, sprite 는 네임스페이스 생략 가능  
⚠️ 다른 파일에서 호출했을때는 네임스페이스 반드시 사용

```scss
// ❌ 권장하지 않음 (네임스페이스 충돌 위험)
@use "./custom" as *;

// ✅ 권장 (명확한 네임스페이스)
@use "./custom" as custom;

.element {
  color: vars.$primary-color;
  @include custom.flex-center;
}
```

### 변수 스코프

`@use`를 사용할 때 변수 스코프가 더 엄격해집니다.

```scss
// custom.scss
$primary-color: #007bff;

// main.scss
@use "./custom" as custom;

.element {
  // ✅ 올바른 사용
  color: custom.$primary-color;

  // ❌ 직접 접근 불가
  // color: $primary-color;
}
```

## CSS 파일 구조

```text
src/assets/css/
├── variables.scss      # 색상, 폰트, 간격, 브레이크포인트 등 변수 정의(token 모음)
├── mixin.scss          # 반복 사용되는 스타일 믹스인 정의
├── reset.scss          # 브라우저 기본 스타일 리셋
├── webfonts.scss       # 웹폰트 정의 및 로드
├── ui.scss             # 레이아웃 및 공통 UI 스타일
├── utilities.scss      # 유틸리티 클래스
├── components.scss     # 재사용 가능한 컴포넌트 스타일(컴포넌트 모음)
├── base.scss           # 기본 스타일
├── base-entry.scss     # 기본 스타일 진입점
├── common.scss         # 위 공통파일 모음
├── common-entry.scss   # 공통 스타일 진입점
│
├── components/         # 컴포넌트별 개별 스타일
│   ├── accordion/
│   ├── badge_label/
│   ├── buttons/
│   ├── case/
│   ├── chips/
│   ├── controls/
│   ├── form/
│   ├── icons/
│   ├── layered/
│   ├── list/
│   ├── tab/
│   ├── table/
│   └── tooltip/
│
├── token/              # 디자인 토큰 (색상, 간격, 타이포그래피 등)
│   ├── _colors.scss
│   ├── _spacing.scss
│   ├── _typography.scss
│   ├── _radius.scss
│   ├── _transition.scss
│   ├── _easing.scss
│   └── _primitives.scss
│
└── pages/              # 페이지별 스타일
    ├── benefit/
    ├── card/
    ├── event/
    ├── finance/
    ├── helpdesk/
    ├── mypage/
    ├── service/
    ├── universal/
    ├── benefit.scss      #폴더 내 파일 분리 후 이 곳에서 병합(메뉴별 css 연결)
    ├── card.scss
    ├── event.scss
    ├── finance.scss
    ├── helpdesk.scss
    ├── main.scss
    ├── mypage.scss
    ├── service.scss
    ├── tops-club.scss
    └── universal.scss
```

## 공통 CSS 구성

```html
<link rel="stylesheet" href="/assets/css/base.scss">
<!-- CSS HUB -->
<link rel="stylesheet" href="/static/lib/swiper-bundle.min.css"><!-- 외 기타 라이브러리 -->
<link rel="stylesheet" href="/assets/css/common.scss">
<link rel="stylesheet" href="/assets/css/pages/card.scss">
<!-- // CSS HUB -->
```

##### base.scss
```scss
// 1. Foundation (기초) - 직접 사용 + 재export
@use "variables";
@forward "variables";
@use "mixin";
@forward "mixin";

// 2. Base (기본) - 직접 사용 + 재export
@use "reset";
@forward "reset";
@use "webfonts";
@forward "webfonts";

//3. Layout (공통 레이아웃)
@use "ui";
@forward "ui";
```
##### common.scss
```scss
// 4. Components (컴포넌트) - 직접 사용 + 재export
@use "components";
@forward "components";

//5. Utilities (유틸리티) - 직접 사용 + 재export
@use "utilities";
@forward "utilities";
```


## 네이밍 가이드

### 기본 규칙

- **!important 사용금지❌**
- 소문자로 사용
- nested 최소한으로 작업 **(page 별로 nested 금지❌)**
- 단어와 단어 사이는 `-` : dash를 사용 (`_` : underscore는 사용하지 않음)
- class 선택자는 소문자 kebab 사용(⚠️camelCase는 사용하지 말 것)
- id, name에 camelCase 사용
- 너무 길지 않은 단어는 풀스펠링 사용 (애매한 약자 사용 금지 chk, prdt 등)
- 너무 길면 가운데 숫자 넣어서 사용가능 (accessiblity -> a11y) : 최소 10자 이상
- 누구나 알 수 있는 축약어는 사용가능 (btn, txt, xl 등)

## CSS 작성 방법

### BEM + OOCSS

#### 1. BEM (Block Element Modifier)

- **블록(Block)**: 독립적인 컴포넌트의 최상위 단위
- **엘리먼트(Element)**: 블록 안에서 기능적으로 구분되는 내부 구성요소
- **모디파이어(Modifier)**: Variation을 나타내는 구분자
- ⚠️ 단어가 길어진다고 임의대로 생략하지 말것
- 🔥사실상 Modifier는 BEM에서 말하는대로 사용하지 않음!!

```scss
/* 블록 */
.component {
    /* 엘리먼트 */
    &__title { ... }
    &__content { ... }
}
/* 모디파이어 (카테고리 하위 타입별로 구분 할 때 - Block 외부에 작성) */
.component--variation {
    /* 엘리먼트 */
    &__title { ... }
    &__content { ... }
}
```

#### 2. OOCSS (Object Oriented CSS)를 통해 Variation을 표현

- **구조(Structure)**: 요소의 크기, 배치 등 레이아웃 관련 스타일
- **외형(Skin)**: 컬러, 폰트, 그림자 등 시각적 스타일

```scss
.component {
    /* 엘리먼트 */
    &__title { ... }
    &__content { ... }

    /* Variation */
    &.theme-primary { ... } /* 컬러  */
    &.size-xl, &.size-m { ... } /* 사이즈  */
    &.is-active { ... } /* 상태값  */
    &.ratio-35-65 { ... } /* 비율값  */
    &.direction-col, &.direction-row { ... } /* 방향  */
}
```

- 상속받는 내용에 따라 변경되는 구조 지양❌(재사용성 없음)

```scss
/* variation 을 부모 상속으로 만들면 안됨 */
.page--credit .card { ... }

```

### HTML 구조 예시

```html
<div class="shc-btn theme-primary size-xl">
  <strong class="shc-btn__title"></strong>
  <div class="shc-btn__content"></div>
</div>
```

## scss 파일에서 이미지를 호출 방법

### 이미지 파일 호출할 때 절대 경로 사용용

```scss
.item {
  background-image: url(/static/img/sample/sample.png);
}
.item--svg {
  background-image: url(/static/svg/sample.svg);
}
```

### scss 파일에서 4kb 이하의 svg 파일을 호출할 때 inline으로 사용

- html 에서는 svg sprite 사용 (상황에 따른 변경이 필요할때 css로 사용)

```scss
.notice {
  display: inline-block;
  width: 28px;
  height: 28px;
  background-image: svg-load("bell.svg");
  @include darkMode {
    background-image: svg-load("bell--dark.svg");
  }
}
```

⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️⬇️

```css
.notice {
  display: inline-block;
  width: 28px;
  height: 28px;
  background-image: url(
    data:image/svg + xml;charset=utf-8,
    %3Csvgwidth="29"height="29"viewBox="0 0 29 29"fill="none"xmlns="http://www.w3.org/2000/svg"%3E%3Cpathd="M14.3888 4.44855C10.5314 4.44875 7.40033 7.54201 7.32825 11.3841V16.9046L7.27942 17.056L6.41809 19.6995H14.3888V21.6995H6.16614C4.93333 21.6994 4.05445 20.4955 4.43762 19.3196L5.32825 16.5892V11.347C5.42056 6.42013 9.43627 2.44875 14.3888 2.44855V4.44855Z"fill="%23101828"/%3E%3Cpathd="M10.2678 21.8498C10.2678 21.5984 10.2971 21.3625 10.3342 21.1525L12.3039 21.5001C12.2794 21.639 12.2678 21.7521 12.2678 21.8498C12.2678 23.0534 13.2478 24.0333 14.4514 24.0333C15.6548 24.0331 16.634 23.0533 16.634 21.8498C16.634 21.7521 16.6224 21.6391 16.5979 21.5001L18.5676 21.1525C18.6047 21.3625 18.634 21.5984 18.634 21.8498C18.634 24.1578 16.7594 26.0332 14.4514 26.0333C12.1432 26.0333 10.2678 24.1579 10.2678 21.8498Z"fill="%23101828"/%3E%3Cpathd="M14.372 4.44863C18.2294 4.44883 21.3604 7.54209 21.4325 11.3842V16.9047L21.4813 17.0561L22.3427 19.6996H14.372V21.6996H22.5946C23.8274 21.6995 24.7063 20.4956 24.3231 19.3197L23.4325 16.5893V11.3471C23.3402 6.42021 19.3245 2.44883 14.372 2.44863V4.44863Z"fill="%23101828"/%3E%3C/svg%3E
  );
}
```

- 경로를 `/static/svg/` 생략
- 용량이 낮은 4kb 이하의 svg만 사용

## PMS 용 페이지 탬플릿

- `head` tag 가 열리는 곳에 탬플릿을 입력해야 PMS 에서 확인 가능
- 탬플릿 사용 방법은 아직 미정

```plaintext
<template>
    id: ALFIC01060010
    issueId: 110, 2177
    title: 사고보험금 접수안내
    menu: 라이프 > 보험금신청 > 사고보험금신청 > 접수내용 입력
    layout: VxSubLayout
    publish: 구유리, 김민서
    publishVersion: 2.15_20230808
    rework: 대상
</template>
```

## Breadcrumb

- head tag 마지막에 브레드크럼 작성 (url은 우선 #)

```js
 <script>
    window.pageBreadcrumb = [
      { text: "라이프", url: "#" },
      { text: "보험금신청", url: "#" },
      { text: "사고보험금신청", url: "#" },
      { text: "접수내용 입력", url: "#" }
    ];
  </script>
```

```html
<nav class="shc-breadcrumb" aria-label="Breadcrumb">
  <ol>
    <li><a href="#">홈</a></li>
    <li><a href="#">라이프</a></li>
    <li><a href="#">보험금신청</a></li>
    <li><a href="#">사고보험금신청</a></li>
    <li aria-current="page">접수내용 입력</li>
  </ol>
</nav>
```

- 개발모드에서는 ui.js 를 통해 입력(onload)
- 빌드모드에서는 vite.config.js를 통해 html로 작성 예정

## Layout (모든 내용은 .app 안에 작성:feedback, skip-nav 제외(스크립트))

```html
<div class="app">
  <!-- 공통 HEADER inject -->
  <inject file="/components/header.html" />

  <!-- Main Content -->
  <main id="shcMainContent" class="shc-main">
    <section class="shc-section">
      <h1 class="headline--l">Layout</h1>
      <div></div>
    </section>
    <section class="shc-section">
      <h2 class="title--m">Layout</h2>
      <div></div>
    </section>
    <section class="shc-section">
      <h3 class="title--s">Layout</h3>
      <div></div>
    </section>

    <!-- 하단 CTA 영역 (버튼이 없어도 존재함)-->
    <section class="shc-cta-area">
      <div class="shc-btn-group">
        <button class="shc-btn theme-primary size-xl" type="button">
          <span class="shc-btn__text">텍스트</span>
        </button>
      </div>
    </section>
  </main>
  <!-- // Main Content -->

  <!-- 공통 FOOTER inject -->
  <inject file="/components/footer.html" />

  <!-- 팝업(풀팝업, 바텀시트, 모달 등)-->
  <div class="shc-layered" data-layered-name="testLayered"></div>
</div>
```

- heading tag는 누락 없이 작성 (웹접근성)
- hidden 은 display:none 임으로 접근성이 필요한 부분은 .sr-only 클래스 활용  
(button, a 태그에 사용안함 - aria-label 사용)
- 필요한 부분에만 wai-aria 작성
- 팝업은 페이지 하단에 작성(app 안에!!!)

#### 변경사항

- :has 셀렉터 사용 금지  
  (기존에 사용한 컴포넌트 작업은 수정 예정)
- `<img data-src="" alt="">`

#### 작업 방법

- 업무망 PC를 통해 업무 분장 및 일정 확인
- ms teams 를 통해 완료된 기획/디자인 확인
- 반복적으로 사용할 컴포넌트가 필요할 경우 요청 (slack 채널)  
  (혹은 직접 작업하여 컴포넌트 등록 요청)
- 나머지 페이지부터 작업 진행 합니다.
- 컴포넌트가 완성되면 적용 후 수정사항 확인
- 디자인 컨펌 요청

##### 작업 시 주의 사항

- 디자인과 기획서는 MS Teams 문서에 연결된 figma 페이지에서 확인 가능합니다.
- 아직은 (09/15 기준) IA문서 등이 정리 되지 않아 Page id를 확인하기 어렵습니다.
- 대략적인 이름을 정해서 작업하고 추후에 변경해야 합니다.
- 작업은 각자의 branch 에서 진행하고 작업이 끝나면 올립니다.
- git user.name 은 각자의 이름으로 해주세요 (사번X)
- shc- 로 시작하는 이름은 재사용 가능한 컴포넌트, 모듈명 입니다.
- :has 셀렉터 사용 금지(09/15 결정 - 기존 제작 컴포넌트 수정 예정)
- gap 은 display:flex 안에서만 사용합니다.
- css 변수 사용 시 두번째 인자를 지워주세요.
- background-image 를 호출 할 떄 `/static/sprite` 를 사용하지 마세요
- 큰 배너등의 이미지를 제외하고는 svg를 이용해 주세요
- svg를 이용할 때 `@mixin svg-load()`을 활용해 주세요.(inline 방식으로 치환됨)
- 웹/앱접근성 인증마크 획득 예정이고, 접근성 컨설트 받는 중 입니다.

#### 작업 전 확인 사항

- 디자인 컨펌 요청 방법 (디자인파트에서 확인 가능한 주소, 테스트서버)
- IA 문서 정리 (페이지 ID, 구조)
- 일정 및 순서 확인

### 다크모드 이미지 경로

```html
<!-- as-is -->
<img src="/static/images/basic/**/*.png" alt="" />
<img src="/static/images/dark/**/*.png" alt="" />

<!-- tobe -->
<img src="**.png" alt="" data-dark />
```

### 다크모드 확인

- OS 에 다크모드를 전환해서 확인하는 방법 "윈도우 > 설정 > 개인설정 > 모드선택"
- 콘솔 창에 gfn_theme.dark() 를 치면 다크모드로 전환 (새로고침하면 원래대로 돌아옴)

#### Page CSS 구분 사용 방법

- /pages/아래 메뉴별 폴더를 만들고 메뉴하위 뎁스 단위로 scss 파일을 쪼개서 사용

menu.scss

```scss
@use "./menu/menu-submenu01";
@use "./menu/menu-submenu02";
@use "./menu/menu-submenu03";
@use "./menu/menu-submenu04";
```

##### CSS 셀렉터 규칙 추가

- 재사용 가능성보다 충돌을 줄이는 것이 중요
- 각 페이지 부모(main or section)에 .(prefix : 1depth 서비스명줄임)\_2depth 서비스명 하위에 작성
- nested depth 2-3단계 이상 작성 가능 (지향하지는 않음)
- 재사용할수 있는 부분은 별도로 작성
- 전체페이지에서 재사용 가능한 부분은 case.scss ~ case05.scss 에 작성
