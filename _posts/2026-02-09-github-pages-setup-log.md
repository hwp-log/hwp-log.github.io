---
layout: single
title: "[next.js] ActiveLink 만들기"
date: 2026-02-09
categories: [next.js]
---

next.js 기본기를 다지고자 이전 정리한 글에 이어 작성하는 기록이다.  
→ [이전 정리 노션 문서 바로가기](https://www.notion.so/next-js_session-1-2fd23927a2a7806c9437dbb874633045?source=copy_link)

강의내용은 x.com을 클론코딩하는 내용으로, 서비스에서 사용되는 기능을 next.js로 구현하는 것이 목적이다.
현재까지 구현한 내용은 x.com의 메인페이지 및 로그인페이지를 구현하였고 Home메뉴의 레이아웃을 작성한 상태이다.
어쨌든 중요한 것은.. 이걸 만들면서도 정작 진짜 필요한 기술인지에 대한 의문이 들었어야 하는 것이다.

예를 들면, 로그인 페이지를 만들때도 모달팝업을 굳이 사용할 필요가 있었냐는 점이다. 티스토리, 네이버 등등.. 여러 사이트들도 모달팝업을 이용한 로그인을 하지 않고 있다. 즉, 모달팝업을 하는 로그인 페이지가 인증정보를 갖고 있다던지.. 기타 추측이 되는 상황인데, 정확히 모르는 알지 못하는 상황에서 만든다면 그것은 상황에 적합하지 않은 기술을 사용한 것과 다름이 없다.

따라서 이번 블로그에 게시하는 글부터는 그런 의문점에서 출발하여 문제를 해결하고 시행착오를 겪는과정을 중심으로 적어야겠다고 생각하였다.

## ActiveLink란?

![ActiveLink 예시](/assets/post_images/2026-02-09/image0.png)

말그대로 "지금 내가 어디 페이지에 있는지 표시" 하는 기능
즉, 어느페이지에 접속해 있는지 메뉴가 볼드처리 되면서 활성화 된 기능을 뜻한다. 사용자 관점에서 본다면, 내가 어느메뉴를 사용하고 있는지 기억하고 있지는 않다. 따라서 이를 확인하려면 메뉴에 테두리가 처지던, 아니면 다른 글씨색이 되던 표시상태가 필요하다.
특히 x-com를 사용한다면, 좌:메뉴 우:내용 으로 구성되어 있기 때문에, 이런 시각적인 요소가 필요하다.

## 그럼 적용은 어떻게?

작성한 서버 컴포넌트에서는 사용이 불가하기 때문에, 클라이언트 컴포넌트로 전환 해줘야 한다.

```tsx
// (afterLogin)/layout.tsx
import { ReactNode } from "react";
import style from "@/app/(afterLogin)/layout.module.css";
import Link from "next/link";
import Image from "next/image";
import zLogo from "../../../public/zlogo.png";

export default function AfterLoginLayout({
  children,
}: {
  children: ReactNode;
}) {
  return (
    <div className={style.container}>
      <header className={style.leftSectionWrapper}>
        // ...
          <nav>
            <ul>
              {/* NavMenu 컴포넌트
              여기를 클라이언트 컴포넌트로 바꿔줘야 한다.*/}
              <NavMenu /> 
            </ul>
          </nav>
      </header>
      <div className={style.rightSectionWrapper}>
        // ...
      </div>
      {children}
    </div>
  );
}
```

<NavMenu /> 컴포넌트 내부에서 클라이언트 컴포넌트로 변경.

```tsx
// (afterLogin)/_component/NavMenu.tsx
"use client"; //클라이언트 컴포넌트로 처리

export default function NavMenu() {
  return <></>;
}
```

이후 useSelectedLayoutSegment()를 사용하여 현재 활성화 된 컴포넌트를 파악한다.

흐름은 아래와 같다.
예)
1. 사용자가 /explore 페이지에 접속
2. NavMenu 컴포넌트가 렌더링됨
3. useSelectedLayoutSegment()가 실행됨
4. segment = 'explore' 값 리턴

navbar는 그 값을 가지고 아래처럼 처리한다.

```tsx
// 예시
<Link className={segment === 'home' ? 'active' : ''}>홈</Link>
<Link className={segment === 'explore' ? 'active' : ''}>탐색하기</Link>
```

→ "지금 어디 페이지에 있는지" 활성화 표시


상세 내부는 아래와 같다.
```tsx
// 예시
<div className={style.navPill}>
  {segment && ['search', 'explore'].includes(segment) ? (
    // 활성 상태: /search, /explore 일 때
    <>
      <svg
        width={26}
        viewBox="0 0 24 24"
        aria-hidden="true"
        className="r-18jsvk2 r-4qtqp9 r-yyyyoo r-lwhw9o r-dnmrzs r-bnwqim r-1plcrui r-lrvibr r-cnnz9e"
      >
        <g>
          {/* 활성용 아이콘 path */}
          <path d="M10.25 4.25c-3.314 0-6 2.686-6 6s2.686 6 6 6c1.657 0 3.155-.67 4.243-1.757 1.087-1.088 1.757-2.586 1.757-4.243 0-3.314-2.686-6-6-6zm-9 6c0-4.971 4.029-9 9-9s9 4.029 9 9c0 1.943-.617 3.744-1.664 5.215l4.475 4.474-2.122 2.122-4.474-4.475c-1.471 1.047-3.272 1.664-5.215 1.664-4.971 0-9-4.029-9-9z" />
        </g>
      </svg>
      <div style={{ fontWeight: 'bold' }}>탐색하기</div>
    </>
  ) : (
    // 비활성 상태: 그 외 나머지 경로일 때
    <>
      <svg
        width={26}
        viewBox="0 0 24 24"
        aria-hidden="true"
        className="r-18jsvk2 r-4qtqp9 r-yyyyoo r-lwhw9o r-dnmrzs r-bnwqim r-1plcrui r-lrvibr r-cnnz9e"
      >
        <g>
          {/* 비활성용 아이콘 path */}
          <path d="M10.25 3.75c-3.59 0-6.5 2.91-6.5 6.5s2.91 6.5 6.5 6.5c1.795 0 3.419-.726 4.596-1.904 1.178-1.177 1.904-2.801 1.904-4.596 0-3.59-2.91-6.5-6.5-6.5zm-8.5 6.5c0-4.694 3.806-8.5 8.5-8.5s8.5 3.806 8.5 8.5c0 1.986-.682 3.815-1.824 5.262l4.781 4.781-1.414 1.414-4.781-4.781c-1.447 1.142-3.276 1.824-5.262 1.824-4.694 0-8.5-3.806-8.5-8.5z" />
        </g>
      </svg>
      <div>탐색하기</div>
    </>
  )}
</div>
```

상태의 활성화/비활성화를 캐치해서 리턴받은뒤, 그 값에 따라 svg아이콘과 폰트의 상태를 바꿔준다.

## 적용 결과

![ActiveLink 적용결과](/assets/post_images/2026-02-09/image0.png)

해당 페이지에서 ActiveLink가 활성화 되었음을 알 수 있다.