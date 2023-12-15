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
