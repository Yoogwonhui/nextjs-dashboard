## Next.js App Router Course - Starter

### Navigating Between Pages

페이지 사이를 연결하려면 일반적으로 <a> HTML 요소 사용한다.
<a> 태그를 사용하면 각 페이지를 탐색할때마다 전체 페이지 새로고침이 발생한다.

이를막기위해 <Link/> 태그를 사용하면 된다.

<Link/> 태그를 쓰게되면 JavaScript를 사용하여 클라이언트측 탐색을 수행할 수 있다.

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

### Fetching Data

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

위와 같은 코드는 waterfall를 생성한다.
이런 경우 이전 요청이 완료되어야 다음 요청을 진행할 수 있다.
이런 패턴은 다음 요청을 하기전에 특정 조건을 충족해야하는경우 필요하다.

waterfall을 방지하는 일반적인 방법은 모든 데이터 요청을 동시에 병렬로 시작하는것이다.
JavaScript에서는 다음을 사용할 수 있습니다.Promise.all()또는Promise.allSettled()은 모든 Promise를 동시에 시작하는 기능을 한다

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
