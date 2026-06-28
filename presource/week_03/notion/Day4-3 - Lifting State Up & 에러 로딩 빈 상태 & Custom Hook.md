## 1. Lifting State Up — 상태 끌어올리기

### 학습 이유

컴포넌트를 분리하다 보면 이런 문제를 마주칠 수 있습니다.

> 
> 
> 
> **상황 가정: 쇼핑몰 장바구니 연동하기**
> 
> 쇼핑몰 화면을 상단의 <Header>와 하단의 <ProductList> 컴포넌트로 나누었다고 해봅시다.
> 
> - <Header>: 우측 상단에 **총 장바구니 개수**를 보여줍니다.
> - <ProductList>: 상품 목록과 **'장바구니 담기' 버튼**이 있습니다.
> 
> 사용자가 하단의 '담기' 버튼을 누르면, 상단의 장바구니 개수가 올라가야 합니다. 즉, 두 컴포넌트가 **'장바구니 개수'라는 같은 데이터(State)를 공유**해야 합니다.
> 
> !https://workspace-help.nhn-commerce.com/aurora-guide/api-1/product-list
> 
> https://workspace-help.nhn-commerce.com/aurora-guide/api-1/product-list
> 

React는 데이터가 **부모 → 자식** 방향으로만 흐릅니다. 형제 컴포넌트끼리는 직접 데이터를 주고받을 수 없습니다. 이 문제를 해결하는 패턴이 **상태 끌어올리기(Lifting State Up)**입니다.

### 핵심 개념

두 컴포넌트가 같은 데이터를 공유해야 한다면 → **공통 부모가 상태를 소유**하고, 각 자식에게 필요한 것만 내려줍니다.

- `ShoppingHeader` → 현재 개수(숫자)만 필요
- `ProductList` → 개수를 올릴 함수만 필요

```jsx
// 시연 1: Lifting State Up
import { useState } from "react";
import ShoppingHeader from "../shared/components/ShoppingHeader";
import ProductList from "../shared/components/ProductList";
import { PRODUCTS } from "../shared/data/products";
import "@/App.css";

export default function ShoppingApp() {
  // 두 자식이 공유해야 할 상태를 공통 부모가 소유
  const [cartCount, setCartCount] = useState(0);
  const handleAddToCart = () => setCartCount((c) => c + 1);

  return (
    <div className="demo-app">
      {/* Header 에게는 '현재 개수'를 내려줌 */}
      <ShoppingHeader count={cartCount} />
      {/* ProductList 에게는 개수를 올릴 '함수'를 내려줌 */}
      <ProductList products={PRODUCTS} onAdd={handleAddToCart} />
    </div>
  );
}
```

```jsx
// ShoppingHeader — 표시만 담당
function ShoppingHeader({ count }) {
  return (
    <header className="shop-header">
      <h1>🛍 쇼핑몰</h1>
      <span className="cart-badge">🛒 {count}개</span>
    </header>
  );
}

// ProductList — 담기 버튼 담당
function ProductList({ products, onAdd }) {
  return (
    <ul className="product-list">
      {products.map(product => (
        <li key={product.id} className="product-item">
          <span>{product.name} — {product.price.toLocaleString()}원</span>
          <button className="add-btn" onClick={onAdd}>담기</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## 2. 로딩·에러·빈 상태 처리

<aside>
✏️

### 학습 이유

실제 서비스에서 상품 목록은 서버에서 불러옵니다. 이때 반드시 발생하는 3가지 상황에 대비해야 합니다.

> "버튼을 눌렀는데 왜 아무것도 안 나오지?" → 로딩 중임을 알려야 한다
> 

> "네트워크가 끊겼을 때 빈 화면이 떠 있으면?" → 에러 메시지와 재시도 버튼이 필요하다
> 

> "등록된 상품이 없으면?" → 빈 상태 안내가 필요하다
> 

이 세 가지를 처리하지 않으면 사용자는 앱이 고장났다고 생각합니다.

</aside>

### 데이터 로딩 패턴: try / catch / finally

```jsx
// demo/02_loading/App.jsx
import { useState, useEffect } from "react";

