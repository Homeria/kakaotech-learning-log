## 1️⃣ Tailwind CSS란?

https://tailwindcss.com/

Tailwind CSS는 웹사이트나 앱의 디자인을 빠르고 쉽게 꾸밀 수 있도록 해주는 **CSS 프레임워크**입니다. 별도의 CSS 파일을 작성하는 대신, `className`에 미리 정의된 짧은 클래스를 조합해 스타일을 적용합니다.

```tsx
// ❌ 기존 CSS 방식 — 클래스명을 직접 만들고 CSS 파일에 따로 정의
<button className="submit-btn">제출</button>

// .submit-btn {
//   background-color: #2563eb;
//   color: white;
//   padding: 8px 16px;
//   border-radius: 8px;
// }

// ✅ Tailwind 방식 — CSS 파일 없이 className 안에서 바로 해결
<button className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">
  제출
</button>
```

---

## 2️⃣ 장점과 단점

### ✅ 장점

| 장점 | 설명 |
| --- | --- |
| 클래스명 고민 불필요 | `.submit-btn`, `.card-wrapper` 같은 이름을 직접 짓지 않아도 됩니다 |
| CSS 파일 관리 불필요 | 모든 스타일이 JSX 안에 있어 파일 전환 없이 개발 가능합니다 |
| 번들 최적화 자동 | 빌드 시 실제 사용된 클래스만 포함 → CSS 파일 크기 최소화 |
| 디자인 시스템 내장 | 색상·간격·폰트 크기가 일관된 스케일로 미리 정의되어 있습니다 |
| 반응형·상태 처리 간편 | `md:`, `hover:` 접두사로 별도 미디어 쿼리 없이 처리 가능합니다 |

### ❌ 단점

| 단점 | 설명 |
| --- | --- |
| className이 길어짐 | 스타일이 많을수록 JSX가 길어 보일 수 있습니다 |
| 초기 러닝커브 | `padding: 16px`가 `p-4`라는 것을 처음엔 매핑해서 외워야 합니다 |
| 복잡한 커스텀 스타일 | 디자인 시스템 외의 세밀한 커스텀 스타일은 `@theme` 설정이나 임의값(`[...]`) 문법을 써야 해서 복잡해질 수 있습니다 |

#### 🎨 Tailwind CSS 적용

> 📌 **Tailwind CSS v4는 `tailwind.config.js` 설정 파일이 필요 없습니다**
> 

### 설치

```bash
npm install -D tailwindcss @tailwindcss/postcss postcss
```

### PostCSS 설정 파일 작성 → 최상위 글로벌 CSS에 불러오고 최상위 레이아웃에 CSS 로드

```jsx
// postcss.config.mjs
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
export default config;
```

```css
/* app/globals.css */
@import "tailwindcss";
```

```tsx
import "./globals.css"; // 글로벌 스타일 임포트

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  );
}
```

---

## 3️⃣ 기본 문법

### 기본 적용

HTML/JSX의 `className`에 클래스를 나열하면 됩니다. 띄어쓰기로 구분합니다.

```tsx
<div className="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
  <h2 className="text-xl font-bold text-gray-900">제목</h2>
  <p className="text-sm text-gray-500 mt-2">설명 텍스트</p>
</div>
```

---

### 반응형 (Responsive)

`sm:` `md:` `lg:` `xl:` 접두사로 화면 크기에 따라 다른 스타일을 적용합니다.

```tsx
{/* 기본(모바일): 세로 배치, md(768px) 이상: 가로 배치 */}
<div className="flex flex-col md:flex-row gap-4">
  <div>왼쪽</div>
  <div>오른쪽</div>
</div>

{/* 기본: 텍스트 작게, lg(1024px) 이상: 크게 */}
<h1 className="text-xl lg:text-3xl font-bold">타이틀</h1>
```

| 접두사 | 최소 너비 | 대상 기기 |
| --- | --- | --- |
| (없음) | 0px ~ | 모든 크기 (기본값) |
| `sm:` | 640px ~ | 작은 태블릿 |
| `md:` | 768px ~ | 태블릿 |
| `lg:` | 1024px ~ | 데스크톱 |

---

### 상태 변화 (hover, focus)

`hover:` `focus:` 접두사로 상태에 따른 스타일을 적용합니다.

```tsx
{/* hover 시 배경색 변경 + 부드러운 전환 효과 */}
<button className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
  버튼
</button>

{/* focus 시 테두리·링 색상 변경 */}
<input className="border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-100 outline-none px-3 py-2 rounded-lg" />
```

---

### 임의값 (Arbitrary Values)

Tailwind 기본 스케일에 없는 정확한 값이 필요할 때 `[값]` 형태로 사용합니다.

```tsx
<div className="w-[380px] text-[13px] bg-[#f0f4ff] mt-[30px]">
  정확한 수치가 필요할 때
</div>
```

<aside>
💡

임의값은 디자이너 시안의 정확한 수치를 반영할 때 유용하지만, 남용하면 Tailwind의 일관성 장점이 사라집니다. 가능하면 기본 스케일(`p-4`, `w-96` 등)을 우선 사용하세요.

</aside>

---

### 레이아웃의 핵심: Flexbox

https://developer.mozilla.org/ko/docs/Web/CSS/Reference/Properties/flex

https://tailwindcss.com/docs/flex

https://flexboxfroggy.com/#ko

!image.png

<aside>
🚧

**flexbox:** 웹 페이지의 레이아웃을 유연하고 효율적으로 배치하기 위해 만들어진 레이아웃 방식

