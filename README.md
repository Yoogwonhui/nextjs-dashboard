# Next.js App Router Course - Starter

## Navigating Between Pages

페이지 사이를 연결하려면 일반적으로 <a> HTML 요소 사용합니다.
<a> 태그를 사용하면 각 페이지를 탐색할때마다 전체 페이지 새로고침이 발생합니다.

이를막기위해 <Link/> 태그를 사용하면 됩니다.

<Link/> 태그를 쓰게되면 JavaScript를 사용하여 클라이언트측 탐색을 수행할 수 있습니다.

- 링크 활성화

```javascript
'use client';  // usePathname() 은 hooks 이므로 클라이언트 구성요소로 바꿔야한다.
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import clsx from 'clsx';

const links = [...];

export default function NavLinks() {
  const pathname = usePathname(); // 현재 경로를 알 수 있다.

  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className={clsx(
              {
                'bg-sky-100 text-blue-600': pathname === link.href, // 조건부로 클래스를 적용할 수 있다.
              },
            )}
          >
          </Link>
        );
      })}
    </>
  );
}
```

## Fetching Data

```javascript
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
  numberOfInvoices,
  numberOfCustomers,
  totalPaidInvoices,
  totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

위와 같은 코드는 waterfall를 생성합니다.
이런 경우 이전 요청이 완료되어야 다음 요청을 진행할 수 있습니다.
이런 패턴은 다음 요청을 하기전에 특정 조건을 충족해야하는경우 필요합니다.

waterfall을 방지하는 일반적인 방법은 모든 데이터 요청을 동시에 병렬로 시작하는것입니다.
JavaScript에서는 다음을 사용할 수 있습니다. Promise.all() 또는 Promise.allSettled()은 모든 Promise를 동시에 시작하는 기능을 합니다.

```javascript
export async function fetchCardData() {
  try {
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;

    const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);
    // ...
  }
}
```

## Static and Dynamic Rendering

정적 렌더링을 사용하면 빌드 시(배포 시) 또는 재검증 중에 데이터 가져오기 및 렌더링이 서버에서 발생합니다. 그런 다음 결과를 콘텐츠 전송 네트워크(CDN)에 배포하고 캐시할 수 있습니다. 그러므로 아래와 같은 이점이 있습니다.

더 빠른 웹사이트 - 사전 렌더링된 콘텐츠를 캐시하고 전 세계에 배포할 수 있습니다. 이를 통해 전 세계 사용자가 귀하의 웹사이트 콘텐츠에 더욱 빠르고 안정적으로 액세스할 수 있습니다.

서버 로드 감소 - 콘텐츠가 캐시되기 때문에 서버는 각 사용자 요청에 대해 콘텐츠를 동적으로 생성할 필요가 없습니다.

SEO - 사전 렌더링된 콘텐츠는 페이지가 로드될 때 이미 콘텐츠를 사용할 수 있으므로 검색 엔진 크롤러가 색인을 생성하기가 더 쉽습니다. 이를 통해 검색 엔진 순위가 향상될 수 있습니다.

정적 렌더링은 데이터가 없거나 또는 사용자 간에 공유되는 데이터가 있는 UI에 유용합니다. (예: 정적 블로그 게시물 또는 제품 페이지)

동적 렌더링을 사용하면 요청 시간(사용자가 페이지를 방문할 때)에 각 사용자의 서버에서 콘텐츠가 렌더링됩니다. 동적 렌더링에는 몇 가지 이점이 있습니다.

실시간 데이터 - 동적 렌더링을 사용하면 애플리케이션이 실시간 또는 자주 업데이트되는 데이터를 표시할 수 있습니다. 이는 데이터가 자주 변경되는 애플리케이션에 이상적입니다.

사용자별 콘텐츠 - 대시보드나 사용자 프로필과 같은 개인화된 콘텐츠를 제공하고 사용자 상호작용을 기반으로 데이터를 업데이트하는 것이 더 쉽습니다.

요청 시간 정보 - 동적 렌더링을 사용하면 쿠키나 URL 검색 매개변수와 같이 요청 시에만 알 수 있는 정보에 액세스할 수 있습니다.

## Streaming

스트리밍은 경로를 더 작은 "청크"로 나눌 수 있는 데이터 전송 기술입니다. 준비가 되면 서버에서 클라이언트로 점진적으로 스트리밍합니다.

스트리밍하면 느린 데이터 요청이 전체 페이지를 차단하는 것을 방지할 수 있습니다. 이를 통해 사용자는 UI가 사용자에게 표시되기 전에 모든 데이터가 로드될 때까지 기다리지 않고 페이지의 일부를 보고 상호 작용할 수 있습니다.

Next.js에서 스트리밍을 구현하는 방법에는 두 가지가 있습니다.

1. 페이지 수준에서 loading.tsx 파일을 사용합니다.
2. 특정 구성요소의 경우 <Suspense>.

## Search

```javascript
'use client';

import { useSearchParams, usePathname, useRouter } from 'next/navigation';

export default function Search({ placeholder }: { placeholder: string }) {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);

    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }

  return (
    <div>
      <input
        placeholder={placeholder}
        onChange={(e) => {
          handleSearch(e.target.value);
        }}
        defaultValue={searchParams.get('query')?.toString()}
      />
    </div>
  );
}

```

${pathname}귀하의 경우 현재 경로입니다 "/dashboard/invoices".
사용자가 검색창에 입력하면 params.toString()이 입력이 URL 친화적인 형식으로 변환됩니다.
replace(${pathname}?${params.toString()})사용자의 검색 데이터로 URL을 업데이트합니다. 예를 들어 /dashboard/invoices?query=lee사용자가 "Lee"를 검색하는 경우입니다.
Next.js의 클라이언트 측 탐색 덕분에 페이지를 다시 로드하지 않고도 URL이 업데이트됩니다.

이때 키를 입력할때마다 URL을 업데이트하므로 키를 누를 때마다 데이터베이스를 쿼리하게 됩니다! 우리 애플리케이션이 작기 때문에 이것은 문제가 되지 않습니다. 그러나 애플리케이션에 수천 명의 사용자가 있고 각 사용자가 키를 누를 때마다 데이터베이스에 새로운 요청을 보내는 경우 문제가 된다.

디바운싱은 함수가 실행될 수 있는 속도를 제한하는 프로그래밍 방식입니다. 우리의 경우에는 사용자가 입력을 중단한 경우에만 데이터베이스를 쿼리하려고 합니다.

npm i use-debounce

디바운싱을 통해 데이터베이스로 전송되는 요청 수를 줄여 리소스를 절약할 수 있습니다.

```javascript
import { useDebouncedCallback } from 'use-debounce';

// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);

  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
```