export default function ShoppingApp() {
  const [cartCount, setCartCount]   = useState(0);
  const [products, setProducts]     = useState([]);
  const [isLoading, setIsLoading]   = useState(true);  // 초기값 true — 처음 진입 시 로딩 상태로 시작
  const [error, setError]           = useState(null);
  const [fetchIndex, setFetchIndex] = useState(0);     // 데이터 재요청(retry)을 위한 트리거 state

  useEffect(() => {
    // useEffect 자체는 async 함수를 가질 수 없으므로(Promise 반환 불가), 내부에 별도 async 함수를 선언합니다.
    const load = async () => {
      try {
        await new Promise(r => setTimeout(r, 1000)); // 서버 응답 시뮬레이션 (1초 지연)
        if (Math.random() < 0.3) throw new Error("서버 응답 오류 (시뮬레이션)");
        setProducts(PRODUCTS);
      } catch (err) {
        setError(err.message || "상품을 불러오지 못했습니다. 다시 시도해 주세요.");
      } finally {
        setIsLoading(false); // 성공·실패 여부와 상관없이 로딩 완료 처리
      }
    };

    load();
  }, [fetchIndex]); // 의존성 배열의 fetchIndex가 변경되면 useEffect가 다시 실행됨 (재시도 기능)

  const handleAddToCart = () => setCartCount(c => c + 1);

  const handleRetry = () => {
    setIsLoading(true);
    setError(null);
    setFetchIndex(i => i + 1); // 상태를 변경하여 의존성 배열에 걸린 useEffect를 깨웁니다.
  };

  // Early Return 패턴 — JSX 내부의 복잡한 삼항 연산자 대신, 조건을 순서대로 읽을 수 있어 가독성이 높습니다.
  const renderContent = () => {
    if (isLoading) return <LoadingSpinner />;
    if (error) return <ErrorMessage message={error} onRetry={handleRetry} />;
    if (products.length === 0) return <EmptyState />;
    return <ProductList products={products} onAdd={handleAddToCart} />;
  };

  return (
    <div className="demo-app">
      <ShoppingHeader count={cartCount} /> {/* 헤더 레이아웃은 로딩·에러 상태와 무관하게 항상 유지 (화면 깜빡임 방지) */}
      <div className="demo-content">
        {renderContent()}
      </div>
    </div>
  );
}
```

- 참고
    
    `isLoading / error / data` 세트를 매번 직접 선언하고 `try/catch/finally`로 관리하면 같은 패턴의 코드가 컴포넌트마다 반복됩니다. 캐싱, 재시도, 백그라운드 갱신 같은 처리를 추가할수록 더 복잡해집니다.
    
    이런 문제를 해결하기 위해 **TanStack Query** (`@tanstack/react-query`) 같은 라이브러리를 쓰거나, React의 `<Suspense>` / `<ErrorBoundary>`를 활용해 로딩·에러 상태를 컴포넌트 트리 단위로 선언적으로 처리하기도 합니다.
    

---

## 3. Custom Hook — UI와 로직 분리하기

<aside>
✏️

### 학습 이유

시연 2의 App.jsx를 보면 상태 선언, 데이터 로딩 로직, UI 렌더링이 한 파일에 섞여 있습니다. 상품 목록을 불러오는 기능이 다른 페이지에도 필요하다면 이 코드를 복붙해야 할까요?

**Custom Hook**은 `useState`, `useEffect` 등의 로직을 재사용 가능한 함수로 추출하는 방법입니다. 이름은 `use`로 시작합니다.

</aside>

https://ko.react.dev/learn/reusing-logic-with-custom-hooks

<aside>
❓

### 1. 언제 쓰는가?

- **상태를 다루는 로직이 중복될 때:** 여러 컴포넌트에서 API 통신(로딩·에러 처리), 입력창 값 변경(`onChange`), 모달 창 열고 닫기 등의 동일한 상태 관리 코드가 반복될 때 사용합니다.
- **컴포넌트 코드가 비대해질 때:** 화면을 그리는 코드(JSX)보다 데이터를 처리하는 로직 코드가 더 길어져 가독성이 떨어질 때 분리합니다.

### 2. 왜 쓰는가?

- **React 고유 기능을 포함한 재사용 가능:** 일반 자바스크립트 함수 내부에서는 `useState`나 `useEffect`를 호출할 수 없습니다. 규칙에 맞게 커스텀 훅으로 만들어야만 React의 기능을 내포한 채로 로직을 재사용할 수 있습니다.
- **관심사 분리 (UI와 로직의 분리):** 컴포넌트는 '화면을 그리는 역할'에만 집중하고, 커스텀 훅은 '데이터를 처리하는 역할'만 전담하여 코드의 가독성과 유지보수성을 높입니다.
- **상태의 독립성 유지:** 하나의 커스텀 훅을 여러 컴포넌트에서 공유하더라도, 각 컴포넌트가 가지는 상태(State)는 서로 영향을 주지 않고 독립적으로 작동합니다.
</aside>

### useProducts — 상품 로딩 로직 분리

```jsx
// demo/03_custom_hook/hooks/useProducts.js
import { useState, useEffect } from "react";
import { PRODUCTS } from "../../shared/data/products";