- 참고
    - **컨테이너 (Container):** 정렬할 요소들을 감싸는 부모 태그입니다.
    - **아이템 (Item):** 컨테이너 안에 나열되는 자식 태그들입니다.
    - **주축 (Main Axis)과 교차축 (Cross Axis):** 요소들이 배치되는 방향인 주축과, 그에 수직인 교차축을 기준으로 정렬과 여백을 제어합니다.

#### 1. 방향 (Direction)

- flex-row: 가로 배치 (기본값)
- flex-col: 세로 배치

#### 2. 정렬 (Alignment)

- **주축 정렬 (justify)**
    - justify-start: 왼쪽(위)으로 붙기
    - justify-center: **가운데 정렬**
    - justify-between: 양 끝으로 쫙 벌리기 (메뉴 바 만들 때 필수!)
    - justify-end: 오른쪽(아래)으로 붙기
- **교차축 정렬 (items)**
    - items-start: 위쪽 정렬
    - items-center: **세로 기준 정렬**
    - items-end: 아래쪽 정렬

#### 3. 간격 (Gap)

- gap: 요소 사이의 간격을 일정하게 띄움
    - gap-4 → 16px 만큼 간격
</aside>

---

## **✏️ [실습] Tailwind CSS로 스타일링해보기**

- Tailwind CSS의 긴 클래스 이름을 관리하고 자동 완성 기능을 지원하는 **VS Code 확장 프로그램**
    - **Tailwind CSS IntelliSense**: 공식 자동 완성 및 힌트 제공
    - **prettier-plugin-tailwindcss**: 복잡한 클래스명을 규칙에 맞춰 자동 정렬

!image.png

!image.png

```jsx
/**
 * Tailwind CSS 실습
 *
 * 각 섹션 주석을 읽고, 표시된 요소의 className에 Tailwind 클래스를 추가하세요.
 */

export default function App() {
  return (
    <main className="flex w-full flex-1 justify-center px-6 py-6">
      <div className="flex w-full max-w-lg flex-col gap-6 text-left">
        <header className="space-y-3 text-center">
          <h1 className="text-2xl font-bold text-slate-900">
            Tailwind CSS 실습
          </h1>
          <p className="text-sm leading-relaxed text-slate-500">
            주석의 문제를 해결하며 className을 채워 보세요.
          </p>
        </header>

        {/* ── 1. 텍스트 크기 · 굵기 ───────────────────────────── */}
        <section className="flex flex-col gap-4">
          <h2 className="text-center text-base font-semibold text-slate-800">
            1. 텍스트 크기 · 굵기
          </h2>

          <div className="flex flex-col gap-4 rounded-xl border border-dashed border-amber-300 bg-amber-50/60 p-8">
            <div className="rounded-lg bg-white px-6 py-5 ring-1 ring-slate-100">
              {/* 문제
              아래 제목 「Tailwind 연습」의 글자 크기를 3xl, 굵기를 bold로 맞춰 주세요. */}
              <h3 className="">Tailwind 연습</h3>
            </div>
            <p className="text-sm leading-relaxed text-slate-500">
              위 제목만 스타일을 적용하면 됩니다.
            </p>
          </div>
        </section>

        {/* ── 2. 반응형 ───────────────────────────────────────── */}
        <section className="flex flex-col gap-4">
          <h2 className="text-center text-base font-semibold text-slate-800">
            2. 반응형
          </h2>

          <div className="flex flex-col gap-4 rounded-xl border border-dashed border-sky-300 bg-sky-50/60 p-8">
            <div className="rounded-lg bg-white px-6 py-5 ring-1 ring-slate-100">
              {/* 문제
              제목은 모바일에서 text-xl, md(768px~) 이상에서 text-4xl 이 되게 해 주세요. */}
              <h3 className="">화면 크기에 따라 달라지는 제목</h3>
            </div>
            <p className="text-sm leading-relaxed text-slate-500">
              브라우저 창 너비를 줄였다 늘려 보세요.
            </p>
          </div>
        </section>

        {/* ── 3. 상태 변화 (Hover) ────────────────────────────── */}
        <section className="flex flex-col gap-4">
          <h2 className="text-center text-base font-semibold text-slate-800">
            3. 상태 변화 (Hover)
          </h2>

          <div className="flex justify-center rounded-xl border border-dashed border-violet-300 bg-violet-50/60 p-8">
            {/* 문제
            버튼: 배경 indigo-600, 흰 글씨, 호버 시 배경 indigo-800 */}
            <button
              type="button"
              className="inline-flex items-center rounded-lg px-6 py-3 text-sm font-medium"
            >
              호버해보세요
            </button>
          </div>
        </section>

        {/* ── 4. Flex · 마진 · 패딩 ───────────────────────────── */}
        <section className="flex flex-col gap-4">
          <h2 className="text-center text-base font-semibold text-slate-800">
            4. Flex · 마진 · 패딩
          </h2>

          <div className="flex flex-col gap-4 rounded-xl border border-dashed border-emerald-300 bg-emerald-50/60 p-8">
            {/* 문제
            프로필 바: flex 가로 배치, 양끝 정렬(justify-between), 세로 가운데(items-center), 안쪽 패딩 p-4, 배경 white, 모서리 rounded-lg */}
            <div className="">
              <span className="font-medium text-slate-800">프로필</span>
              <button
                type="button"
                className="rounded-md px-4 py-2 text-sm text-indigo-600 underline"
              >
                설정
              </button>
            </div>
            <p className="text-sm leading-relaxed text-slate-500">
              점선 박스 안에서 프로필과 설정 버튼이 양쪽 끝에 배치되면
              성공입니다.
            </p>
          </div>
        </section>

        <footer className="border-t border-slate-200 pt-8 text-center text-xs leading-relaxed text-slate-400">
          네 가지 유형을 모두 적용했는지 확인해 보세요.
        </footer>
      </div>
    </main>
  );
}
```