// 상품 목록 로딩 담당: 상품 데이터, 로딩/에러 상태, 재시도 함수를 제공합니다.
export function useProducts() {
  const [products, setProducts]     = useState([]);
  const [isLoading, setIsLoading]   = useState(true);
  const [error, setError]           = useState(null);
  const [fetchIndex, setFetchIndex] = useState(0);

  useEffect(() => {
    const load = async () => {
      try {
        await new Promise(r => setTimeout(r, 1000));
        if (Math.random() < 0.3) throw new Error("서버 응답 오류 (시뮬레이션)");
        setProducts(PRODUCTS);
      } catch (err) {
        setError(err.message || "상품을 불러오지 못했습니다. 다시 시도해 주세요.");
      } finally {
        setIsLoading(false);
      }
    };

    load();
  }, [fetchIndex]);

  const reload = () => {
    setIsLoading(true);
    setError(null);
    setFetchIndex(i => i + 1);
  };

  return { products, isLoading, error, reload };
}
```

### 훅 적용 후 App.jsx — UI만 남음

```jsx
// 시연 3: Custom Hook
// 비동기 로딩 로직(useProducts)만 훅으로 분리하고,
// 단순한 카운터 상태는 컴포넌트에 직접 둔다.
import { useState } from "react";
import { useProducts } from "./hooks/useProducts";
import ShoppingHeader from "../shared/components/ShoppingHeader";
import ProductList from "../shared/components/ProductList";
import Loading from "../shared/components/Loading";
import ErrorMessage from "../shared/components/ErrorMessage";
import EmptyState from "../shared/components/EmptyState";
import "@/App.css";

export default function ShoppingApp() {
  const { products, isLoading, error, reload } = useProducts();
  const [cartCount, setCartCount] = useState(0);

  const handleAddToCart = () => setCartCount((c) => c + 1);

  const renderContent = () => {
    if (isLoading) return <Loading />;
    if (error) return <ErrorMessage message={error} onRetry={reload} />;
    if (products.length === 0) return <EmptyState />;
    return <ProductList products={products} onAdd={handleAddToCart} />;
  };

  return (
    <div className="demo-app">
      <ShoppingHeader count={cartCount} />
      <div className="demo-content">{renderContent()}</div>
    </div>
  );
}
```

<aside>
📌

**비교:** 시연 2 App.jsx(30줄 상태+로직+UI 혼재) → 시연 3 App.jsx(15줄 UI만). `useProducts` 하나가 products, isLoading, error, reload를 전부 책임집니다.

</aside>

<aside>
🎓

**Custom Hook 규칙**

- 함수 이름이 반드시 `use`로 시작해야 합니다 (ex. `useProducts`, `useFetch`, `useLocalStorage`)
- 내부에서 다른 Hook(`useState`, `useEffect` 등)을 호출할 수 있습니다
- 일반 JS 함수처럼 import해서 어디서든 재사용할 수 있습니다
- 모든 로직을 훅으로 빼는 것이 아니라, **복잡한 관심사**만 분리합니다
</aside>